# Выполнение домашнего задания на тему централизованный сбор логов.
Создаем стенд
```
Vagrant.configure("2") do |config|

  # Base VM OS configuration.
  config.vm.box = "ubuntu/jammy64"
  config.vm.provider :virtualbox do |v|
    v.memory = 2024
    v.cpus = 2
  end

  # Define two VMs with static private IP addresses.
  boxes = [
    { :name => "web",
      :ip => "192.168.56.10",
    },
    { :name => "log",
      :ip => "192.168.56.15",
    }
  ]

  # Provision each of the VMs.
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.network "private_network", ip: opts[:ip]
    end
  end

end

```
 Проверяем, что оба сервера синхронизируют время с ntp сервером
```
root@web:/home/vagrant# timedatectl status
               Local time: Tue 2025-04-22 22:00:14 UTC
           Universal time: Tue 2025-04-22 22:00:14 UTC
                 RTC time: Tue 2025-04-22 22:00:14
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```


## Настройка центрального сервера сбора логов
1. Подключаемся к серверу log и проверям, что на нем установлен rsyslog
```
vagrant ssh log

vagrant@log:~$ apt list rsyslog
Listing... Done
rsyslog/jammy-updates,jammy-security,now 8.2112.0-2ubuntu2.2 amd64 [installed,automatic]
N: There is 1 additional version. Please use the '-a' switch to see it
```
2. Все настройки Rsyslog хранятся в файле /etc/rsyslog.conf . Для того, чтобы наш сервер мог принимать логи, нам необходимо внести следующие изменения в файл: 
```
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")
```
В конце файла добавляем правило приема сообщенйи от хостов
```
#Add remote logs
$Template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
stop
```
3. Перезапускаем службу rsyslog
```
systemctl restart rsyslog
```
4. Проверяем, что требуемые порты открыты
```
root@log:/home/vagrant# ss -tulpan | grep rsyslog
udp   UNCONN 0      0               0.0.0.0:514       0.0.0.0:*     users:(("rsyslogd",pid=2262,fd=5))
udp   UNCONN 0      0                  [::]:514          [::]:*     users:(("rsyslogd",pid=2262,fd=6))
tcp   LISTEN 0      25              0.0.0.0:514       0.0.0.0:*     users:(("rsyslogd",pid=2262,fd=7))
tcp   LISTEN 0      25                 [::]:514          [::]:*     users:(("rsyslogd",pid=2262,fd=8))
```
### Настройка отправки логов с хоста WEB
1. На сервере web устанавливаем Nginx
```
apt update && apt install -y nginx
```
2. Проверяем
```
root@web:/home/vagrant# curl 192.168.56.10
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```
3. Находим в файле /etc/nginx/nginx.conf раздел с логами и приводим их к следующему виду:
```
access_log /var/log/nginx/access.log;
access_log syslog:server=192.168.56.15:514,tag=nginx_access,severity=info combined;
error_log /var/log/nginx/error.log;
error_log syslog:server=192.168.56.15:514,tag=nginx_error;
```
4. Проверяем конфигурацию nginx
```
root@web:/home/vagrant# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
5. Перезапускаем Nginx
```
root@web:/home/vagrant# systemctl restart nginx
```
6. Проверяем. заходим на log-сервер и смотрим информацию об nginx:
```
root@log:/var/log# cat /var/log/rsyslog/web/nginx_access.log
Apr 22 23:30:59 web nginx_access: 192.168.56.15 - - [22/Apr/2025:23:30:59 +0000] "GET / HTTP/1.1" 403 162 "-" "curl/7.81.0"
Apr 22 23:31:00 web nginx_access: 192.168.56.15 - - [22/Apr/2025:23:31:00 +0000] "GET / HTTP/1.1" 403 162 "-" "curl/7.81.0"
Apr 22 23:31:05 web nginx_access: 192.168.56.10 - - [22/Apr/2025:23:31:05 +0000] "GET / HTTP/1.1" 403 162 "-" "curl/7.81.0"
Apr 22 23:31:06 web nginx_access: 192.168.56.10 - - [22/Apr/2025:23:31:06 +0000] "GET / HTTP/1.1" 403 162 "-" "curl/7.81.0"
root@log:/var/log# cat /var/log/rsyslog/web/nginx_error.log
Apr 22 23:30:59 web nginx_error: 2025/04/22 23:30:59 [error] 3318#3318: *12 directory index of "/var/www/html/" is forbidden, client: 192.168.56.15, server: _, request: "GET / HTTP/1.1", host: "192.168.56.10"
Apr 22 23:31:00 web nginx_error: 2025/04/22 23:31:00 [error] 3318#3318: *13 directory index of "/var/www/html/" is forbidden, client: 192.168.56.15, server: _, request: "GET / HTTP/1.1", host: "192.168.56.10"
Apr 22 23:31:05 web nginx_error: 2025/04/22 23:31:05 [error] 3318#3318: *14 directory index of "/var/www/html/" is forbidden, client: 192.168.56.10, server: _, request: "GET / HTTP/1.1", host: "192.168.56.10"
Apr 22 23:31:06 web nginx_error: 2025/04/22 23:31:06 [error] 3318#3318: *15 directory index of "/var/www/html/" is forbidden, client: 192.168.56.10, server: _, request: "GET / HTTP/1.1", host: "192.168.56.10"
```
### Настройка отправки логов с доп.хоста
1. На хосте внесем в конфигурацию информацию куда и что мы хотим слать на удаленный сервер rsyslog
```
nano /etc/rsyslog.conf

#remove host
*.err @192.168.56.15:514
```
2. Перезапускаем службу rsyslog
```
systemctl restart rsyslog
```
3. Сделаем тестовое сообщение об ошибке
```
logger -p err "Test error message"
```
4. Проверяем на сервере rsyslog
```
root@log:/var/log/rsyslog/dop# cat root.log
Apr 23 00:59:36 dop root: Test error message
```

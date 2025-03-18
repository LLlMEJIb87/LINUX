# Выполнение домашнего задания на тему SELinux
## Задани №1 Запуск nginx на нестандартном порту 3-мя разными способами 
1. Проверяем, что в OS отключен файерволл
```
[root@selinux ~]# systemctl status firewalld
○ firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:firewalld(1)
```
2. Проверяем, что конфигурация nginx настроена без ошибок
```
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
3. Проверяем режим работы SELinux
```
[root@selinux ~]# getenforce
Enforcing
```
4. Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setseboo
- Находим в логах (/var/log/audit/audit.log) информацию о блокировании порта
```
[root@selinux ~]# grep 1742098175.687:618 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1742098175.687:618): avc:  denied  { name_bind } for  pid=3623 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly. 
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
```
Утилита audit2why показала почему трафик блокируется. Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр nis_enabled
```
[root@selinux ~]# setsebool -P nis_enabled on
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2025-03-16 04:13:40 UTC; 8s ago
    Process: 3649 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 3650 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 3651 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 3652 (nginx)
      Tasks: 3 (limit: 10740)
     Memory: 2.9M
        CPU: 40ms
     CGroup: /system.slice/nginx.service
             ├─3652 "nginx: master process /usr/sbin/nginx"
             ├─3653 "nginx: worker process"
             └─3654 "nginx: worker process"

Mar 16 04:13:39 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 16 04:13:39 selinux nginx[3650]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 16 04:13:39 selinux nginx[3650]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 16 04:13:40 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```
Проверяем
```
[root@selinux ~]# curl -s http://192.168.1.211:4881/
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
        <head>
                <title>Test Page for the HTTP Server on AlmaLinux</title>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
                <style type="text/css">
                        /*<![CDATA[*/
                        body {
                                background-color: #FAF5F5;
                                color: #000;
                                font-size: 0.9em;
                                font-family: sans-serif,helvetica;
                                margin: 0;
                                padding: 0;
...
```
- Вернём запрет работы nginx на порту 4881 обратно. Для этого отключим nis_enabled
```
setsebool -P nis_enabled off
```

5. Теперь разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип
- Поиск имеющегося типа, для http трафика: 
```
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```
- Добавим порт в тип http_port_t:
```
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
```
Проверяем
```
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
```
- Теперь перезапускаем службу nginx и проверим её работу
```
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2025-03-16 04:26:16 UTC; 9s ago
    Process: 3721 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 3722 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 3723 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 3724 (nginx)
      Tasks: 3 (limit: 10740)
     Memory: 2.9M
        CPU: 37ms
     CGroup: /system.slice/nginx.service
             ├─3724 "nginx: master process /usr/sbin/nginx"
             ├─3725 "nginx: worker process"
             └─3726 "nginx: worker process"

Mar 16 04:26:16 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 16 04:26:16 selinux nginx[3722]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 16 04:26:16 selinux nginx[3722]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 16 04:26:16 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```
- Удалить нестандартный порт из имеющегося типа можно с помощью команды:
```
semanage port -d -t http_port_t -p tcp 4881
```
Проверяем
```
[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```
6. Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux
- Смотрим логи, которые относятся к nginx
```
type=SYSCALL msg=audit(1742099366.160:642): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=5593c6069ff0 a2=10 a3=7fff8c7a8e50 items=0 ppid=1 pid=3744 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1742099366.161:643): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
```
- Воспользуемся утилитой audit2allow для того, чтобы на основе логов SELinux сделать модуль, разрешающий работу nginx на нестандартном порту: 
```
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
```
- Audit2allow сформировал модуль, и сообщил нам команду, с помощью которой можно применить данный модуль
```
semodule -i nginx.pp
```
Проверяем
```
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2025-03-16 04:33:52 UTC; 6s ago
    Process: 3774 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 3775 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 3776 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 3777 (nginx)
      Tasks: 3 (limit: 10740)
     Memory: 2.9M
        CPU: 38ms
     CGroup: /system.slice/nginx.service
             ├─3777 "nginx: master process /usr/sbin/nginx"
             ├─3778 "nginx: worker process"
             └─3779 "nginx: worker process"

Mar 16 04:33:52 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 16 04:33:52 selinux nginx[3775]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 16 04:33:52 selinux nginx[3775]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 16 04:33:52 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
```
Итог:    
- При использовании модуля изменения сохранятся после перезагрузки. 
- Просмотр всех установленных модулей: semodule -l
- Для удаления модуля воспользуемся командой: semodule -r nginx

## Задание №2 Обеспечение работоспособности приложения при включенном SELinux
Произвел установку серверов с помощью vagrant согласно методическому пособию.
1. Попробуем внести изменения в зону
```
[vagrant@client ~]$  nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
update failed: SERVFAIL
> quit
```
2. Изменения внести не получилось. Cмотрим логи SELinux, чтобы понять в чём может быть проблема.
```
[root@client ~]# cat /var/log/audit/audit.log | audit2why
```
Тут мы видим, что на клиенте отсутствуют ошибки     
3. Не закрывая сессию на клиенте, подключимся к серверу ns01 и проверим логи SELinux:
```
[root@ns01 ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1742297523.382:2325): avc:  denied  { write } for  pid=25149 comm="isc-net-0001" name="dynamic" dev="sda4" ino=33575229 scontext=system_u:system_r:named_t:s0 tcontext=unconfined_u:object_r:named_conf_t:s0 tclass=dir permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.
```
В логах мы видим, что ошибка в контексте безопасности. Целевой контекст named_conf_t      
4. Для сравнения посмотрим существующую зону (localhost) и её контекст:
```
[root@ns01 ~]# ls -alZ /var/named/named.localhost
-rw-r-----. 1 root named system_u:object_r:named_zone_t:s0 152 Feb 19 16:04 /var/named/named.localhost
```
У наших конфигов в /etc/named вместо типа named_zone_t используется тип named_conf_t.      
5. Проверим данную проблему в каталоге /etc/named:
```
[root@ns01 ~]# ls -laZ /etc/named
total 28
drw-rwx---.  3 root named system_u:object_r:named_conf_t:s0      121 Mar 18 11:23 .
drwxr-xr-x. 85 root root  system_u:object_r:etc_t:s0            8192 Mar 18 11:23 ..
drw-rwx---.  2 root named unconfined_u:object_r:named_conf_t:s0   56 Mar 18 11:23 dynamic
-rw-rw----.  1 root named system_u:object_r:named_conf_t:s0      784 Mar 18 11:23 named.50.168.192.rev
-rw-rw----.  1 root named system_u:object_r:named_conf_t:s0      610 Mar 18 11:23 named.dns.lab
-rw-rw----.  1 root named system_u:object_r:named_conf_t:s0      609 Mar 18 11:23 named.dns.lab.view1
-rw-rw----.  1 root named system_u:object_r:named_conf_t:s0      657 Mar 18 11:23 named.newdns.lab
```
Тут мы также видим, что контекст безопасности неправильный. Проблема заключается в том, что конфигурационные файлы лежат в другом каталоге.      
6. Cмотрим в каком каталоги должны лежать, файлы, чтобы на них распространялись правильные политики SELinux можно с помощью команды: 
```
/etc/named(/.*)?                                   all files          system_u:object_r:named_conf_t:s0 
/var/named(/.*)?                                   all files          system_u:object_r:named_zone_t:s0 
```
7. Изменим тип контекста безопасности для каталога /etc/named:
```
sudo chcon -R -t named_zone_t /etc/named
```
Проверяем
```
[root@ns01 ~]# ls -laZ /etc/named
total 28
drw-rwx---.  3 root named system_u:object_r:named_zone_t:s0      121 Mar 18 11:23 .
drwxr-xr-x. 85 root root  system_u:object_r:etc_t:s0            8192 Mar 18 11:23 ..
drw-rwx---.  2 root named unconfined_u:object_r:named_zone_t:s0   56 Mar 18 11:23 dynamic
-rw-rw----.  1 root named system_u:object_r:named_zone_t:s0      784 Mar 18 11:23 named.50.168.192.rev
-rw-rw----.  1 root named system_u:object_r:named_zone_t:s0      610 Mar 18 11:23 named.dns.lab
-rw-rw----.  1 root named system_u:object_r:named_zone_t:s0      609 Mar 18 11:23 named.dns.lab.view1
-rw-rw----.  1 root named system_u:object_r:named_zone_t:s0      657 Mar 18 11:23 named.newdns.lab
```
8. Попробуем снова внести изменения с клиента: 
```
[root@client ~]# nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
> quit
```
Проверяем

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%91%D0%B5%D0%B7%D0%BE%D0%BF%D0%B0%D1%81%D0%BD%D0%BE%D1%81%D1%82%D1%8C/Pictures/dig.PNG">
</p>


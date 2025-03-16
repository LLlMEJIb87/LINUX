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

5. Теперь разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип:

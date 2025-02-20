# Домашняя работа по теме ZFS
__Задача: Развернуть сервис NFS и подключить к нему клиентов.__    
1. Устанавливаем сервер NFS
```
apt install nfs-kernel-server
```
Проверяем наличие слушающих портов 2049 tcp/udp, 111 tcp/udp
```
root@lvm:~#  ss -tulpan | grep -E '2049|111'
udp   UNCONN 0      0                     0.0.0.0:111                 0.0.0.0:*     users:(("rpcbind",pid=12875,fd=5),("systemd",pid=1,fd=124))
udp   UNCONN 0      0                        [::]:111                    [::]:*     users:(("rpcbind",pid=12875,fd=7),("systemd",pid=1,fd=130))
tcp   LISTEN 0      64                    0.0.0.0:2049                0.0.0.0:*                                                                
tcp   LISTEN 0      4096                  0.0.0.0:111                 0.0.0.0:*     users:(("rpcbind",pid=12875,fd=4),("systemd",pid=1,fd=122))
tcp   LISTEN 0      64                       [::]:2049                   [::]:*                                                                
tcp   LISTEN 0      4096                     [::]:111                    [::]:*     users:(("rpcbind",pid=12875,fd=6),("systemd",pid=1,fd=129))
```
2. Создаём и настраиваем директорию, которая будет экспортирована в будущем ("Экспортировать" директорию — значит сделать её доступной по сети для других машин через NFS.)
```
root@lvm:~# mkdir -p /srv/share/upload
root@lvm:~# chown -R nobody:nogroup /srv/share #стандартный "безымянный" пользователь и группа для NFS (в анонимных экспортируемых папках)
root@lvm:~# chmod 0777 /srv/share/upload
```
3. Создаем конфиг экспорта, в файле /etc/exports указываем какую папку будем "шарить" в сеть, кто к ней может подключаться и какие права иметь
```
/srv/share 192.168.1.0/24(rw,sync,root_squash)
```
Применяем экспорт
```
exportfs -ra
```
Проверяем
```
exportfs -v
/srv/share      192.168.1.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
Готово

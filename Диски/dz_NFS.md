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
Готово, далее пробуем подключиться на клиенте:
1. Устанавливаем клиента NFS
```
apt update && sudo apt install -y nfs-common
```
2. Монтируем сетевой диск 
```
mount -t nfs 192.168.1.210:/srv/share /mnt
```
3. Пропишем в fstab для автомонтирования при загрузки системы
```
echo "192.168.1.210:/srv/share /mnt nfs noauto,x-systemd.automount 0 0" >> /etc/fstab
```
4. Перечитаем конфигурацию
```
systemctl daemon-reload
```
5. Перезапускаем сервис монтирования удалённых файловых систем.(для того чтобы убедиться, что удаленная директория примонтируется после перезапуска системы не перезапуская систему)
```
systemctl restart remote-fs.target
```
Проверяем 
```
mount | grep mnt
192.168.1.210:/srv/share on /mnt type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.1.250,local_lock=none,addr=192.168.1.210)
```
6. На сервере создали тестовый файл, првоеряем появился ли он на клиенте
```
ls -lah /mnt/upload/
drwxrwxrwx 2 nobody nogroup 4.0K Feb 20 10:01 .
drwxr-xr-x 3 nobody nogroup 4.0K Feb 20 09:16 ..
-rw-r--r-- 1 root   root       0 Feb 20 10:01 test
```

# Домашняя работа на тему LVM
## Задача №1 Уменьшить том под / до 8G
Изначальный размер тома / 12G , так как мы не можем уменьшать размер тома на "живую" его нужно отмонтировать, но перед этим подготовить врменный том для переноса файлов.
```
df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  197M  1.1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    12G  4.3G  6.4G  41% /
```
1. Подготовим временный том на другом диске
```
pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
vgcreate root_vg /dev/sdb
  Volume group "root_vg" successfully created
lvcreate -l +100%FREE -n root_lv root_vg
  Logical volume "root_lv" created.
```
2. Создаем файловую систему и монтируем
```
mkfs.ext4 /dev/root_vg/root_lv
mount /dev/root_vg/root_lv /mnt
```
Проверяем
```
df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  197M  1.1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    12G  4.3G  6.4G  41% /
tmpfs                             tmpfs  984M     0  984M   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G   95M  1.7G   6% /boot
tmpfs                             tmpfs  197M   12K  197M   1% /run/user/1000
/dev/mapper/root_vg-root_lv       ext4    25G   24K   24G   1% /mnt
```
3. Клонируем корневую файловую систему на временный логический том
```
rsync -avxHAX --progress / /mnt
```
4. Для того, чтобы нам можно было загрузиться с нового  тома, делаем следующее
- монтируем необходимые катологи
```
for dir in dev proc sys run boot; do mount --bind /$dir /mnt/$dir; done
```
- Переходим в новое окружение
```
chroot /mnt
```
Проверяем
```
df -hT
Filesystem                  Type   Size  Used Avail Use% Mounted on
/dev/mapper/root_vg-root_lv ext4    25G  4.3G   19G  19% /
tmpfs                       tmpfs  197M  1.1M  196M   1% /run
/dev/sda2                   ext4   2.0G   95M  1.7G   6% /boot
```
- Создаём конфигурационный файл GRUB на основе текущей системы.
```
grub-mkconfig -o /boot/grub/grub.cfg
```
- Обновляет initramfs
```
update-initramfs -u
```
- Перезагружаем систему
```
exit
shutdown -r now
```
Проверяем
```
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:1    0 11.5G  0 lvm  
sdb                         8:16   0   25G  0 disk 
└─root_vg-root_lv         252:0    0   25G  0 lvm  / #Видим, что система  подмонтирована к диску sdb
sdc                         8:32   0   10G  0 disk 
sdd                         8:48   0   10G  0 disk 
sr0                        11:0    1 1024M  0 rom  
```
5. Создаём новый том на 8G
- Удаляем целевой том
```
lvremove /dev/ubuntu-vg/ubuntu-lv
```
- Создаем новый
```
lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
```
- создаем файловую систему 
```
mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv
```
- монтируем
```
mount /dev/ubuntu-vg/ubuntu-lv /mnt
```
6. Клонируем корневую файловую систему на временный том
```
rsync -avxHAX --progress / /mnt
```
7. Для того, чтобы нам можно было загрузиться с нового тома, делаем следующее
- монтируем необходимые катологи
```
for dir in dev proc sys run boot; do mount --bind /$dir /mnt/$dir; done
```
- Переходим в новое окружение
```
chroot /mnt
```
- Создаём конфигурационный файл GRUB на основе текущей системы.
```
grub-mkconfig -o /boot/grub/grub.cfg
```
- Обновляет initramfs
```
update-initramfs -u
```
- Перезагружаем систему
```
exit
shutdown -r now
```
Проверяем
```
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:1    0    8G  0 lvm  / #Цель достигнута
sdb                         8:16   0   25G  0 disk 
└─root_vg-root_lv         252:0    0   25G  0 lvm  
```
8. Удаляем временный том
```
lvremove /dev/root_vg/root_lv 
vgremove root_vg 
pvremove /dev/sdb
```

## Задача №2 Выделить том под /var в зеркало
1. Создаем зеркало
- cоздаем физические тома на свободных дисках
```
pvcreate /dev/sdc /dev/sdd
```
- объединяем физические тома в группу
```
vgcreate var_vg /dev/sdc /dev/sdd
```
- создаем логический том
```
lvcreate -L 5G -m1 -n var_lv var_vg
```
- проверяем
```
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
...
sdc                         8:32   0   10G  0 disk 
├─var_vg-var_lv_rmeta_0   252:2    0    4M  0 lvm  
│ └─var_vg-var_lv         252:6    0    5G  0 lvm  
└─var_vg-var_lv_rimage_0  252:3    0    5G  0 lvm  
  └─var_vg-var_lv         252:6    0    5G  0 lvm  
sdd                         8:48   0   10G  0 disk 
├─var_vg-var_lv_rmeta_1   252:4    0    4M  0 lvm  
│ └─var_vg-var_lv         252:6    0    5G  0 lvm  
└─var_vg-var_lv_rimage_1  252:5    0    5G  0 lvm  
  └─var_vg-var_lv         252:6    0    5G  0 lvm  
```
2. Создаем файловую систем и монтируем 
```
mkfs.ext4 /dev/var_vg/var_lv
mount /dev/var_vg/var_lv /mnt/
```
3. Копируем всё из директори /var на новый том
```
cp -aR /var/* /mnt/
```
4. Монтируем новый том к /var
```
umount /mnt
mount /dev/var_vg/var_lv /var
```
5. Автоматизируем процесс монтирования католога
```
e2label /dev/mapper/var_vg-var_lv var
echo "`blkid | grep 'LABEL=\"var\"' | awk -F '\"' '{print \"UUID=\"$4}'` \
/var ext4 defaults 0 0" >> /etc/fstab
```
6. Перезагружаемся и проверяем
```
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0    8G  0 lvm  /
sdb                         8:16   0   25G  0 disk 
sdc                         8:32   0   10G  0 disk 
├─var_vg-var_lv_rmeta_0   252:1    0    4M  0 lvm  
│ └─var_vg-var_lv         252:5    0    5G  0 lvm  /var
└─var_vg-var_lv_rimage_0  252:2    0    5G  0 lvm  
  └─var_vg-var_lv         252:5    0    5G  0 lvm  /var
sdd                         8:48   0   10G  0 disk 
├─var_vg-var_lv_rmeta_1   252:3    0    4M  0 lvm  
│ └─var_vg-var_lv         252:5    0    5G  0 lvm  /var
└─var_vg-var_lv_rimage_1  252:4    0    5G  0 lvm  
  └─var_vg-var_lv         252:5    0    5G  0 lvm  /var

df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  197M  1.1M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4   7.8G  4.4G  3.1G  59% /
tmpfs                             tmpfs  984M     0  984M   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/mapper/var_vg-var_lv         ext4   4.9G  403M  4.2G   9% /var
/dev/sda2                         ext4   2.0G   95M  1.7G   6% /boot
tmpfs                             tmpfs  197M   12K  197M   1% /run/user/1000
```

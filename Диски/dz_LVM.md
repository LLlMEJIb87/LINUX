# Домашняя работа на тему LVM
## Задача №1 Уменьшить том под / до 8G
Изначальный размер тома / 12G , так как мы не можем уменьшать размер тома на "живую" его нужно отмонтировать, но перед этим подготовить врменный том для переноса файлов.
1. Подготовим временный том
```
root@raidlvm:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
root@raidlvm:~# vgcreate root_vg /dev/sdb
  Volume group "root_vg" successfully created
root@raidlvm:~# lvcreate -l +100%FREE -n root_lv root_vg
  Logical volume "root_lv" created.
```
2. Создаем файловую систему и монтируем
```
mkfs.ext4 /dev/root_vg/root_lv
mount /dev/root_vg/root_lv /mnt/root/
```
3. Клонируем корневую файловую систему на временный том
```
rsync -avxHAX --progress / /mnt/root
```
4. Для того, чтобы нам можно было загрузиться с нового тома, делаем следующее
- монтируем необходимые катологи
```
for dir in dev proc sys run boot; do mount --bind /$dir /mnt/root/$dir; done
```
- Переходим в новое окружение
```
chroot /mnt/root/
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
mount /dev/ubuntu-vg/ubuntu-lv /mnt/root/
```
6. Клонируем корневую файловую систему на временный том
```
rsync -avxHAX --progress / /mnt/root/
```
7. Для того, чтобы нам можно было загрузиться с нового тома, делаем следующее
- монтируем необходимые катологи
```
for dir in dev proc sys run boot; do mount --bind /$dir /mnt/root/$dir; done
```
- Переходим в новое окружение
```
chroot /mnt/root/
```
- Создаём конфигурационный файл GRUB на основе текущей системы.
```
grub-mkconfig -o /boot/grub/grub.cfg
```
- Обновляет initramfs
```
update-initramfs -u
```

## Задача №2 Выделить том под /var в зеркало
1. Создаем зеркало
- cоздаем физические тома на свободных дисках
```
pvcreate /dev/sdc /dev/sdd
```
- объединяем физические тома в группу
```
vgcreate vg_var /dev/sdc /dev/sdd
```
- создаем логический том
```
lvcreate -L 2G -m1 -n lv_var vg_var
```
- проверяем
```
root@raidlvm:/# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root_lv   root_vg   -wi-ao---- <25.00g                                                    
  ubuntu-lv ubuntu-vg -wi-ao----   8.00g                                                    
  lv_var    vg_var    rwi-a-r---   2.00g
```

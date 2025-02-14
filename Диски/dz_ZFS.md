# Домашняя работа на тему ZFS
## Задача №1 Определение алгоритма  с наилучшим сжатием.
1. Смотрим список всех дисков, которые есть в виртуальной машине
```
lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   25G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   23G  0 part 
  ├─ubuntu--vg-ubuntu--lv 252:0    0    8G  0 lvm  /
  └─ubuntu--vg-home_lv    252:1    0   10G  0 lvm  /home
sdb                         8:16   0   25G  0 disk 
sdc                         8:32   0   10G  0 disk 
sdd                         8:48   0   10G  0 disk 
sde                         8:64   0   10G  0 disk 
sdf                         8:80   0   10G  0 disk 
sdg                         8:96   0   10G  0 disk 
sdh                         8:112  0   10G  0 disk 
sdi                         8:128  0   10G  0 disk 
```
2. Создаём 4 пула из свободных дисков в режиме RAID 1:
```
zpool create -f test_1 mirror /dev/sdb /dev/sdc
zpool create -f test_2 mirror /dev/sdd /dev/sde
zpool create test_3 mirror /dev/sdf /dev/sdg
zpool create test_4 mirror /dev/sdh /dev/sdi
```
Проверяем
```
zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
test_1  9.50G   122K  9.50G        -         -     0%     0%  1.00x    ONLINE  -
test_2  9.50G   111K  9.50G        -         -     0%     0%  1.00x    ONLINE  -
test_3  9.50G   122K  9.50G        -         -     0%     0%  1.00x    ONLINE  -
test_4  9.50G   111K  9.50G        -         -     0%     0%  1.00x    ONLINE  -
```
3. Добавим разные алгоритмы сжатия в каждую файловую систему:
```
zfs set compression=lzjb test_1
zfs set compression=lz4 test_2
zfs set compression=gzip-9 test_3
zfs set compression=zle test_4
```
Проверяем
```
zfs get all | grep compression
test_1  compression           lzjb                   local
test_2  compression           lz4                    local
test_3  compression           gzip-9                 local
test_4  compression           zle                    local
```

# Домашняя работа по теме RAID массивов.
## Создание RAID массива
1. Определяемся с тем какой RAID нам нужен. К примеру наш массив будет использоваться для хранения баз данных, так как нагрузка планируется быть большой и данные важны, то в качестве массива будем использовать RAID10.
2. В нашем распоряжение 4 диска
```
root@vm-nginx:/home/shmel# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   40G  0 disk 
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0   10G  0 disk 
sdc      8:32   0   10G  0 disk 
sdd      8:48   0   10G  0 disk 
sde      8:64   0   10G  0 disk 
```
3. Приступаем к созданию RAID10
3. 1 На всякий случай удалим метаданные от возможного предыдущего RAID массива
```
mdadm --zero-superblock /dev/sd{b,c,d,e}
```
3.2 Создаем RAID10
```
mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}
```
3.3 Проверяем
```
cat /proc/mdstat
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      20953088 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
```
```
root@vm-nginx:/home/shmel#  mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jan 29 10:28:28 2025
        Raid Level : raid10
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Jan 29 10:30:13 2025
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : vm-nginx:0  (local to host vm-nginx)
              UUID : cada600c:ee13a98e:01a1a5e4:589157dd
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```
3.4 Для исключения возможных сбоев (невозможности собрать массив при старте системы)  создаем mdadm.conf - автосборка RAID
```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf #записываем в файл указание искать массив на дисках и разделах
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf #записываем в файл информацию об массиве
```
4. Создадим файловую систему
```
mkfs.ext4 /dev/md0
```
5. Теперь наш RAID для возможности использования в системе нужно примонтировать к каталогу
```
mount /dev/md0 /mnt/mysql
```
5.1 Проверяем
```
df -hT
Filesystem     Type    Size  Used Avail Use% Mounted on
/dev/md0       ext4     20G   24K   19G   1% /mnt/mysql
```
6. Делаем чтобы RAID автоматически монтировался при загрузке системы
6.1 назначаем label
```
e2label /dev/md0 RAID_MYSQL
```
6.2 Добавляем запись в /etc/fstab
```
echo "LABEL=RAID_MYSQL /mnt/mysql/ ext4 defaults 0 2" > /etc/fstab
```

## Работа с RAID массивом
1. Имитируем выход из строя одного из дисков
```
mdadm /dev/md127 --fail /dev/sdb
```
2. Cмотрим что произошло
```
root@vm-nginx:/home/shmel# mdadm -D /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Wed Jan 29 10:28:28 2025
        Raid Level : raid10
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Jan 29 11:37:05 2025
             State : clean, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : vm-nginx:0  (local to host vm-nginx)
              UUID : cada600c:ee13a98e:01a1a5e4:589157dd
            Events : 23

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       0       8       16        -      faulty   /dev/sdb #сломанный диск
```

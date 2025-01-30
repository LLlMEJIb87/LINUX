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
- На всякий случай удалим метаданные от возможного предыдущего RAID массива
```
mdadm --zero-superblock /dev/sd{b,c,d,e}
```
- Создаем RAID10
```
mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}
```
- Проверяем
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
- Для исключения возможных сбоев (невозможности собрать массив при старте системы)  создаем mdadm.conf - автосборка RAID
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
- Проверяем
```
df -hT
Filesystem     Type    Size  Used Avail Use% Mounted on
/dev/md0       ext4     20G   24K   19G   1% /mnt/mysql
```
6. Делаем чтобы RAID автоматически монтировался при загрузке системы
- назначаем label
```
e2label /dev/md0 RAID_MYSQL
```
- Добавляем запись в /etc/fstab
```
echo "LABEL=RAID_MYSQL /mnt/mysql/ ext4 defaults 0 2" >> /etc/fstab
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
3. Удаляем сломанный диск из массива
```
mdadm /dev/md127 --remove /dev/sdb
```
4. Добавляем новый диск
```
root@vm-nginx:/home/shmel# mdadm /dev/md127 --add /dev/sdf
mdadm: added /dev/sdf
```
5. Cмотрим как оно там выглядит в процессе
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

       Update Time : Wed Jan 29 11:50:07 2025
             State : clean, degraded, recovering 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 13% complete

              Name : vm-nginx:0  (local to host vm-nginx)
              UUID : cada600c:ee13a98e:01a1a5e4:589157dd
            Events : 31

    Number   Major   Minor   RaidDevice State
       4       8       80        0      spare rebuilding   /dev/sdf #происходит процесс коопирования данных на новый диск
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```
### Делим RAID на разделы
1. Отменим предыдущее монтирование и почистим fstab
2. Создадим таблицу разделов GPT
```
parted -s /dev/md127 mklabel gpt
```
Проверим
```
parted /dev/md127 print
Model: Linux Software RAID Array (md)
Disk /dev/md127: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt #Видим таблицу
Disk Flags: 
```
3. Разбиваем массив на 5 разделов
```
parted /dev/md127 mkpart primary ext4 0% 20%
parted /dev/md127 mkpart primary ext4 20% 40%            
parted /dev/md127 mkpart primary ext4 40% 60%           
parted /dev/md127 mkpart primary ext4 60% 80%           
parted /dev/md127 mkpart primary ext4 80% 100%          
```
4. Создаем файловую систему
```
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md127p$i; done
```
Проверяем
```
lsblk -f
NAME        FSTYPE            FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdc         linux_raid_member 1.2   vm-nginx:0      cada600c-ee13-a98e-01a1-a5e4589157dd                
└─md127                                                                                                 
  ├─md127p1 ext4              1.0                   829af6c5-05f2-451c-93f2-8d5aa2ed02b5                
  ├─md127p2 ext4              1.0                   8ef0bf50-913b-4638-a499-1d8a5192bbcd                
  ├─md127p3 ext4              1.0                   6d463965-6b20-48cd-8867-997661b6f7f3                
  ├─md127p4 ext4              1.0                   0f1a6924-2d2e-40d6-9dd3-205d10a797b9                
  └─md127p5 ext4              1.0                   c915660f-5370-4082-9681-fa458dba64bb                
```

5. Монтируем
```
for i in $(seq 1 5); do mount /dev/md127p$i /raid/part$i; done
```
Проверяем
```
df -hT
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/md127p1   ext4   3.9G   24K  3.7G   1% /raid/part1
/dev/md127p2   ext4   3.9G   24K  3.7G   1% /raid/part2
/dev/md127p3   ext4   3.9G   24K  3.7G   1% /raid/part3
/dev/md127p4   ext4   3.9G   24K  3.7G   1% /raid/part4
/dev/md127p5   ext4   3.9G   24K  3.7G   1% /raid/part5
```

# LVM
LVM (Logical Volume Management или Управление Логическими Томами) — это дополнительный слой абстракции от железа, позволяющий собрать кучи разнородных дисков в один, и затем снова разбить этот один именно так как нам хочется.    

Плюсы LVM:
- Динамическое выделение места
- Изменение размера логических томов
- Конститентные бекапы (snapshot)
- Кэширование данных
- Формирование RAID массивов (замена MD RAID)      

Минусы LVM:   


Cуществует 3 группу абстракции:
1. PV (Physical Volume) — физические тома (это могут быть разделы или целые «неразбитые» диски)
2. VG (Volume Group) — группа томов (объединяем физические тома (PV) в группу, создаём единый диск, который будем дальше разбивать так, как нам хочется)
3. LV (Logical Volume) — логические разделы, собственно раздел нашего нового «единого диска» ака Группы Томов, который мы потом форматируем и используем как обычный раздел, обычного жёсткого диска.  
PE (Physical Extend) - минимальная единица диского пространства которой может управлять LVM. По дефолту размер блока 4MB, но его можно менять в зависимости от задачи.

 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Диски/Картинки/LVM_abstrakcia.PNG">
</p>


Чтобы диск мог быть использован с LVM , его нужно инициализировать для использования LVM. В процессе инициализации на диск записываются метаданные, которые сообщают системе, что диск будет использоваться для LVM.
## Работа с LVM
### PV

**pvs** - посмотреть наличие физических томов
```
PV         VG        Fmt  Attr PSize   PFree
/dev/sda3  ubuntu-vg lvm2 a--  <13.25g <3.25g
```
**pvdisplay** - более подроная информация о физических томах
```
shmel@lvm:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <13.25 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              3391
  Free PE               831
  Allocated PE          2560
  PV UUID               ueP84H-3KJt-eJj7-PC0W-0ouO-IOoE-v31EWX
```
 **pvcreate**  - добавить том в подсистему LVM
```
shmel@lvm:~$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
```
### VG
**vgs** - посмотреть созданные LVM группы
```
shmel@lvm:~$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <13.25g <3.25g
```
**vgdisplay** - показать более детальную информацию  

**vgcreate** - создать новую группу LVM
```
shmel@lvm:~$ sudo vgcreate vg_test /dev/sdb1
  Volume group "vg_test" successfully created
```
### VL
**vls** - посмотреть созданные LVM логические тома
```
shmel@lvm:~$ sudo lvs
  LV        VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- 10.00g    
```
**vldisplay** - показать более детальную информацию   

**lvcreate**  - создать логический диск
```
shmel@lvm:~$ sudo lvcreate -L 1024M -n lv_test vg_test
  Logical volume "lv_test" created.
```
**mkfs.ext4** - поставить файловую систему на новосозданный логический диск
```
shmel@lvm:~$ sudo mkfs.ext4 /dev/vg_test/lv_test
```
**mount** - примонтировать созданный логический диск к файловой системе
```
shmel@lvm:~$ sudo mount /dev/vg_test/lv_test  /mnt/lv1/
```

### Расширение пространства
**lvextend** - раширить логический том
```
shmel@lvm:~$ sudo lvextend -L +100M /dev/vg_test/lv_test (если добавить опцию -r , то LVM сразу пересчитает файловую систему)
  Size of logical volume vg_test/lv_test changed from 1.00 GiB (256 extents) to 1.39 GiB (356 extents).
  Logical volume vg_test/lv_test successfully resized.
```
**resize2fs** - пересчитать файловую систему после изменений
```
shmel@lvm:~$ sudo resize2fs /dev/vg_test/lv_test
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/vg_test/lv_test is mounted on /mnt/lv1; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg_test/lv_test is now 364544 (4k) blocks long.
```
**vgextend** - расширить группу путем добавления физических томов
```
shmel@lvm:~$ sudo pvcreate /dev/sdb2
  Physical volume "/dev/sdb2" successfully created.
shmel@lvm:~$ sudo vgextend vg_test /dev/sdb2
  Volume group "vg_test" successfully extended
```
### Уменьшение пространства
Для того чтобы уменьшить пространство логического тома, его в любом случае нужно размонировать   
**umount** - размонтировать 
```
shmel@lvm:~$ sudo umount /mnt/lv1
```
**e2fsck** - выполнит првоерку файловой системы на ошибки
```
shmel@lvm:~$ sudo e2fsck -f /dev/vg_test/lv_test
e2fsck 1.47.0 (5-Feb-2023)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vg_test/lv_test: 75/98304 files (0.0% non-contiguous), 22893/364544 blocks
```
```
shmel@lvm:~$ sudo resize2fs /dev/vg_test/lv_test 900M указываем сколько будет занимать места наша файловая система
resize2fs 1.47.0 (5-Feb-2023)
Resizing the filesystem on /dev/vg_test/lv_test to 230400 (4k) blocks.
The filesystem on /dev/vg_test/lv_test is now 230400 (4k) blocks long.
```
**lvreduce** - изменение размера логического тома в LVM
```
shmel@lvm:~$ sudo lvreduce -L -900M /dev/vg_test/lv_test - указываем сколько места будет занимать наш логический том
  WARNING: Reducing active logical volume to 524.00 MiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce vg_test/lv_test? [y/n]: y
  Size of logical volume vg_test/lv_test changed from 1.39 GiB (356 extents) to 524.00 MiB (131 extents).
  Logical volume vg_test/lv_test successfully resized.
```
Мы можем столкнуться с проблемой, что количество блоков нашей файловой системы может не совпасть к количеством блоков нашего физического тома
```
shmel@lvm:~$ sudo e2fsck -fy /dev/vg_test/lv_test
e2fsck 1.47.0 (5-Feb-2023)
The filesystem size (according to the superblock) is 230400 blocks
The physical size of the device is 134144 blocks
Either the superblock or the partition table is likely to be corrupt!
```
В таком случае лучшее, что мы можем сделать это вернуть как было
```
shmel@lvm:~$ sudo lvextend -L +900M /dev/vg_test/lv_test
  Size of logical volume vg_test/lv_test changed from 524.00 MiB (131 extents) to 1.39 GiB (356 extents).
  Logical volume vg_test/lv_test successfully resized.
shmel@lvm:~$ sudo e2fsck -fy /dev/vg_test/lv_test
e2fsck 1.47.0 (5-Feb-2023)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vg_test/lv_test: 75/65536 files (0.0% non-contiguous), 20708/230400 blocks
```
```
shmel@lvm:~$ sudo mount /dev/vg_test/lv_test /mnt/lv1 примонтировать диск обратно к точке монтирования
```
### Перенос информации ( Добавление нового жесткого диска и отключение старого)
**pvmove** - переносит данные (екстенты) с одного диска на другой
```
shmel@lvm:~$ sudo pvmove /dev/sdb1 ( тут мы указываем, только откуда мы хотим перенести и далее LVM сам размажет данные по свободному пространству
```
```
shmel@lvm:~$ sudo pvmove /dev/sdb1 /dev/sdb2 -в данном случае мы явно указываем куда хотим перенести данные
```
**vgs** -o+devices - покажет как используются наши диски
```
shmel@lvm:~$ sudo vgs -o+devices
  VG        #PV #LV #SN Attr   VSize   VFree  Devices
  ubuntu-vg   1   1   0 wz--n- <13.25g <3.25g /dev/sda3(0)
  vg_test     2   1   0 wz--n-   2.99g  1.60g /dev/sdb1(0)
```
**vgreduce** - удалит  (освободит) один из выбранных логических дисков
```
shmel@lvm:~$ sudo vgreduce vg_test /dev/sdb2
  Removed "/dev/sdb2" from volume group "vg_test"
```

## LVM Professional
### LVM snapshots
Суть snapshots снять бэкап данных, когда идет постоянная запись/чтение на диск (база данных к примеру) т.е файлы постоянно изменяются.    
Снэпшот позволяет заморозить данные на новом логическом томе. Данные не копируются как таковые, а просто создается карта соответсвия данных на новом логическом томе старым данным     

#### Созданием Snapshot
1. Делаем snapshot
```
lvcreate --snapshot test_vg/lv01 -n s_lv01 -L 100M
```
где:
- lvcreate Создаёт новый логический том (LV) (под снапшот)
- --snapshot -флаг указывает, что создаётся снимок (snapshot) LV
- test_vg/lv01 - Оригинальный логический том, для которого создаётся снимок
- -n s_lv01 - Назначает имя s_lv01 для нового снапшота
- -L 100M Выделяет 100 MB места под снапшот
2. Монтируем snapshot
```
mount /dev/test_vg/s_lv01 /mnt/01_s
```
#### Merge Snapshot
1. нужно отмонтировать ориганальный lv и lv snapshot
```
umount /mnt/01
umount /mnt/01_s/
```
2. Делаем слияние Merge
```
lvconvert --merge /dev/mapper/test_vg-s_lv01
```
3. Монтируем обратно vl
```
mount /dev/mapper/test_vg-lv01 /mnt/01
```
Посли слияния снапшот удалится автоматически

#### Удаление snapshot
После того, как мы поняли что snapshot нам более не нужен, удаляем его, так как он подтормаживает систему и разрастается в объемах:    
1. Отмонтируем
```
umount  /mnt/01_s
```
2. Удалим логический раздел
```
lvremove /dev/test_vg/s_lv01
```
### Кэширование Volume Group

Кэш-пул в LVM используется для ускорения работы с данными, размещёнными на основном логическом томе. Это создаёт систему кэширования, где часто используемые данные хранятся на быстром устройстве (например, SSD), а менее часто используемые — на медленном устройстве (например, HDD).      

Процесс работает следующим образом:

1. Основной том (Volume): хранит все данные, но работает с медленным диском.
2. Кэш-пул: быстрый диск (например, SSD), где сохраняются горячие данные (часто используемые). Эти данные быстрее читаются и записываются.
3. Управление кэшем: LVM автоматически решает, какие блоки данных стоит хранить в кэше, и что нужно выгружать обратно на основной диск.

Далее представим, что у нас есть медленные диски HDD и быстрый диск SSD - его мы будем использовать как кеш, тоесть часто используемая информация при чтение/записи c медленного HDD будет кешироваться на более быстром SDD:
1. Cоздаем кеш раздел
```
lvcreate  --type cache-pool test_vg -n cache -L 10M /dev/sdd
```
- lvcreate		Создаёт новый логический том (LV).
- --type cache-pool		Указывает, что создаётся кэш-пул (cache-pool). Это специальный тип тома в LVM, используемый для кэширования данных.
- test_vg Указывает, что кэш-пул будет создан в группе томов test_vg.
- -n Имя тома	Указывает имя нового тома как cache.
- -L 10M	Размер	Указывает размер кэш-пула, в данном случае это 10 MB.
- /dev/sdd	Physical Volume (PV)	Указывает устройство, на котором будет создан кэш-пул. В этом случае, это физический диск или раздел /dev/sdd.

2. Связываем с основным томом HDD
```
lvconvert --cache --cache-pool test_vg/cache test_vg/lv02
```` 
3. Если нужно отключить кэш
```
lvconvert --uncache test_vg/lv01
```

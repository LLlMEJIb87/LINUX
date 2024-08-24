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


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
4. Сжимаем (Сжатие файлов будет работать только с файлами, которые были добавлены после включение настройки сжатия)
- Скачаем один и тот же текстовый файл во все пулы:
```
for i in {1..4}; do wget -P /test_$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```
- Смотрим разницу
```
zfs list
NAME     USED  AVAIL  REFER  MOUNTPOINT
test_1  21.7M  9.18G  21.6M  /test_1
test_2  17.7M  9.19G  17.6M  /test_2
test_3  10.9M  9.19G  10.7M  /test_3
test_4  39.4M  9.16G  39.3M  /test_4
```
```
zfs get all | grep compressratio | grep -v ref
test_1  compressratio         1.81x                  
test_2  compressratio         2.23x                  
test_3  compressratio         3.65x                  
test_4  compressratio         1.00x                  
```
Наиболее эффективном в данном случае оказался алгоритм  gzip-9      

## Задача №2 Определение настроек пула
1. Скачиваем архив в домашний каталог и разархивируем
```
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download' 
tar -xzvf archive.tar.gz
```
2. В архиве лежит некий виртуальный пул созданный с помощью zfs на другом устройстве, проверим, возможно ли импортировать данный каталог в пул:
```
root@lvm:~# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
        (Note that they may be intentionally disabled if the
        'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
        some features will not be available without an explicit 'zpool upgrade'.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
```
Вывод показывает:   
- pool: otus — найден пул с именем otus.
- id: 6554193320433390805 — уникальный идентификатор пула.
- state: ONLINE — пул в рабочем состоянии.    

Так же в выводе есть информация, что импортируем пул был создан старой версией zfs и предлагает обновить его.    
3. Делаем импорт пула OTUS
```
zpool import -d zpoolexport/ otus
```
Проверяем
```
root@lvm:~# zpool status
  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```
4. Обновляем
```
zpool upgrade otus
This system supports ZFS pool feature flags.

Enabled the following features on 'otus':
  redaction_bookmarks
  redacted_datasets
  bookmark_written
  log_spacemap
  livelist
  device_rebuild
  zstd_compress
  draid
  zilsaxattr
  head_errlog
  blake3
  block_cloning
  vdev_zaps_v2

```
5. Далее определяем настройки пула
```
zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      12130510563720500937           -
otus  autotrim                       off                            default
otus  compatibility                  off                            default
otus  bcloneused                     0                              -
otus  bclonesaved                    0                              -
otus  bcloneratio                    1.00x                          -
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
otus  feature@redaction_bookmarks    enabled                        local
otus  feature@redacted_datasets      enabled                        local
otus  feature@bookmark_written       enabled                        local
otus  feature@log_spacemap           active                         local
otus  feature@livelist               enabled                        local
otus  feature@device_rebuild         enabled                        local
otus  feature@zstd_compress          enabled                        local
otus  feature@draid                  enabled                        local
otus  feature@zilsaxattr             enabled                        local
otus  feature@head_errlog            active                         local
otus  feature@blake3                 enabled                        local
otus  feature@block_cloning          enabled                        local
otus  feature@vdev_zaps_v2           enabled                        local
```
Так же можно посомтреть отдельно интересующие нас параметры:
- Посмотреть свободное пространство в пуле
```
zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
```
- Посмотреть включен ли параметр readonly
```
zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
```
- Посмотреть размер блока, который zfs использует при записи
```
zfs get recordsize otus
```
- Посмотреть тип сжатия
```
zfs get compression otus
```

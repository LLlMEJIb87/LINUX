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

Так же в выоде есть информация, что импортируем пул был создан старой версией zfs и предлагает обновить его.    
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

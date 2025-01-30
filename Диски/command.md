**df** (аббревиатура от disk free) — утилита в UNIX и UNIX-подобных системах, показывает список всех файловых систем по именам устройств, сообщает их размер, занятое и свободное пространство и точки монтирования
- ключ -h -покажет размер файловых систем в читаемом виде (G,KB)
- ключ -T -покажет тип файловой системы (EXT)
- ключ -i -покажет кол-во свободных inode
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Диски/df.PNG">
</p>

___

**du** (сокращенно от disk usage) — покажет занятое пространство файлом или директорией. Используют с ключами -sh и далее путь к папке.
- ключ -h -покажет размер файловых систем в читаемом виде (G,KB)
- ключ -s -суммирует размер всех файлов 
___

**fdisk** -l /dev/sda - покажет информацию о физическом диске.
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Диски/fdisk.PNG">
</p>

___

**lsblk** - выведет список всех блочных устройтсв (если диск только установлен в систему в выоде этой команды он будет отображен)
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Диски/lsblk.PNG">
</p>

___

**tune2fs -l /dev/sda1** - покажет детальную  информацию о файловой системе
```
root@vm-nginx:~# tune2fs -l /dev/sda1
tune2fs 1.46.5 (30-Dec-2021)
Filesystem volume name:   cloudimg-rootfs
Last mounted on:          /
Filesystem UUID:          b796ff2c-f407-4fbc-8eba-3752ca683ea4
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent 64bit flex_bg sparse_super large_file huge_file dir_nlink extra_isize metadata_csum
Filesystem flags:         signed_directory_hash 
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              5120000
Block count:              10485499
Reserved block count:     0
Overhead clusters:        340108
Free blocks:              9489495
Free inodes:              4973456
First block:              0
Block size:               4096
Fragment size:            4096
Group descriptor size:    64
Reserved GDT blocks:      251
Blocks per group:         32768
Fragments per group:      32768
Inodes per group:         16000
Inode blocks per group:   1000
Flex block group size:    16
Filesystem created:       Tue Dec 17 02:15:37 2024
Last mount time:          Wed Jan 29 22:22:28 2025
Last write time:          Wed Jan 29 12:47:24 2025
Mount count:              9
Maximum mount count:      -1
Last checked:             Tue Dec 17 02:15:37 2024
Check interval:           0 (<none>)
Lifetime writes:          7299 MB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:               256
Required extra isize:     32
Desired extra isize:      32
Journal inode:            8
Default directory hash:   half_md4
Directory Hash Seed:      41d198de-b74c-4e43-9229-c25927f9456f
Journal backup:           inode blocks
Checksum type:            crc32c
Checksum:                 0xbdba1d19
```

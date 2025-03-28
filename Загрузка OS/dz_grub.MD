# Домашняя работа на тему загрузка системы
## Задание №1 Включить отображение меню Grub
1. редактируем конфигурационный файл grub
```
nano /etc/default/grub


#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```
2. Обновляем конфигурацию загрузчика
```
update-grub
```
3. Проверяем
```
shutdown -r now
```
  <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0%20OS/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_grub.PNG">
</p>

## Задание №2 Попасть в систему без пароля
Переходим в меню изменения параметра загрузки, нажимаем **e** в окне загрузчика
Способ. 1: init=/bin/bash
1. В конце строки, начинающейся с linux, добавляем init=/bin/bash 

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0%20OS/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_grub2.PNG">
</p>

2. ctrl-x
3. Перемонтируем файловую систему в режиме Read-Write
```
mount -o remount,rw /
```
Способ 2. Recovery mode
1. В меню загрузчика выбираем Advandced options
2. Выбираем Recovery mode
3. В меню восстановления включаем поддержку сети Network
3. Выбираем пункт root

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0%20OS/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_grub4.PNG">
</p>

Задание №3 Переименовать VG
1. Смотрим текущее название volume group
```
root@lvm:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree 
  ubuntu-vg   1   2   0 wz--n- <23.00g <5.00g
```
2. Меняем название
```
vgrename ubuntu-vg ubuntu-otus
```
3. Редактируем конфигурационный файл grub
```
nano /boot/grub/grub.cfg
```
меняем все строки содержащие старое название группы ubuntu--vg на новое ubuntu--otus     
4. Перезагружаемся и проверяем
```
root@lvm:~# vgs
  VG          #PV #LV #SN Attr   VSize   VFree 
  ubuntu-otus   1   2   0 wz--n- <23.00g <5.00g
```

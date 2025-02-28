# Домашняя работа на тему загрузка системы
Задание №1 Включить отображение меню Grub
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

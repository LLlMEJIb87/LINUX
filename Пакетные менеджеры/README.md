# Пакетные менеджеры
_ _ _
__APT (Advanced Packaging Tool)__ — это набор утилит для управления программными пакетами в операционных системах, основанных на Debian.   
```
-update - перечитывает списки из репозитория,  обеспечивает наличие последней информации из всех настроенных репозиториев перед установкой или обновлением пакетов.
-upgrade -  используется для обновления установленных пакетов в системе до их последних доступных версий
-install - установить пакет
-reinstall- переустановить пакет
-remove - удалить пакет
-purge - удалит пакет и конфигурационные файлы
-download - скачать .deb пакет
-source - скачать исходник
```

__dpkg__ - менеджер пакетов, который работает строго в рамаках нашей машины
```
-- list - посомтреть список установленных пакетов
-l name - посмотреть информацию о пакете
-i name.deb - установить скаченный пакет
```

**wget** https://download.virtualbox.org/virtualbox/7.1.4/virtualbox-7.1_7.1.4-165100~Ubuntu~noble_amd64.deb - скачать пакет напрямую с ресурса   
Далее переходим в /etc/apt и настраиваем обновление для этого пакета с заданного нами репозитория
```
/etc/apt/sources.list.d$ sudo touch vbox.list - создаем файл для указания откуда мы будем обновлять virtual box
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg --dearmor  - устанавливаем ключ для возможности обновления с ресурса
deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian noble contrib  - говорит откуда скачивать обновление
apt update - перечитываем списки репозиториев
apt upgrade virtualbox - обновляем ПО
```

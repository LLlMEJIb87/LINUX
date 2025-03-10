# Домашнее задание на темму пакетных менеджеров.
## Задача №1 Создать свой .deb пакет
 Устанавливаем набор утилит, которые потребуется в процессе работы
```
apt update && sudo apt install -y wget dpkg-dev debhelper build-essential cmake gcc git
```
Для примера возьмем пакет Nginx и соберем его с дополнительным модулем ngx_brodli     
1.  Загрузим SDEB пакет Nginx для дальнейшей работы над ним:
```
mkdir deb && cd deb
nano /etc/apt/sources.list.d/ubuntu.sources
#Добавляем секцию для возможности скачать SDEB пакет Nginx
Types: deb deb-src
URIs: http://archive.ubuntu.com/ubuntu
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
#загружаем
apt update
apt source nginx
```
2. Установим зависимости для сборки пакета Nginx
```
apt build-dep nginx
```
3. Загружаем исходный  код модуля ngn_brodli
```
cd ~
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
```
4. Собираем модуль ngx_brotli
```
cd ~/ngx_brotli/deps/brotli
mkdir out && cd out
```
- настраиваем будущую сборку
```
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
```
- компилируем
```
cmake --build . --config Release -j 2
```
5. Модифицируем конфигурацию сборки Nginx, добавляя модуль brotli
```
cd ~/deb/nginx-1.24.0
#Правим конфигурационный файл
nano debian/rules
#Добавляеем в конфигурацию bin_configure_flags следующее
--add-module=$(HOME)/ngx_brotli
```
6. Собираем пакет
```
cd /deb/nginx-1.24.0
debuild -us -uc
```
7. Устанавливаем собранный пакет
```
dpkg -i ~/deb/nginx-common_1.24.0-2ubuntu7.1_all.deb ~/deb/nginx_1.24.0-2ubuntu7.1_amd64.deb
```
Проверяем
```
root@lvm:~/deb/nginx-1.24.0# nginx -V 2>&1 | grep -o -- "--add-module=/root/ngx_brotli"
--add-module=/root/ngx_brotli
```

## Задача №2 Создать свой репозиторий и разместить там ранее собранные DEB пакет
1. Создаем каталог repo и скопируем наш ранее собранный пакет
```
mkdir -p /var/www/html/repo
cp ~/deb/*.deb /var/www/html/repo
```
2. Генерируем список пакетов для APT
```
dpkg-scanpackages . /dev/null | gzip -9 > Packages.gz
```
3. Включаем автоиндексацию каталога в Nginx
```
nano /etc/nginx/sites-available/default
```
Вносим
```

        location /repo/ {
        autoindex on;   # Включаем автоиндексацию файлов
        root /var/www/html;  # Указываем путь к репозиторию
```
Проверяем синтаксис:
```
root@lvm:/var/www/html/repo# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Перезапускаем Nginx
```
root@lvm:/var/www/html/repo# systemctl restart nginx
```
Проверяем:
```
root@lvm:/var/www/html/repo# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="Packages.gz">Packages.gz</a>                                        10-Mar-2025 01:46                4586
<a href="libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7.1_amd64.ddeb">libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7...&gt;</a> 10-Mar-2025 01:45               36272
<a href="libnginx-mod-http-geoip_1.24.0-2ubuntu7.1_amd64.deb">libnginx-mod-http-geoip_1.24.0-2ubuntu7.1_amd64..&gt;</a> 10-Mar-2025 01:45               21834
...
```
4. Добавить репозиторий в систему
- создаим файл
```
touch /etc/apt/sources.list.d/custom-repo.list
```
- добавим в него
```
nano/etc/apt/sources.list.d/custom-repo.list

deb [trusted=yes] http://localhost/repo/ ./
```
- обновим репозиторий
```
apt update
```

5. Добавим пакет в наш репозиторий
```
cd /var/www/html/repo/
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
```
6. Обновим список пакетов в репозитории
```
apt update
```
Проверяем
```
root@lvm:/var/www/html/repo# apt-cache policy percona-release
percona-release:
  Installed: 1.0-30.generic
  Candidate: 1.0-30.generic
  Version table:
 *** 1.0-30.generic 500
        500 http://localhost/repo ./ Packages
        100 /var/lib/dpkg/status
     1.0-29.generic 500
        500 http://repo.percona.com/prel/apt noble/main amd64 Packages
     1.0-28.generic 500
        500 http://repo.percona.com/prel/apt noble/main amd64 Packages
```
Устанавливаем
```
apt install percona-release
```

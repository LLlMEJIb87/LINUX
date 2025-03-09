# Домащнее задание на темму пакетных менеджеров.
## Задача №1 Создать свой .deb пакет
 Устанавливаем набор утилит, которые потребуется в процессе работы
```
sudo apt update && sudo apt install -y wget dpkg-dev debhelper build-essential cmake gcc git
```
Для примера возьмем пакет Nginx и соберем его с дополнительным модулем ngx_broli     
1.  Загрузим SDEB пакет Nginx для дальнейшей работы над ним:
```
mkdir deb && cd deb
nano /etc/apt/sources.list.d/ubuntu.sources
#Добавляем секцию для возможности скачать SDEB пакет Nginx
Types: deb deb-src
URIs: http://archive.ubuntu.com/ubuntu
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
#загружаем
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
cd /ngx_brotli/deps/brotli
mkdir out && cd out
```
- настраиваем будущую сборку
```
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
```
- компилируем
```
cmake --build . --config Release -j 2 --target brotlienc
```
5. Модифицируем конфигурацию сборки Nginx, добавляя модуль brotli
```
cd /deb/nginx-1.24.0
#Правим конфигурационный файл
nano debian/rules
#Добавляеем в конфигурацию bin_configure_flags следующее
--add-module=/home/shmel/ngx_brotli
```
6. Собираем пакет
```
cd /deb/nginx-1.24.0
debuild -us -uc
```
7. Устанавливаем собранный пакет
```
cd ~
mkdir ngnix_1.24.0_brotli
cp ~/deb/nginx_1.24.0-2ubuntu7.1_amd64.deb ~/ngnix_1.24.0_brotli
dpkg -i nginx_1.24.0-2ubuntu7.1_amd64.deb nginx_1.24.0-2ubuntu7.1_amd64.deb
```

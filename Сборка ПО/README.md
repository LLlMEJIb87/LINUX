# Сборка ПО их изходного кода
Что такое сборка?
1. Получение исходных кодов
2. Конфигурация (опции)
3. Компиляция программы из исходных кодов
4. Установка

## Сборка WRK
1. Устанавливаем пакет программа для сборки ПО их изходников
```
apt install build-essential
```
2. Копируем исходник ПО например с GitHub
```
git clone https://github.com/wg/wrk.git
```
3. **make** — утилита предназначенная для автоматизации преобразования файлов из одной формы в другую. Правила преобразования задаются в скрипте с именем Makefile, который должен находиться в корне рабочей директории проекта. Сам скрипт состоит из набора правил, которые в свою очередь описываются:
- целями (то, что данное правило делает);
- реквизитами (то, что необходимо для выполнения правила и получения целей);
- командами (выполняющими данные преобразования).

То есть, правило make это ответы на три вопроса:
```
{Из чего делаем? (реквизиты)} ---> [Как делаем? (команды)] ---> {Что делаем? (цели)}
```
В данном случае запустив утилиту make мы компилируем файл из исходного кода ( написанную на высокоуровневом языке) в исполняемый - понятный компьютеру. 
4. checkinstall

## Cборка Nnix + boringssl + brotli
1. Устанавливаем набор программ, который потребуется для компиляции
```
sudo apt install git wget cmake curl ninja-build libunwind-dev golang gnupg2 ca-certificates lsb-release ubuntu-keyring checkinstall
```
2. Качаем исходник библиотеки boringssl
```
git clone https://boringssl.googlesource.com/boringssl
```
3. Для сборки указываем использование компилятора Gninja
```
сmake -GNinja -B build -DCMAKE_BUILD_TYPE=Release
```
4. Запускаем компилирование
```
ninja -C build
```
5. Cкачиваем cборку Nginx
```
https://nginx.org/download/nginx-1.27.3.tar.gz
```
6. Распоковываем
```
tar -xzvf nginx-1.27.3.tar.gz
```
7. Качаем исходник brotli
```
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
```
8. Идем в
```
cd ngx_brotli/deps/brotli
```
и создаем директорию для сборки
```
mkdir out && cd out
```
9. Подготавливаем файл для сборки
```
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
```
10. Собираем библиотеку
```
cmake --build . --config Release --target brotlienc
```
и выходим из дириктории
cd ../../../..
11. устанавливаем переменное окружение для сборки brotli
```
cd nginx-1.27.3/
export CFLAGS="-m64 -march=native -mtune=native -Ofast -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections"
export LDFLAGS="-m64 -Wl,-s -Wl,-Bsymbolic -Wl,--gc-sections"
```
12. Идем в Nginx и делаем свой конфиг (указываем подготовленные ранее модули) сущетсвующий конфиг можно посмотреть nginx -V
```
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_perl_module=dynamic --with-threads --with-stream --with-stream_ssl_module --with-stream_geoip_module=dynamic --with-http_slice_module --with-mail --with-mail_ssl_module --with-file-aio --with-http_v2_module --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -DTCP_FASTOPEN=23' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' --with-debug --with-http_v3_module --with-cc-opt=-I../boringssl/include --with-ld-opt='-L../boringssl/build/ssl -L../boringssl/build/crypto' --add-module=/home/shmel/ngx_brotli
```
13 запускаем процесс компиляции
```
make
```
14. Создаем deb пакет из подготовленного ранее бинарника
```
sudo checkinstall --pkgname=nginx --pkgversion=1.27.3 --nodoc
```

# DOCKER
_ _ _
Докер это программная платформа для контейнеризации приложений в OS.

## Архитектура
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Docker_architektura.PNG">
</p>


**Компоненты** **Docker**
- Сервер (демон dockerd)
- Клиент (команда docker). Команды по операциям с образами, сетями, томами и контейнерами
- Клиент и сервер взаимодействует по REST API
   

**Объекты** **Docker**
1. Образы (images) — дистрибутивы для контейнеров
- Есть открытый реестр (registry) Docker Hub https://hub.docker.com/
- Можно собираты собственные образы на базе существующих
- Состоят из слоёв (layers)
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/sloi.PNG">
</p>

2. Контейнеры (containers) — развёрнутые, рабочие образы
3. Сети (networks) — передача данных
4. Тома (volumes) — хранение данных


## Установка Docker
https://docs.docker.com/engine/install/ubuntu/  - документауия по установки docker на OS
1. Настроим репозиторий
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Установка пакета docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Убедимся, что установка прошла успешно, загрузим тестовый контейнер
```
sudo docker run hello-world
```
4. Сделаем автозагрузку ПО
```
systemctl enable --now docker
```  
5. Чтобы запускать Docker без sudo, добавьте себя в группу docker
```
sudo usermod -aG docker $(whoami)
```
- docker --version - проверка версии
- systemctl status docker - посмотреть состояние приложения

## Работа с Docker
### Работа с контейнерами

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/zapusk_konteinera.PNG">
</p>

- Cоздать  контейнер
```
docker run -d -p 80:80 --name nginx_test nginx # d - demon
```
- остановить контейнер
```
docker stop nginx_test
```
- Запустить контейнер
```
docker start nginx1
```
- остановить и запустить контейнер
```
docker restart nginx1
```
- посмотреть запущенные контейнеры 
```
docker ps # -a покажет все контейнеры
``` 
- удалить контейнер
```
docker rm <container_id>
```
- информация об контейнере
```
docker inspect nginx_test 
```
- Посмотреть логи контейнера
```
docker logs <container_id>  # -r будет показвать логи в реальном времени
```
__Для работы в контейнере в него нужно "провалиться"__
```
docker exec -it nginx_test bash
```

### Работа с образами
- список образов, которые у нас есть ( )
```
docker images ls
```
- Загрузка образа из Docker Hub
```
docker pull nginx
``` 
- удаление образа
```
docker rmi nginx
```

#### Cборка образа c помощью Dockerfile
__Dockerfile__ - текстовый файл, содержащий последовательность команд, которые Docker использует для создания образа.    


__Dockerfile описывает:__
- Какой базовый образ использовать
- Какие зависимости установить
- Какие файлы добавить
- Какие команды выполнять при запуске контейнера


<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/docker_file.PNG">
</p>

__Пример базового Dockerfile__
```
#Используем официальный образ Python
FROM python:3.9
#Копируем файлы приложения внутрь контейнера
COPY app.py /app/app.py
#Устанавливаем рабочую директорию
WORKDIR /app
#Запускаем приложение
CMD ["python", "app.py"]
```
__Пример продвинутого Dockerfile__
```
FROM ubuntu:latest
RUN apt update && apt install -y nginx curl
COPY nginx.conf /etc/nginx/nginx.conf
WORKDIR /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
__Cборка образа__     
 После создания Dockerfile можно собрать образ с помощью команды
```
# -t my-nginx – задает имя образа
# . – указывает, что Dockerfile находится в текущей директории
docker build -t my-nginx .
docker images
```
__Cборка образа__    
После сборки можно запустить контейнер
```
docker run -d -p 8080:80 my-nginx
```
Теперь Nginx будет доступен на порту 8080    



- просмотр списка томов
```
docker volume ls
```
- Удаление тома
```
docker volume rm myvolume
```

## Docker Volumes

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/docker_hranenie.PNG">
</p>

Docker Volumes — это механизм хранения данных в Docker, который позволяет контейнерам сохранять и обмениваться данными независимо от их жизненного цикла. В отличие от стандартных слоев контейнеров (которые теряются при удалении контейнера), тома (volumes) существуют отдельно и могут переживать перезапуск и удаление контейнеров.    

Какие данные хранить:
- Конфигурационные файлы
- Скачиваемые для использования плагины
- Базы данных
- Настройки пользователей
- Артефакты работы приложения     

**Drivers**    
- overlay2 является предпочтительным драйвером хранилища для всех поддерживаемых в настоящее время дистрибутивов Linux и не требует дополнительной настройки
- В btrfs и zfs драйверах имеются дополнительные опции, такие как создание «снапшотов», но требуют большего обслуживания и настройки      

**Команды**    

- просмотр списка томов
```
docker volume ls
```
- Удаление тома
```
docker volume rm myvolume
```
### Практика
1. Связанные папки (bind mounts) (используются, когда требуется оперативно вносить правки в конфиги, так как папки связаны, то изменения файлов на хосте приведет к изменению файлов в контейнере)
```
docker run -d -it --name devtest -v "$(pwd)"/target:/app node:lts
#или
docker run -d -it --name devtest --mount type=bind,source="$(pwd)"/target,target=/app node:lts
```
2. Тома (volumes) - данные хранятся на диске хоста
```
docker run -d --name devtest -v my-vol:/app node:lts #Запускаем контейнер и привязываем папку app внутри контейнера к volume my-vol
#или
docker run -d --name devtest --mount source=my-vol,target=/app node:lts
# в режиме для чтения 
docker run --name devtest -v volume-name:/path/in/container:ro node:lts
```
3. Tmpfs - хранение в оперативной памяти (используется для ускорения передачи информации)
```
# С mount
docker run -d -it --name tmptest --mount type=tmpfs,destination=/app node:lts
# Короткая форма
docker run -d -it --name tmptest --tmpfs /app node:lts
```
4.  Из Dockerfile
```
FROM ubuntu:latest
RUN mkdir /data
WORKDIR /data
RUN echo "Hello from Volume" > test
VOLUME /data
```

## Docker Network
**Режим** **сети** контейнеров в докер
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Rezhim_seti.PNG">
</p>

__Типы сетевых драйверов Docker__    
- **Bridge Network** - при запуске Docker автоматически создается сеть типа мост по умолчанию. Недавно запущенные контейнеры будут автоматически  подключаться к нему. Мы так же можем создавать пользовательские настраиваемые мостовые сети. Пользовательские мостовые сети превосходят сетевые мосты по умолчанию.
- **Host Network** - удаляет сетевую изоляцию между контейнером и хостом Docker и напрямую использует сеть хоста. Если мы запускаем контейнер, который привязывается к порту 80, и мы используем хост-сеть, приложение контейнера доступно через порт 80 по ip адресу хоста. Означает, что мы не сможем запускать несколько контейнеров на одном хосте, на одном и том же порту, так как порт теперь является общим для всех контейнеров в сети хоста
- **None network** - в сети такого типа контейнеры не подключены ни к одной сети и не имеют доступа к внешней сети или другим контейнерам. Эта сеть используется, когда мы хотим полностью отключить сетевой стек в контейнере
- **Overlay network** - создаёт внутреннюю частную сеть, которая охватывает все узлы, участвующие в кластере swarm. Таким образом, оверлейные сети облегчают обмен данными между сервисом Docker Swarm и автономным контейнером или между двумя автономными контейнерами на разных демонах Docker
- **Macvlan network** - некоторые приложения, особенно устаревшие приложения, отслеживающие сетеовй трафик, ожидают прямого подключения к физической сети, В такой ситуации мы можем использовать сетевой драйвер Macvlan для назначения MAC-адреса виртуальному сетевому адресу интерфейсу каждого контейнера, что делает его физическим интерфейсом, напрямую подключенным к физической сети
- Cписок сетей
```
 docker network ls 
```
- создать отдельную сеть для контейнеров (при добавление контейнеров в эту сеть будут выдаваться адреса из этой сети, все контейнеры которые будут входить в эту сеть будут иметь ip связанность)
```
docker network create test_network
docker container run -d --name test_nginx --network=test_network nginx # создать контейнер и подключить его в ранее созданную сеть
```
- покажет информацию об созданной сети, ip адреса, какие контейнеры к ней привязаны
```
 docker network inspect test_network
```
- подключить контейнер к сети bridge (таким образом мы можем делать связанность контейнеров из разных сетей)
```
docker network connect bridge name_conatiner
```
- Удаление сети
```
docker network rm test_network
```
### Практика
1. Дефолтная bridge-сеть
```
docker run -d --name nginx -p 80:80 nginx #Проброс порта на все интерфейсы, Docker автоматически подключит контейнер к сети bridge 172.17.0.0/16, доступен по порту 80 хоста
docker run -d --name nginx -p 127.0.0.1:80:80 nginx #Ограниченный проброс на 127.0.0.1, доступность только с хостовой машины
docker run -d --name nginx -p 192.168.0.1:80:80 nginx #Контейнерный порт 80 мапится только на 192.168.0.1:80 хостовой машины.
docker network ls
```
2. Кастомная сеть
```
docker network create --driver bridge net(имя сети) #Cоздаем новую сеть bridge
docker run -dt --name n1 --network net nginx:alpine #Запускаем контейнеры и указываем, к какой сети они будут относиться
docker run -dt --name n2 --network net nginx:alpine
docker run -it -d --network host --name nginx nginx #Так мы запускаем контейнер напрямую в сети хоста
docker network inspect net
```
## Docker Compose
Инструмент для описания и управления многоконтейнерными приложениями с помощью файла конфигурации (docker-compose.yml).   

__Преимущества Docker Compose__     
- Позволяет запускать несколько контейнеров одной командой
- Упрощает настройку зависимостей между контейнерами
- Облегчает работу в локальной среде разработки
- Позволяет хранить конфигурацию в одном месте (docker-compose.yml)

__Dockercompose__ представляет из себя yml файл:
```
---
services:
 web:
 image: nginx
 ports:
 - "8080:80"
 depends_on:
 - db
 db:
 image: postgres
 environment:
 POSTGRES_USER: user
 POSTGRES_PASSWORD: password
 POSTGRES_DB: mydatabase
```
__Команды__     
- запуск всех контейнеров
```
docker compose up -d # -d запускает контейнеры в фоновом режиме
```
- Просмотр списка запущенных сервисов
```
docker compose ps
```
- Просмотр логов сервисов
```
docker compose logs -f
```

## Docker registry
Это репозиторий, в котором хранятся образы. Когда разработчики создают приложения, они размещают свои образы в этих репозиториях, откуда их могут скачать другие люди. Есть публичные репозитории, например Docker Hub. А можно создать свой репозиторий, для использования внутри компании или команды.     

Один из видов локального реестра **Harbor** — реестр для Docker-контейнеров с безопасностью «из коробки»  https://goharbor.io/    

**Установка**     

1. Генерируем самоподписанные сертификаты
```
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=RU/ST=Moscow/L=Moscow/O=example/OU=MY company/CN=mycompany.net" -key ca.key -out ca.crt
openssl genrsa -out harbor.mycompany.net.key 4096
openssl req -sha512 -new -subj "/C=RU/ST=Moscow/L=Moscow/O=example/OU=My company/CN=mycompany.net" -key harbor.mycompany.net.key -out harbor.mycompany.net.csr
openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in harbor.mycompany.net.csr -out harbor.mycompany.net.crt
openssl x509 -inform PEM -in harbor.mycompany.net.crt -out harbor.mycompany.net.cert
```
2. Закидываем серверные ключи в каталог
```
sudo cp harbor.mycompany.crt /etc/ssl/certs/
sudo cp harbor.mycompany.key /etc/ssl/certs/
```
3. Скачиваем harbor-offline-installer-v1.8.1.tgz https://github.com/goharbor/harbor/releases
4. Распоковываем
```
tar xzvf harbor-offline-installer-v2.12.2.tgz
```
4. Конфигурим файл harbor.yml ( если его нет, то копируем harbor.yml.tmpl)
```
hostname: 192.168.1.210

https:
# https port for harbor, default is 443
port: 443
# The path of cert and key files for nginx
certificate: /etc/ssl/certs/harbor.mycompany.net.crt
private_key: /etc/ssl/certs/harbor.mycompany.net.key

harbor_admin_password: Harbor12345
```
5. Запускаем установочный скрипт
```
./install.sh
```
__Настраиваем docker на клиенте (у разработчика)__
1. Cоздаем директорию
```
mkdir /etc/docker/certs.d/harbor.mycompany.net/
```
2. Закидываем в этот каталог файлы
```
/etc/docker/certs.d/harbor.mycompany.net/
├── ca.crt
├── harbor.mycompany.net.cert
└── harbor.mycompany.net.key
```
3. Собираем докер образ и кладём его в харбор:
```
docker login
docker build -t harbor.mycompany.net/<project>/<name>:<tag> .
docker push harbor.mycompany.net/<project>/<name>:<tag>
```

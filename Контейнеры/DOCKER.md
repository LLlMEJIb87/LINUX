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
systemctl enable --now docke
```  
5. Чтобы запускать Docker без sudo, добавьте себя в группу docker
```
sudo usermod -aG docker $(whoami)
```
- docker --version - проверка версии
- systemctl status docker - посмотреть состояние приложения

## Работа с Docker
**Команды:**  
- docker ps - посмотреть запущенные контейнеры ( с ключом -a можно посомтреть все контейнеры, в том чилсе не ативные) ( docker rm id/имя - удалить контейнер)
- docker images - список образов, которые у нас есть (  docker rmi hello-world:latest - удалит образ)
- docker pull nginx:1.26.2-alpine-slim - cкачать образ  
- docker system df покажет сколько места занимают контейнеры на диске
- docker info посмотреть информацию о версии и т.п 


### Запуск контейнера

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/zapusk_konteinera.PNG">
</p>

1. Запускаем контейнер
```
 docker run -d --name nginx1 -p 80:80 -v /var/www/html:/usr/share/nginx/html nginx:1.26.2-alpine-slim
```
- docker stop nginx1 - остановить контейнер
- docker start nginx1 - запустить контейнер
- docker restart nginx1 - остановить и запустить
- docker inspect nginx1 - информмация об контейнере
- docker logs <container_id> - логи контейнера
2. Для работы в контейнере в него нужно "провалиться"
```
 docker exec -it nginx1 sh (bash)
```
3. В оболочке контейнера нет многих привычных нам команд, поэтому "выкачиваем" из контейнера нужные файлы в OS и там их редактируем,далее вставляем обратно
- ls -al - покажет файлы
```
sudo docker cp nginx1:/etc/nginx /home/nginx-etc/ - команда запускается из OS, не из докера
```
4. Останавливаем и удаляем контейнер
```
sudo docker stop nginx1
sudo docker rm nginx1
```
5. запускаем контейнер, но монтируем конфиг из ранее созданной папки в OS (куда ранее копировали из докера)
```
sudo docker run -d --name nginx1 -p 80:80 -v /home/nginx-etc/nginx/:/etc/nginx -v /var/www/html:/usr/share/nginx/html nginx:1.26.2-alpine-slim
```
### Работа с образами
- список доступных образов
```
docker images
```

### Работа с томами
- Создание и подключение тома
```
docker volume create myvolume
docker run -v myvolume:/data -it ubuntu bash
```
Теперь файлы в /data контейнера будут сохраняться даже после его удаления
## Docker Network
**Режим** **сети** контейнеров в докер
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Rezhim_seti.PNG">
</p>

```
 docker network ls покажет какие сети есть в докер
```
```
 docker network inspect MYNET покажет информацию об созданной сети, ip адреса, какие контейнеры к ней привязаны
```
```
docker network create MYNET создать отдельную сеть для контейнеров, при добавление контейнеров в эту сеть будут выдаваться адреса из этой сети, все контейнеры которые будут входить в эту сеть будут иметь ip связанность
docker run --name nginx2 --network=MYNET -p 80:80 nginx - создать контейне и определить к какой сети он будет присоединен
```
```
docker network connect bridge name_conatiner - подключить контейнер к сети bridge, таким образом мы можем делать связанность контейнеров из рахных сетей
```
## Сборка образа Docker file
1. Создаем директорию mkdir nginx-docker
2. Cоздаем Dockerfile 
```
cat > Dockerfile
FROM nginx:latest (latest - последняя версия из репозитория докер)
COPY ./index.html /usr/share/nginx/html/index.html
```
3. Получаем индексовый файл
```
 cp /var/www/html/index.html ./
```
4. Делаем сборку
```
docker build -t nginx-custom . (точка - откуда будет браться инофрмация для сборки)
```

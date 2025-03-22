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
docker build -t my_app
```
5. Запуск контейнера из образа
```
docker run my_app
```
### Работа с томами
Прокинуть директорию можно двумя способами:
1. прокинуть директорию в контейнер
```
mkdir nginx_test
docker run -d -v ./:/data --name nginx_test nginx
```
теперь файлы созданные в data контейнера будут сохраняться на хостовой машине  

    
2. Создание и подключение тома
```
docker volume create myvolume
docker run -v myvolume:/data -it ubuntu bash
```
Теперь файлы в /data контейнера будут сохраняться даже после его удаления


- просмотр списка томов
```
docker volume ls
```
- Удаление тома
```
docker volume rm myvolume
```

## Docker Network
**Режим** **сети** контейнеров в докер
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Rezhim_seti.PNG">
</p>

- Cписок сетей
```
 docker network ls 
```
- создать отдельную сеть для контейнеров (при добавление контейнеров в эту сеть будут выдаваться адреса из этой сети, все контейнеры которые будут входить в эту сеть будут иметь ip связанность)
```
docker network create MYNET
```
- покажет информацию об созданной сети, ip адреса, какие контейнеры к ней привязаны
```
 docker network inspect MYNET
```
- подключить контейнер к сети bridge (таким образом мы можем делать связанность контейнеров из разных сетей)
```
docker network connect bridge name_conatiner
```


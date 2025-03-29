# Выполнение домашнего задания по теме Docker
1. Установка пакета Docker на хост машину
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
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
2. Создаём свой кастомный образ nginx на базе alpine
2.1 Создаем рабочую директорию и перейдем в нее
```
mkdir my-nginx && cd my-nginx
```
2.2 Создаём Dockerfile
```
touch Dockerfile
nano Dcokerfile

# Используем базовый образ Alpine с Nginx
FROM nginx:alpine

# Копируем кастомную страницу в папку Nginx
COPY index.html /usr/share/nginx/html/index.html

# Открываем порт 80
EXPOSE 80

# Запускаем Nginx в фоновом режиме
CMD ["nginx", "-g", "daemon off;"]
```
2.3 Создаём кастомную страницу для NGinx
```
touch index.html
nano index.html

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мой кастомный Nginx</title>
</head>
<body>
    <h1>Привет, OTUS!</h1>
    <p>Это мой кастомный образ Nginx на Alpine.</p>
</body>
</html>
```
2.4 Собираем образ
```
docker build -t nginx-otus .
```
2.5 Запускаем контейнер
```
docker run -d -p 80:80 nginx-otus
```
2.6 Проверяем
```
root@vm-dz-upgrade:~/nginx-otus# curl 192.168.1.210
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мой кастомный Nginx</title>
</head>
<body>
    <h1>Привет, OTUS!</h1>
    <p>Это мой кастомный образ Nginx на Alpine.</p>
</body>
</html>
```
3. Определите разницу между контейнером и образом    
- Образ это статический шаблон, который используется для создания контейнеров
- Контейнер это запущенный экземпляр образа, динамический процесс     

4. Ответьте на вопрос: Можно ли в контейнере собрать ядро?      
Собрать (скомпилировать) ядро в контейнере – да, можно. Запустить контейнер с другим ядром – нет, так как он использует ядро хоста.

5. Закидываем наш custom образ на Docker Hub     

5.1 Регестрируемся на https://hub.docker.com/     
5.2 Авторизируемся на хосте
```
docker login
```
5.3 Переименоввываем образ перед пушем
```
docker tag nginx-otus lllmejib87/nginx-otus:latest
```
5.3 Пушим на DockerHub
```
docker push lllmejib87/nginx-otus:latest
```

https://hub.docker.com/repository/docker/lllmejib87/nginx-otus/general

# Nginx+Apache2
_ _ _
 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/nginx%2Bapache.PNG">
</p>
У этих web серверов есть свои достоинства и недостатки, можно взять лучшее от обоих и объеденить. Nginx может поддержваить много запросов на одном процессе и хорошо справляться со статикой, а apache передавать тяжелый динамический контент. В таком случае nginx выступает в роли frontend сервера, а apache в роли backend.   

```
apt install nginx
```
```
apt install apache2
```
После установки apache он не сможет стартануть, так как по дефолту слушает 80 порт, а он уже занят nginx. Для решения этой проблемы требуется сменить порт:   
1. Идем в /etc/apache2 и редактируем  ports.conf - ставим нужный порт (к примеру 8080) всё остальное закоментируем.
2. Идем в /etc/apache2/sites-enabled и редактируем 000-default.conf - <VirtualHost *:8080>   

При установке сервера в папке /var/www/html - появляется файл, это приветсвенная страница на которую мы будем попадать при обращение к серверу по ip через браузер


### Конфигурация проксирования
1. Идем в /etc/nginx/sites-enabled - выбираем файл с конфигом сайта  и вносим следующие дерективы:
```
 location(обрабатывает запросы) / {
                proxy_pass http://localhost:8080; - мы говорим что отправлять запросы на локалхост 8080
                proxy_set_header Host $host; - пердаем заголовок хост, чтобы можно было по доменам ориентироваться
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr; - передаем информацию о реальном ip
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #try_files $uri $uri/ =404;
        }
````
nginx -t - проверяем конфиг    
systemctl reload nginx - перечитываем конфигурацию    
apachectl -t - проверяем конфиг    
systemctl reload apache2 - перечитываем конфигурацию


### Конфигурация балансировки
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Balansirovka.PNG">
</p>

В нашем примере мы сделаем 3 "отдельных" apache2 сервера для 
проксирования с nginx путем добавления 3х отдельных сайтов слушающих разные порты, на самом деле это могут быть 3 отдельных сервера.    
1. Идем в /etc/apache2/sites-available
2. Создаем дополнительные конфиги  cp 000-default.conf 002.conf  cp 000-default.conf 003.conf
3. Кидаем симлинки  на папку sites-available - cистема с этой папки подгружает конфигурацию 
- ln -s /etc/apache2/sites-available/002.conf /etc/apache2/sites-enabled/002.conf
- ln -s /etc/apache2/sites-available/003.conf /etc/apache2/sites-enabled/003.conf
4. В конфиги 002.conf и 003.conf вносим изменения
- указываем порт <VirtualHost *:8081> 
- DocumentRoot /var/www/html1 - указываем на какой файл ссылаться при открытие странички в браузере
5. Далее идем /etc/apache2/ и редактируем ports.conf, говорим слушать дополнительные порты   
Listen 8080   
Listen 8081   
Listen 8082   
6. Далее конфигурируем nginx чтобы он отсылал запросы на разные сервера apache
- идем /etc/nginx/sites-enabled и редактируем конфиг default, создаем доп блок восходящих бекэнд серверов
```
upstream backend {
        server 127.0.0.1:8080 weight=2;
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
}
```
- в блоке location говорим проксировать на ранее созданный блок backend
```
 location / {
                proxy_pass http://backend;
```

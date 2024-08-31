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
После установки apache он не сможет стартануть, так как по дефолту случает 80 порт, а он уже занят nginx. Для решения этой проблемы требуется сменить порт:   
1. Идем в /etc/apache2 и редактируем  ports.conf - ставим нужный порт (к примеру 8080) всё остальное закоментируем.
2. Идем в /etc/apache2/sites-enabled и редактируем 000-default.conf - <VirtualHost *:8080>





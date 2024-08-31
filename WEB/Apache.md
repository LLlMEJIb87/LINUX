# Apache
_ _ _
Apache («Апачи», Apache HTTP Server) — это открытое кросс-платформенное программное обеспечение для размещения и поддержки веб-сервера. https://httpd.apache.org/docs/2.4/en/
```
apt install apache2
```
### Конфигурация
Основной конфиг: /etc/apache2/apache2.conf
- Подключение дополнительных конфигов: IncludeOptional conf.d/*.conf
- Главный сервер (сайт) описан в apache2.conf
- Отдельные сайты: блоки <VirtualHost></VirtualHost>
- Выбор сайта для обработки: директивы Listen и ServerName, ServerAlias
- Разделитель директив: новая строка
- Конфигурация по директориям: блоки <Directory></Directory> и <Files></Files> и файлý .htaccess
- Конфигурация по URL: блоки <Location></Location>
- Директивы наследуются

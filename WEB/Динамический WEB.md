 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/Картинки/din_web_arch_pril.PNG">
</p>

Протоколы взаимодействия:
- Протоколы взаимодействия с сервером приложений
  - CGI (устарел)
  - FastCGI (используется)
  - SCGI
  - WSGI
- Клиентские протоколы (браузер-сервер)
  - AJAX (XHR)
  - WebSocket

**FastCGI**   
- Протокол взаимодействия между веб-сервером и сервером приложений
- Блокирующий доступ
- Используется TCP или UNIX socket
- Пример конфигурации в Nginx:
```
location ~ ^/(status|ping)$ {
allow 127.0.0.1;
deny all;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_index index.php;
include fastcgi_params;
fastcgi_pass unix:/var/run/phpfpm-api.sock;
}
```
FastCGI — принципы работы
- Программа загружается в память в качестве при первом обращении к серверу
- Не требуется процесс интерпретации и подготовки кода к исполнению
- Один и тот же процесс обрабатывает множество запросов
- Высокая производительность
- Большее потребление оперативной памяти

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/Картинки/din_web_fcgi.PNG">
</p>

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/Картинки/din_web_php.PNG">
</p>

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

## Протоколы взаимодействия с сервером приложений

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

**WSGI framework**
- Стандарт взаимодействия между Python-программой, выполняется на стороне сервера, и самим веб-сервером
- Должно быть вызываемым (callable) объектом (обычно это функция или метод)
- Параметры на входе
   - Словарь переменных окружения (environ)
   - Обработчик запроса (start_response)
- Вызывает обработчик запроса с кодом HTTP-ответа и HTTP-заголовками
- Возвращает итерпритируемый объект с телом ответа


**uWSGI**    
- uWSGI - веб-сервер и сервер веб-приложений, первоначально реализованный для запуска приложений Python через протокол WSGI (и его бинарный вариант uwsgi)
- Версия 2.0 поддерживает также запуск веб-приложений Lua, Perl, Ruby и других   

## Клиентские протоколы (браузер сервер)
**AJAX (XHR)**
- Asynchronous Javascript and XML - «асинхронный JavaScript и XML»
- XMLHttpRequest (XMLHTTP, XHR) Ƃ API в JS для отправки HTTP-запросов
- fetch - современная версия API в JS
- Отправка запросов без перезагрузки страницы
- Обновление частей страницы
- Динамическая загрузка (бесконечная прокрутка)
- Данные в формате JSON или HTML

**WebSocket**
- Протокол для долгоживущих соединений поверх TCP
- Соединение устанавливается с помощью HTTP
- Сообщения могут идти в обе стороны
- Схема URL:
  - HTTP: ws://ws.my.site
  - HTTPS: wss://wss.my.site

https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/Картинки/OTUS_Linux_Professional_Динамический_веб-224190-5d5e57.pdf

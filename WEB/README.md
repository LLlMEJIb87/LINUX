**Термины**    
- HTTP (HyperText Transport Protocol) - протокол передачи данных
- HTML (HyperText Markup Language) - язык разметки
- URI (Uniform Resource Identifier) - унифицированный идентификатор ресурса. Позволяет идентифицировать физический ресурс. Ранее URI строго делились на URL и URN. Сейчас см. ссылку (https://www.w3.org/TR/uri-clarification/). Пример: https://habr.com/img/favicon-16.png
- URL (Uniform Resource Locator) - помогает найти какой-либо ресурс
- URN (Uniform Resource Name) - единообразное название ресурса. Может по одному только название дать вам ресурс (абстрактный или физический). Пример: urn:isbn:540609601X


 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/WEB/Картинки/URI.PNG">
</p>


**HTTP-методы**    
- GET - получить содержимое указанного ресурса
- HEAD - получить только заголовки
- POST - отправить данные
- DELETE - удалить указанный ресурс
- OPTIONS - определить параметры сервера
- PUT - обновление
- PATCH - частичное обновление
- TRACE - возврат с трассировкой
- CONNECT - Преобразует соединение запроса в прозрачный TCP/IPтуннель


**HTTP-заголовки**    
- Данные, уточняющие запрос или ответ
- Используются для управления запросом и или ответом
- Кеширование
- Сжатие
- Идентификация
- Аутентификация
- Безопасность
- Свойства запроса/ответа
- Keep-alive соединения
- Cookie



**Коды ответа в HTTP**    
- 1xx: Informational (информационные)
- 2xx: Success (успешно)
- 3xx: Redirection (перенаправление)
   - 301 Moved Permanently («перемещено навсегда»)
   - 302 Found («найдено»)
- 4xx: Client Error (ошибка клиента)
   - 403 Forbidden («запрещено (не уполномочен)»)
   - 404 Not Found («не найдено»)
- 5xx: Server Error (ошибка сервера)
   - 500 Internal Server Error («внутренняя ошибка сервера»)
   - 502 Bad Gateway («плохой, ошибочный шлюз»)
   - 504 Gateway Timeout («шлюз не отвечает»)


  ## HTTPS

HTTPS - расширение HTTP для работы с SSL/TLS     
Версии протоколов: 
- SSL 2.0 (1995Ɓ2011)
- SSL 3.0 (1996)
- TLS 1.0 (1999Ɓ2021)
- TLS 1.1 (2006Ɓ2021)
- TLS 1.2 (2008)
- TLS 1.3 (2018)

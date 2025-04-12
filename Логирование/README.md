# Логирование
_ _ _
**Задачи** **журналирования:**
- Фиксация событий системы и сервисов (syslog)
- Аудит — событий безопасности, инциденты (auth.log)
- Исходные данные для мониторинга после агрегации (access.log)
- Отладка проблем и поиск сбоев (error.log)
- Восстановление действий до указанного момента (binlog)
- Аналитика и поведение пользователей (access.log)
- Сценарий для нагрузочного тестированиā (access.log)

  ## Типы логов
Типы логов
1. Текстовые
- Прямой записи: /var/log/nginx/access.log
- Через rsyslogd: /var/log/syslog или /var/log/messages
2. Бинарные (через systemd-journald): /var/log/journal/
3. База данных (Elasticsearch, MySQL)

  ### Текстовые логи 
- /var/log/syslog или /var/log/messages содержит глобальный системный журнал, в котором пишутся сообщения с момента запуска системы, от ядра Linux, различных служб, обнаруженных устройствах, сетевых интерфейсов и много другого.
- /var/log/auth.log или /var/log/secure — информация об авторизации пользователей, включая удачные и неудачные попытки входа в систему, а также задействованные механизмы аутентификации.
- /var/log/dmesg — драйвера устройств. Одноименной командой можно просмотреть вывод содержимого файла. Размер журнала ограничен, когда файл достигнет своего предела, старые сообщения будут перезаписаны более новыми. Задав ключ --level= можно отфильтровать вывод по критерию значимости    
- /var/log/alternatives.log — Вывод программы update-alternatives, в котором находятся символические ссылки на команды или библиотеки по умолчанию.
- /var/log/anaconda.log — Записи, зарегистрированные во время установки системы.
- /var/log/audit — Записи, созданные службой аудита auditd.
- /var/log/boot.log — Информация, которая пишется при загрузке операционной системы.
- /var/log/cron — Отчет службы crond об исполняемых командах и сообщения от самих команд.
- /var/log/cups — Все, что связано с печатью и принтерами

__Поддерживаемые уровни журналирования (приоритеты):__
- emerg - система неиспользуемая
- alert - действие должно быть произведено немедленно
- crit - условия критичности
- err - условия ошибок
- warn - условия предупреждений
- notice - обычные, но значимые условия
- info - информационный
- debug - отладочные сообщения

```
cat syslog | grep err  - фильтрация логов
```
```
tail -n 10 syslog  - показать последние 10 строк
```
```
tail -f syslog - смотреть лог в реальном времени
```
```
head -n 10 syslog - показать первые 10 строк лога
```
__nano /etc/rsyslog.d/50-default.conf__ - конфигурационный файл в котором можно настроить сбор логов    

__/etc/logrotate.d__  в системе сущетсвует ротация логов, старые логи сохраняются и сжимаются, в этом файле находятся файлы с настройками для каждого сервиса.   

__Ротация логов:__
- Хранение истории
- Сжатие старых логов
- Конфигурация /etc/logrotate.conf, /etc/logrotate.d/*
- Часто настройки ротации создаётся при установке пакета
- Скрипт запуска: /etc/cron.daily/logrotat

  ### Journald
  Современная система логирования   
__/etc/systemd/journald.conf__ - конфигурационный файл journald

__journalctl -b__ - покажет сообщения с ммоента загрузки системы     
__journalctl -b | grep__  фильтруем по слову     
__journalctl --since "2024-10-15 00:00:00"__ - фильтруем по дате    
__journalctl --since "2024-10-15 00:00:00" --until "2024-10-15 08:00:00"__ - фильтруем от и до    
__journalctl - xeu nginx.service__ - посмотреть логи относящиеся к конкретному сервису   
__journalctl -p err -b__ - посмотреть только ошибки с момента последней загрузки    

### dmesg
Команда dmesg показывает кольцевой буфер сообщений ядра — это логи, которые записывает ядро Linux о загрузке системы, обнаружении оборудования, подключении устройств, драйверах и других системных событиях.
```
dmesg -T
```

## Централизованный сбор логов с помощью ELK
__Компоненты__
- **E**lasticsearch — база данных для надёжного хранения документов
- **L**ogstash — обработка данных из логов
- **K**ibana — визуализация данных из Elasticsearch (дашборды)
- Beats — агенты сбора данных для сохранения в ELK-стеке   
  Компоненты могут быть требовательными к оперативной памяти, полный стек — 4 GB минимум

  __Beats__
- Filebeat — основной поставщик данных в ELK
- Heartbeat — проверка сервисов
- Auditbeat — события auditd (безопасность)
- Metricbeat — метрики для мониторинга системы

 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9B%D0%BE%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/Picture/ELK_dvij.PNG">
</p>    

Beats - собирают логи и отправляют   
Logstah - обработка/преобразование логов
Elasticsearch - хранение логов
kibana - визуализация   

### Cеть
 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9B%D0%BE%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/Picture/ELK_network.PNG">
</p>    

## Установка и настройка
### Elasticsearch - база данных
1. Устанавливаем зависимости, так как elasticsearch работает на java
```
sudo apt install default-jdk -y
```
2. Устанавливаем пакет с elasticsearch
```
sudo dpkg -i elasticsearch-8.9.1-amd64.deb
```
3. Прописываем лимит оперативной памяти для java
```
root@elk:/etc/elasticsearch/jvm.options.d# cat > jvn.options
-Xms1g
-Xmx1g
```
4. Настраиваем конфиг elasticsearch в упрощенном режиме
```
nano /etc/elasticsearch/elasticsearch.yml
xpack.security.enabled: false - выключаем
xpack.security.http.ssl:
  enabled: false - выключаем
xpack.security.transport.ssl:
  enabled: false - выключаем

http.host: 0.0.0.0 - если устанволены все нули, то сможем зайти по ip машины
```
5. Стартуем elasticsearch
```
systemctl daemon-reload  - обновляем описание сервисов
systemctl enable --now elasticsearch.service - ставим в автозагрузку и автоматически стартуем
```
6. Elasticsearch слушает 9200 порт, проверить можно следующим образом
```
curl http://localhost:9200
```
### Kibana - визуализация
1. Устанавливаем пакет
```
dpkg -i kibana-8.9.1-amd64.deb
```
2. Меняем настройки
```
nano /etc/kibana/kibana.yml
server.port: 5601 - раскоментировали
erver.host: "0.0.0.0" установили нули, для возможности заходить на него по ip машины
```
3. Стартуем сервис
```
systemctl daemon-reload  - обновляем описание сервисов
systemctl enable --now kibana.service - ставим в автозагрузку и автоматически стартуем
```
4. Kibana слушает 5601 порт
### logstash - обработка логов, посредник который принимает файлы с beats и отправляет в elasticsearch
1. Устанавливаем сервис
```
dpkg -i logstash-8.9.1-amd64.deb
```
2. Меняем настройки
```
nano /etc/logstash/logstash.yml
path.config: /etc/logstash/conf.d указываем путь
```
3. Стартуем сервис
```
systemctl daemon-reload  - обновляем описание сервисов
systemctl enable --now logstash.service - ставим в автозагрузку и автоматически стартуем
```
4. Делаем настройки, которые будут преобразовывать файлы logstash для NGINX
```
cat > /etc/logstash/conf.d/logstash-nginx-es.conf - создаем файл
input {                #input - откуда он получает данные
    beats {
        port => 5400
    }
}

filter {               #filter - что он делает с этими данными
 grok {
   match => [ "message" , "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"]
   overwrite => [ "message" ]
 }
 mutate {
   convert => ["response", "integer"]
   convert => ["bytes", "integer"]
   convert => ["responsetime", "float"]
 }
 date {
   match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
   remove_field => [ "timestamp" ]
 }
 useragent {
   source => "agent"
 }
}

output {             #output - куда мы отправляем данные
 elasticsearch {
   hosts => ["http://localhost:9200"]
   #cacert => '/etc/logstash/certs/http_ca.crt'
   #ssl => true
   index => "weblogs-%{+YYYY.MM.dd}"
   document_type => "nginx_logs"
 }
 stdout { codec => rubydebug }
}
```
### filebeat агент сбора логов
1. Устанавливаем сервис
```
dpkg -i filebeat-8.9.1-amd64.deb
```
2. Настраиваем
```
nano /etc/filebeat/filebeat.yml
после этой строки добавляем запись ниже  #- c:\programdata\elasticsearch\logs\* 
filebeat.inputs:
- type: filestream
  paths:
    - /var/log/nginx/*.log

Коментим эти строки:
#output.elasticsearch:
# hosts: ["localhost:9200"]
Раскоментируем следующие строки:
output.logstash:
hosts: ["localhost:5400"] - меняем порт на 5400
```
3. запускаем сервис
```
systemctl daemon-reload  - обновляем описание сервисов
systemctl enable --now filebeat.service - ставим в автозагрузку и автоматически стартуем
```
4. filebeat слушает порт 5400 - который мы прописали в конфиге

## Проверка и просмотр
1. Можем тестов нагенерить логов, обращася к curl localhost
2. Идем в **kibana** через web на ip машины порт 5061
3. Выбираем Explore on my own
4. discover -> create data view
5. собираем index (это как таблица в базе данных с который мы будем получать данные) -> Name Nginx -> Index Pattern weblogs* -> Timestap @timestamp -> save data view

__Делаем визуализацию__
1. Вкаладка dashboard -> create dashboard
2. Create vizualization     
...    
3. Save - сохранить проект   




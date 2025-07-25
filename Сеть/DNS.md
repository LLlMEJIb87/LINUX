# DNS
**Клиент DNS. Утилиты.**    
```
# Для работы с утилитами необходимо установить пакет bind-utils
$ yum install bind-utils
# Диагностика
$ dig www.otus.ru
$ host www.otus.ru
$ nslookup www.otus.ru
# Пример получения ресурсных записей
$ dig MX www.otus.ru
$ dig TXT www.otus.ru
```

**Ресурсные записи (RR)**
- Записи DNS обладают следующими атрибутами:
  - имя
  - TTL (время жизни в кеше)
  - класс
  - тип
  - значение (или массив значений)


**Типы записей**     
- A — адрес IPv4, соответствующий имени
- АААА — адрес IPv6, соответствующий имени
- CNAME — имя, соответствующее имени (canonical name)
- MX — массив (приоритет и имя) почтовых серверов для домена
- TXT — текстовая информация
- SOA — ключевая запись домена (start of authority)
- NS — имя сервера имён для домена (nameserver)
- PTR — имя, соотвествующее IP-адресу (pointer для in- addr.arpa и ip6.arpa)
- SRV — указание на расположение сервиса


**PTR-запись**    
- Представляет собой “обратную A-запись”
- A-запись нужна для преобразования доменного имени в IP-адрес, PTR — для преобразования IP-адреса в доменное имя
- В PTR-записи IP-адрес записывается в обратном порядке. То есть IP-адрес 11.22.33.44 в обратной зоне будет записан как 44.33.22.11.
- Зачастую прямая и обратная зоны находятся на разных DNS-серверах, так как ресурсы принадлежат разным компаниям (доменное имя покупает компания, а IPадрес принадлежит провайдеру)
- Пример:     
Обратная зона: 10.168.192.in-addr.arpa     
Запись: 1 IN PTR www.otus.ru     
Результат: 1.10.168.192.in-addr.arpa IN PTR www.otus.ru     

## DNS Сервер

**Типы серверов**    
Типы серверов (по свойствам и функциям):       
- главные (primary или master) — авторитетные, хранят главную копию информации о зоне;
- вторичные (secondary или slave) — получают копию информации о зоне с главного или вторичного сервера и работают с ней;
- кеширующие — кешируют ответы на запросы пользователя;
- рекурсивные — выполняют полный поиск по иерархии DNS;
- нерекурсивные — не выполняют полный поиск (не умеют или им запрещено)


**Установка и настройка**    
```
# Ставим сервер bind9
$ yum install bind
# Конфигурационный файл
$ vim /etc/named.conf
# В конфигурационном файле задаем роль сервера, расположение файлов зоны
# Делаем рестарт сервиса
$ systemctl restart named
# Добавляем адрес созданного сервера имен в /etc/resolv.conf
$ vim /etc/resolv.conf
nameserver 1.1.1.1
```

**Пример файла зоны полные имена**
```
$ cat /etc/named/zones/db.dns.lab
dns.lab. IN SOA ns01 .dns.lab. root.dns.lab. (
 2711201407 ; serial
 3600 ; refresh (1 hour)
 600 ; retry (10 minutes)
 86400 ; expire (1 day)
 600 ; minimum (10 minutes)
 )
 IN NS ns01 .dns.lab.
; DNS Servers
ns01.dns.lab. IN A 10.0.0.23
; Web
web1.dns.lab. IN A 10.0.0.23
web2.dns.lab. IN A 10.0.0.23
```


**Пример файла зоны - сокращения**    
```
$ cat /etc/named/zones/db.dns.lab
$TTL 3600
; описание зоны dns.lab.
$ORIGIN dns.lab.
@ IN SOA ns01.dns.lab. root.dns.lab. (
 2711201407 ; serial
 3600 ; refresh (1 hour)
 600 ; retry (10 minutes)
 86400 ; expire (1 day)
 600 ; minimum (10 minutes)
 )
 IN NS ns01.dns.lab.
; DNS Servers
ns01 IN A 10.0.0.23
; Web
web1 IN A 10.0.0.23
web2 IN A 10.0.0.23
```


**Пример файла обратной зоны**
```
# Описание обратной зоны 0.0.10.in-addr.arpa.
$ cat /etc/named/zones/db.0.0.10
$TTL 604800
@ IN SOA ns01.dns.lab. root.dns.lab. (
 20210806 ; Serial
 604800 ; Refresh
 86400 ; Retry
 2419200 ; Expire
 604800 ) ; Negative Cache TTL
;
; name servers
@ IN NS ns01.dns.lab.
; PTR Records
23 IN PTR ns01.dns.lab. ;10.0.0.23
24 IN PTR testptr.dns.lab. ;10.0.0.24
```

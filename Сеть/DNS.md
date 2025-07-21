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

# Выполнение домашней работы по теме SystemD
## Задание №1 Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
1. Создаём файл с конфигурацией для сервиса в директории /etc/default - из неё сервис будет брать необходимые переменные.
```
touch /etc/default/watchlog
````
Вносим конфигурацию
```
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
2. Затем создаем /var/log/watchlog.log и пишем туда строки, в которых будет  ключевое слово "ALERT"
```
touch /var/log/watchlog.lo
```
Закинем несколько строк и добавим ключевое слово
```
Feb 11 11:38:29 lvm systemd[1]: Finished systemd-sysctl.service - Apply Kernel Variables.
Feb 11 11:38:29 lvm systemd[1]: Finished systemd-journal-flush.service - Flush Journal to Persistent Storage.
Feb 11 11:38:29 lvm multipathd[387]: multipathd v0.9.4: start up ALERT #Ключевое слово
Feb 11 11:38:29 lvm multipathd[387]: reconfigure: setting up paths and maps
Feb 11 11:38:29 lvm systemd[1]: Finished systemd-sysusers.service - Create System Users.
```
3. Создаём скрипт, который будет искать ключевое слово в нашем импровизированном журнале
```
touch /opt/watchlog.sh
```
```
#!/bin/bash

WORD="$1"
LOG="$2"
DATE=$(date)

# Проверка, переданы ли оба аргумента
if [[ -z "$WORD" || -z "$LOG" ]]; then
    echo "Usage: $0 <word> <logfile>"
    exit 1
fi

# Проверка существования файла
if [[ ! -f "$LOG" ]]; then
    echo "Error: File $LOG does not exist!"
    exit 2
fi

if grep -q "$WORD" "$LOG"; then
    logger "$DATE: I found the word '$WORD' in $LOG!"
fi
```
Добавим права на запуск скрипту
```
chmod +x /opt/watchlog.sh
```
4. Создаём юнит для сервиса
```
/etc/systemd/system/watchlog.service
```
```                    
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
````

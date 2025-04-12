  # Journald
journald - система регистрации событий в systemd    

__Особенности:__
- бинарный формат логов (защита от подделки, возможность конвертации в другие форматы)
- не требует специальной настройки
- структурированные данные (multi-field, multi-line)
- индексированные данные
- центральное хранилище логов


__Какие логи принимает journald?__     
- простые syslog логи логи ядра (kmsg)
- структурированные данные через Journal API
- логи и статусы systemd юнитов записи системы аудита (auditd)

__/etc/systemd/journald.conf__ - конфигурационный файл journald

__journald: параметры конфигурации__
- Storage (volatile, persistent, auto, none) - по-умолчанию auto (пишет логи в tmpfs), чтобы писать логи на диск - нужно поставить persistent
- Compress (yes, no) - сжимает данные перед записью
- Seal (yes, no) - накладывает криптографическую печать
- ForwardToSyslog, ForwardToKMsg, ForwardToConsole, ForwardToWall - опции перенаправления сообщений
- MaxLevelStore, MaxLevelSyslog, MaxLevelKMsg, MaxLevelConsole, MaxLevelWall - задаем уровни важности сообщений для разных логов
- SystemMaxUse - максимальный объем, который логи могут занимать на диске
- SystemKeepFree - объем свободного места на диске, после сохранения логов
- SystemMaxFileSize - объем файла лога, по достижении которого он должен быть удален с диска
- RuntimeMaxUse - максимальный объем, который логи могут занимать в файловой системе /run
- RuntimeKeepFree - объем свободного места на /run, после сохранения логов
- RuntimeMaxFileSize - объем файла лога, по достижении которого он должен быть удален из файловой системы/run

__Работа с журналом__    

__journalctl -b__ - покажет сообщения с ммоента загрузки системы 
__journalctl -p err -b__ - посмотреть только ошибки с момента последней загрузки        
__journalctl -e__  - покажет журнал с конца    
__journalctl -xe__ - покажет журнал с конца с расширенными событиями    
__journalctl -xe -o json-pretty__ - Вывод сообщений в структурированном json           
__journalctl -b | grep__  фильтруем по слову     
__journalctl --since "2024-10-15 00:00:00"__ - фильтруем по дате    
__journalctl --since "2024-10-15 00:00:00" --until "2024-10-15 08:00:00"__ - фильтруем от и до    
__journalctl - xeu nginx.service__ - посмотреть логи относящиеся к конкретному сервису   
__journactl -u mysqld.service -f__ - Вывод сообщений по заданному systemd юниту в формате чтения из файла    
__journalctl__ _UID=1001 -  Вывод сообщений процессов, запущенных от имени пользователя с заданным UID    
__journalctl -n 3 -p crit__ -   Вывод последних трех сообщений с уровнем важности crit    


__journalctl --flush__ - Перенести все логи из /run в /var   
__journalctl --vacuum-size=1G__ - Задать максимальный размер хранящихся логов    
__journalctl --vacuum-time=1years__ - Задать максимальное время хранения логов   
__journalctl --disk-usage__ - Показать занимаемый логами объем диска     

__journald: централизованное хранение__    

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9B%D0%BE%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5/Picture/journal_d.PNG">
</p>  

- systemd-journal-remote - демон для приема и сбора логов с удаленных хостов (работает в активном или пассивном режиме)
- systemd-journal-gatewayd - http-сервер для приема логов на центральный хост по протоколу http
- systemd-journal-upload - демон для загрузки логов с локальной машины в удаленное хранилище

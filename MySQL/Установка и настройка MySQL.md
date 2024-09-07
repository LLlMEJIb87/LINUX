# MySQL
_ _ _
**Базы** **данных** — это просто файлы на диске компьютера, куда можно записывать новые элементы. Но сами БД ничего не умеют и для них нужно писать свои методы для управления — например, для добавления нового элемента или поиска нужной записи. Чтобы облегчить работу программистам, придумали СУБД.    

**Система** **управления** **базами** **данных** **(СУБД)** — это набор инструментов, которые позволяют удобно управлять базами данных: удалять, добавлять, фильтровать и находить элементы, менять их структуру и создавать резервные копии.    


**MySQL** — свободная реляционная система управления базами данных (СУБД). Под словом «свободная» подразумевается ее бесплатность, под «реляционная» – работа с базами данных, основанных на двумерных таблицах. 

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/MySQL/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Arhitektura.PNG">
</p>


### Установка
1. Устанавливаем программу из репозитория
```
apt update
apt install mysql-server-8.0
```
```
ss -ntple проверяем, что сервер начал слушать порт tcp 3306
```
```
ps -afx видим,что запущен процесс /usr/sbin/mysqld
```
### Файловая структура
- Настройки системные: /etc/mysql/mysql.conf.d/, /etc/mysql/conf.d/
- Базы данных и бинлоги: /var/lib/mysql/ - нужно быть аккуратным с этим файлом.
- Настройки клиента: ~/.my.cnf
- https://dev.mysql.com/doc/refman/8.0/en/option-files.html
```
shmel@mysql-master:/etc/mysql$ tree
.
├── conf.d - файл с настройками клиента
│   ├── mysql.cnf
│   └── mysqldump.cnf
├── debian.cnf
├── debian-start
├── my.cnf -> /etc/alternatives/my.cnf  - файл исключительно для совместимости
├── my.cnf.fallback
├── mysql.cnf
└── mysql.conf.d
    ├── mysql.cnf
    └── mysqld.cnf - файл с настройками сервера
```
### Работа с MySQL
```
sudo mysql - зайти в программу
```
```
mysql> show databases; - посмотреть список баз данных
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```
```
use mysql - заходим в базу данных , в данном примере база данных mysql
```
```
mysql> show tables; - покажет содержимое базы данных mysql
```
```
mysql> select * from user \G  - смотрим, что находится в этой таблице user
```
#### Cоздание пользователя
```
mysql> create user test@'%' identified by 'password'; - создали пользователя с именем test (%-на любом хосте) и задали пароль
mysql> grant all on test.* to test@'%'; - задали все права этому пользователю
```
```
mysql> SELECT User, Host FROM mysql.user; - посмотреть всех пользователей 
mysql> select * from user where User='test'\G - фильтр и просмотр созданного пользователя
```
```
mysql> show grants for test@'%'; - посмотреть привилегии пользователя
```
```
mysql -utest -p - вошли под пользователем тест
```
## Репликация
**Задачи** **репликации**
- Высокая доступность (переключение в случае сбоя)
- Горизонтальное масштабирование
- Источник бекапа БД
- Разделение по типам нагрузки (OLTP, OLAP)    

### Настройка асинхронной реплицации
На Master ( он же source)   
1. Нужно настроить сервер MySQL в части возможности подключения другой машины, открыть сокет и задать другие настройки
- идем в /etc/mysql/mysql.conf.d и редактируем mysqld.cnf
- меняем bind-address = 0.0.0.0 (как пример, ip может быть разным в зависимости от ситуации)
- раскоментируем server-id  = 1
- добавим следующие команды:    
  gtid-node=ON   
  enforce-gtid-consistency    
  log-replica-updates    
- раскоментируем binlog_expire_logs_seconds   = 2592000
- Далее требуется сделать рестарт сервиса, MySQL не поддерживает reload
2. Создаем пользователя для репликации и указываем привелегии
```  
CREATE USER repl@'%' IDENTIFIED WITH 'caching_sha2_password' BY '!K3115007s';
GRANT REPLICATION SLAVE ON *.* TO repl@'%'; даём права на все базы данных * и все таблицы* пользователю repl
```
3. mysql> SHOW MASTER STATUS; Посмотреть статус


На Slave (он же replica)    
1. Нужно настроить сервер MySQL 
- идем в /etc/mysql/mysql.conf.d и редактируем mysqld.cnf
- раскоментируем и установим server-id  = 2
- раскоментируем binlog_expire_logs_seconds   = 2592000
- добавим следующие команды:    
  relay-log = relay-log-server
  read-only = ON
  gtid-mode=ON
  enforce-gtid-consistency
  log-replica-updates
2. Настраиваем mysql
```
mysql>  CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.201', SOURCE_USER='repl', SOURCE_PASSWORD='!K3115007s', SOURCE_AUTO_POSITION = 1, GET_SOURCE_PUBLIC_KEY = 1;
```
3. Запускаем реплику
```
mysql> START REPLICA;
```
4. Посмотреть статус
```
mysql> show replica status \G
```
## Бэкап базы данных
Типы бекапов
1.Логический (mysqldump, текстовый файл, SQL)
- Медленный
- Удобный
- Переносимый
2. Физический (бинарные файлы, binlog)
- Быстрый
- Внешние утилиты
- Percona XtraBackup
- https://www.percona.com/doc/percona-xtrabackup/2.4/index.html


### Создание бэкапа
1. Идем в директорию, где лежат базы данных /var/lib/mysql/ и выбираем нужную базу данных для бэкапа
2. Указываем таблицы для бекапа
```
root@mysql-master:/var/lib/mysql/world# mysqldump world city > /home/db/test_back.sql
```
3. Если база данных будет заливаться не на новый сервер (реплики), то требуется удалить/---закоментировать из бекапа строку   
 SET @@GLOBAL.GTID_PURGED=/*!80000 '+'*/ '5b8ad486-691e-11ef-89d5-08002702a0d0:1-15';

**Пример** скрипта для автоматического создания и сохранения бекапа потаблично:
```
#!/bin/bash
mysql -u root -p -e "STOP REPLICA"
mysqldump --set-gtid-purged=OFF world city > /home/db/world/world_city_bkp.sql
mysqldump --set-gtid-purged=OFF world country > /home/db/world/world_country_bkp.sql
mysqldump --set-gtid-purged=OFF world country_language > /home/db/world/world_country_language_bkp.sql
mysql -e "START REPLICA"
exit
```

### Восстановление из бэкапа
```
shmel@mysql-master:~$ mysql < world.sql "скармливаем" бекапную базу данных mysql
```

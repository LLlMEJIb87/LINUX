# IPTABLES
Iptables - это мощный инструмент управления сетью в Linux, который позволяет администраторам управлять входящими и исходящими пакетами данных. Это основной инструмент для настройки межсетевых экранов в системах Linux.    

Iptables работает путем проверки пакетов данных на соответствие определенным критериям и выполнения заданных действий, если пакеты соответствуют этим критериям. Эти критерии и действия определяются в таблицах, которые состоят из набора правил. 

В Iptables есть четыре основные таблицы:
1.  **Raw** - эта таблица используется для обхода системы отслеживания состояний.
2. **Mangle** - эта таблица используется для специальной обработки пакетов.
3.  **NAT** - эта таблица используется для настройки NAT (Network Address Translation)
4. **Filter** - это основная таблица, используемая для фильтрации пакетов.

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/chain.PNG">
</p>

Каждая таблица состоит из набора цепочек. Цепочки - это последовательности правил, которые применяются к пакетам. В Iptables есть три встроенные цепочки:
- **PREROUTING** -
- **INPUT** - эта цепочка применяется к пакетам, которые предназначены для самой системы.
- **FORWARD** - эта цепочка применяется к пакетам, которые проходят через систему.
- **OUTPUT** - эта цепочка применяется к пакетам, которые исходят из системы.
- **POSTROUTING** -
   
## Движение пакетов
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/filter.PNG">
</p>


### Команды
- iptables -nvL --line – просмотр списка правил ( по умолчанию смотрим таблицу FILTER, если нужна другая - iptables -nvL -t nat)
- iptables -F – сброс правил (политика остаётся)
- iptables -P – установка политики по умолчанию
- iptables -I – вставить правило в начало списка
- iptables -A – добавить правило в конец списка
- iptables -D – удалить правило   

- -p – проколол
- -i – интерфейс источника
- -o – интерфейс назначения
- -s – адрес источника
- --dport – порт назначения
- --sport – порт источника
- -m multiport --dports – несколько портов назначения
- -m state --state – статус соединения
- --icmp-type – тип ICMP-сообщения
- -j – действие    

__Действия с пакетами — target, jump (-j)__    

- ACCEPT — разрешить
- DROP — выкинуть
- REJECT — отклонить
   - iptables -A INPUT -s 10.26.95.20 -j REJECT --reject-with tcp-reset
   - iptables -A INPUT -p tcp -j REJECT --reject-with tcp-reset
   - iptables -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
   - iptables -A INPUT -j REJECT --reject-with icmp-proto-unreachable
- REDIRECT — перенаправить
   - iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
- DNAT/SNAT — destination/source NAT (network address translation)
   - iptables -t nat -A PREROUTING -p tcp --dport 9022 -j DNAT --to 192.168.56.6:22
- LOG — записать в лог
- RETURN — выйти из цепочки      

__Состояния пакетов__
- Сохранение состояния соединений — stateful firewall
- lsmod | grep conntrack
- iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
- NEW — пакет для создания нового соединения
- ESTABLISHED — пакет, принадлежащий к существующему соединению
- RELATED — пакет для создания нового соединения, но связанный с существующим (например, FTP)
- INVALID — пакет не соответствует ни одному соединению из таблицы
- UNTRACKED — пакет был помечен как неотслеживаемый в таблице raw     


## Базовая настройка фильтрации трафика таблицы FILTER
```
iptables -A INPUT -p tcp --dport=22 -j ACCEPT  разрешили подключение по SSH 
```
```
iptables -A INPUT -p icmp -j ACCEPT - разрешаем протокол ICMP
iptables -I INPUT 5 -p icmp --icmp-type echo-request -j DROP - если мы не хотим отвечать на пинги то устанавливаем это правило выше разрешающего icmp
```
```
iptables -I INPUT -i lo -j ACCEPT - полностью разрешаем локальный трафик ( -I -внести правило в начало списка)
```
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT разрешить входящий трафик, который был инициирован приложением 
```
**(После настройки правил, меняем политику входящего трафика с разрешающей на запрещающую)**
```
iptables -P INPUT DROP
```
**(Если требуется удалить правило)**:
```
iptables -D INPUT 8 -t filter
```
Выше мы ввелик команды, которые будут работать здесь и сейчас, но если перезагрузить машину, то првала не сохранятся. Для сохранения правил требуется сделать конфиг.

### Таблица NAT
Если требуется пробросить порт
```
iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080  (cоответсвенно порт 8080 должен быть разрешен в таблице INPUT иначе не заработает)
```
## Cохранение правил 
1. Устанавливаем требуемое ПО для сохранения конфига
```
apt install iptables-persistent netfilter-persistent  
```
2. После написаний правил сохраняем в конфиг
```
netfilter-persistent save
```
## IPSET
В случае если однородных правил становится слишком много (более 1000) то нужно использовать IPSET    

ipset — это сопутствующее приложение для межсетевого экрана Linux iptables. Он позволяет, помимо прочего, задавать правила для быстрой и простой блокировки набора IP-адресов.   

**Использование** **списков** **ipset**
- Создать (отдельные IP): ipset -N ddos iphash
- Создать (подсети): ipset create blacklist nethash
- Добавить подсеть: ipset -A ddos 109.95.48.0/21
- Посмотреть список: ipset -L ddos
- Проверить: ipset test ddos 185.174.102.1
- Сохранение: sudo ipset save blacklist -f ipset-blacklist.backup
- Восстановление: sudo ipset restore -! < ipset-blacklist.backup
- Очистка: sudo ipset flush blacklist
- Правило:   
 iptables -I PREROUTING -t raw -m set --match-set ddos src -j DROP

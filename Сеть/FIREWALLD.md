# FIREWALLD
firewalld — это демон (служба) для управления настройками брандмауэра (firewall) в Linux-системах, основанных на iptables, nftables, или ip6tables. Он предоставляет более высокоуровневый, динамический и зонально-ориентированный подход к настройке правил фаервола.     

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/firewalld.PNG">
</p>   

- Надстройка для удобного управления правилами firewall
- Содержит несколько утилит и сервис
- Имеет разделение правил и интерфейсов по зонам
- Для работы необходимо добавить интерфейс или диапазон source IP в зону (только в одну)
- Правила определяется по сервисам
- Может работать с различными бекендами (iptables, nft, ipset…)
- Конкуренты: ufw, Shorewall

### Конфигурация     
__Конфигурация firewalld__    
- Общий каталог (по умолчанию): /usr/lib/firewalld
- Зоны (по умолчанию): /usr/lib/firewalld/zones/*.xml
- Сервисы (по умолчанию): /usr/lib/firewalld/services/*.xml
- Настройки системы: /etc/firewalld/*
- Активная конфигурация: **runtime**
- Постоянная конфигурация: **permanent**

### Зоны    
__Зоны в firewalld__
- drop: самый низкий уровень доверия, DROP
- block: отклоняется с сообщением icmp-host-prohibited или icmp6-adm-prohibited, REJECT
- public: публичные сети, к которым нет доверия
- external: внешние сети, эта зона настроена для работы с NAT
- internal: для внутренней части шлюза
- dmz: используется для компьютеров в ДМЗ (изолированные компьютеры, у которых нет доступа к остальной части вашей сети)
- work: используется для рабочих компьютеров
- home: домашняя среда, доверие большей части других компьютеров
- trusted: все соединения разрешены

__Настройки зоны__  
- База
```
firewall-cmd --get-default-zone - выводит зону по умолчанию
firewall-cmd --get-active-zones - список активных зон
firewall-cmd --get-zones - список всех доступных зон
firewall-cmd --list-all - список всех служб текущей зоны
firewall-cmd --set-default-zone=work - изменить зону по умолчанию
firewall-cmd --get-services - список всех доступных служб в системе
firewall-cmd --zone=home --change-interface=eth0 - изменение зоны интерфейса
```   
- Сервисы (service)
```
- firewall-cmd --zone=public --add-service=http - разрешить временно
- firewall-cmd --zone=public --add-service=ftp
- firewall-cmd --permanent --zone=public --add-service=http - разрешить постоянно
- firewall-cmd --permanent --zone=public --add-service=ftp
- firewall-cmd --reload - подгрузка правил из permanent
- firewall-cmd --permanent --zone=public --list-services
```
- Порты (port)
- Блоки ICMP (icmp-block)
- Маскарадинг (masquerade)
- Проброс портов (forward-port)
```
firewall-cmd --permanent --new-zone=otus
 firewall-cmd --zone=otus --add-forward-port=port=2222:proto=tcp:toport=22:toaddr=1.1.1.1
 firewall-cmd --zone=otus --add-masquerade
```
- IPSET
```
- firewall-cmd --get-ipset-types
- firewall-cmd --permanent --new-ipset=test --type=hash:net
- firewall-cmd --permanent --info-ipset=test
- firewall-cmd --permanent --ipset=test --add-entry=192.168.0.1
- firewall-cmd --permanent --ipset=test --get-entries
- firewall-cmd --permanent --ipset=test --add-entries-from-file=ipl
- firewall-cmd --permanent --zone=drop --add-source=ipset:test
```
- Rich rules (rule)
```
rule
[source]
[destination]
service|port|protocol|icmp-block|icmp-type|
masquerade|forward-port|source-port
[log]
[audit]
[accept|reject|drop|mark]
```
rule
```
rule [family="ipv4|ipv6"] [priority="priority"]
```
source
```
source [not]
address="address[/mask]"|mac="mac-address"|ipset="ipset"
```
destination
```
destination [not] address="address[/mask]"
```
service
```
service name="service name"
```
port
```
port port="port value" protocol="tcp|udp"
```
protocol
```
protocol value="protocol value"
```

# NFTables

__Компоненты nftables__
- Family - семейства протоколов
- Ruleset - набор всех таблиц, цепочек и правил
- Table - таблицы, контейнеры для цепочек
- Chain - наборы правил, привязываются к Netfilter-хукам     


__Синтаксис nftables__
- Нижний регистр
- Необходимо экранировать для выполнения в bash (; и т.д.)
- Каждое правило на своей строчке
- Может быть несколько действий в правиле
- Опционалыные счетчики (counter), зависит от положения слова в правиле
- Формат вывода почти соответствует формату конфигурации
- nft -i - интерактивная консоль   
```
nft add table ip filter
nft 'add chain ip filter input { type filter hook input priority 0 ; policy accept; }'
```
https://wiki.nftables.org/wiki-nftables/index.php/Counters     


__Просмотр правил__
- iptables -V - проверка режима iptables (legacy, nf_tables)
- nft list ruleset - просмотр всей конфигурации. Тут нужно быть осторожным, если есть большие списки ipset это будет тормозить систему при выводе. Можно добавить ключ -t чтобы исключить вывод ipset
- nft list table ip filter - просмотр правил таблицы filter
- nft flush ruleset - очистка правил
- nft list tables - список всех таблиц
- nft list tables ip - список таблиц семейства ip (IPv4)
- nft list chains - список цепочек


Семейства цепочек и таблиц
- ip (default) — IPv4 протокол: по умолчанию, если не указано
- ip6 — IPv6 протокол
- inet — IPv4 и IPv6 протоколы. Dual Stack
- arp — ARP протокол
- bridge — мосты (свичи)
- netdev — привязка к одному интерфейсу, весь трафик (ingress хук)
 
## Цепочки
__Типы цепочек__
- filter — для фильтрации пакетов. Семейства таблиц arp, bridge, ip, ip6 и inet
- route — перенаправление пакетов. Семейства таблиц ip, ip6 inet
- nat — Networking Address Translation (NAT). Только первый пакет в соединении попадает в цепочку. Семейства таблиц ip, ip6 inet

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/NFT_hook.PNG">
</p>   

https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks    

__Хуки для цепочек (hooks)__
- ingress (только в netdev (4.2) и inet (5.10): видит пакеты сразу после драйвера сетевого интерфейса, даже до prerouting. Альтернатива для traffic control
- prerouting: все входящие пакеты до решения о роутинге
- input: входящие пакеты в сторону локальной системы
- forward: входящие пакеты не в локальную систему
- output: исходящие пакеты, сформированы в локальной системе
- postrouting: все пакеты после роутинга, как раз до выхода из локальной системы

https://wiki.nftables.org/wiki-nftables/index.php/Configuring_chains#Base_chain_types   


__Работа с цепочками и таблицами__
```
#Таблицы    
nft list tables [<family>]     
nft list table [<family>] <name> [-n] [-a]    
nft (add | delete | flush) table [<family>] <name>    
#Цепочки    
nft (add | create) chain [<family>] <table> <name> [ { type <type> hook <hook> [device <device>] priority <priority> \; [policy <policy> \;] } ]     
nft (delete | list | flush) chain [<family>] <table> <name>     
nft rename chain [<family>] <table> <name> <newname>     
```
https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes    

__Базовая конфигурация nftables__     
```
nft add table ip filter
nft 'add chain ip filter input { type filter hook input priority 0; policy accept; }'
nft add rule filter input tcp dport 22 counter accept
nft add rule filter input tcp dport {80, 443} ct state new accept
nft add rule filter input ct state related,established accept
nft add rule filter input ct state invalid drop
nft add rule filter input iifname "lo" accept
nft add rule filter input ip protocol icmp accept
nft 'chain filter input { policy drop; }
```

__Удаление и замена правил__
```
nft --handle list chain filter input
nft --handle --numeric list chain filter input
nft delete rule filter input handle 3
nft replace rule filter input handle 11 tcp dport 80 ct state new counter accept
nft add rule [<family>] <table> <chain> <matches> <statements>
nft insert rule [<family>] <table> <chain> [position <position>] <matches> <statements>
nft replace rule [<family>] <table> <chain> [handle <handle>] <matches> <statements>
nft delete rule [<family>] <table> <chain> [handle <handle>]
```
https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes     

__Сохранение и восстановление правил__
```
echo "flush ruleset" > /etc/nftables.conf
nft -s list ruleset >> /etc/nftables.conf
nft -f nft.rules — атомарнаā загрузка правил
systemctl enable nftables.service Ƃ вклĀùаем сервис
nft reset counters Ƃ сброс сùетùиков
(echo "flush ruleset"; nft --stateless list ruleset) | nft -f -
```

### Переход с iptables на nftables
```
iptables-translate -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
ip6tables-translate -A FORWARD -i eth0 -o eth3 -p udp -m multiport --dports 111,222 -j ACCEPT
ipset-translate restore < sets.ipset
iptables-save > save.txt
iptables-restore-translate -f save.txt > ruleset.nft
```

### IP SETS

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

# VLAN/LACP
## VLAN
Широковещательный домен – область сети, все устройства в которой получат один и тот же широковещательный кадр.    

**VLAN** — механизм для создания логической топологии сети независимо от ее физической топологии    

**ПРАКТИКА**    
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Сеть/picture/vlan_praktika.PNG">
</p>   

1. Настраиваем linux машину которая будет выступать в качестве коммутатора:
- создаем bridge интерфейс
```
ip link add name br0 type bridge vlan_filtering 1
```
- добвляем физические интерфейсы в bridge
```
ip link set eth0 master br0
ip link set eth1 master br0
ip link set eth2 master br0
```
- добавляем vlan и указываем в каком режиме должен работать интерфейс
```
bridge vlan add dev eth0 vid 10 pvid untagged
bridge vlan add dev eth1 vid 20 pvid untagged
bridge vlan add dev eth2 vid 10
bridge vlan add dev eth2 vid 20
```
- включаем интерфейс
```
ip link set bro up
```
- посмотреть настройки
```
bridge vlan show
```
2. Настраиваем адресацию на хостах ( они ничего не должны знать про vlan)
3. Настраиваем Server
- настраиваем sub интерфейсы
```
ip link add link eth0 name eth0.10 type vlan id 10
ip link add link eth0 name eth0.20 type vlan id 10
```
- настраиваем адресацию на sub интерфейсах
```
ip a add 192.168.10.10/24 dev eth0.10
ip a add 192.168.20.20/24 dev eth0.20
```
- активируем интерфейсы
```
ip link set eth0.10 up
ip link set eth0.20 up
```

## Ling agregation 
Агрегирование каналов (англ. link aggregation) — технологии объединения нескольких параллелыных каналов передачи данных в один логический.

**Bonding**

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Сеть/picture/bonding.PNG">
</p>   

**Практика**    

1. Статическое агрегирование
- Настройка Bonding (nmcli)

```
# Просмотр сетевых интерфейсов
$ nmcli con
# Задаем интерфейс bond0, задаем режим и ip-адрес
$ nmcli con add type bond con-name bond0 ifname bond0 mode active-backup ip4 10.16.10.7/24
$ nmcli con mod id bond0 bond.options mode=active-backup,fail_over_mac=1,miimon=100
# Добавляем сетевые интерфейсы в логический интерфейс
$ nmcli con add type bond-slave ifname eth0 master bond0
$ nmcli con add type bond-slave ifname eth1 master bond0
# Последовательно поднимаем интерфейсы
$ nmcli con up bond-slave-eth0
$ nmcli con up bond-slave-eth1
$ nmcli connection up bond0
```
- Настройка Bonding (через конфиги 1)
```
$ vim /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.0.0.1
NETMASK=255.255.255.0
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
NM_CONTROLLED=no
USERCTL=no
```
- Настройка Bonding (через конфиги 2)
```
$ vim /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no
USERCTL=no
$ vim /etc/sysconfig/network-scripts/ifcfg-eth2
DEVICE=eth2
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no
USERCTL=no
```

# Linux Bridge

В Linux есть встроенный свитч с поддержкой VLAN и STP, который может объеденить до 1012 интерфейсов. Bridge - програмный аналог физического свитча   

Управляется с помощью iproute2 (iplink), либо с помощью утилиты brctl(legacy) из пакета bridge-utils:
 ```
sudo ip link add name br0 type bridge
```

Возможно объеденить только L2 интерфейсы (eth, tap, wifi), L3 (tun,ppp) - в bridge добавить нельзя.Возможности:
- объединение интерфейсов в один домен коллизий
- филтрация MAC-адресов
- поддержка VLAN и STP


При объединение ethernet и wifi нужно, чтобы MAC-адресом bridge был установлен MAC-адрес wifi-интерфейса, иначе невозможно будет подключиться к wi-fi.   


**VLAN**     
Virtual LAN - логическая сегментация сети
- Тегирование трафика (IEEE 802.1Q)
- Пример создания VLAN-интерфейса
```
sudo ip link add link eth0 name eth0.10 type vlan id 10
```
- добавление в мост:
```
sudo ip link set eth0.10 master br0
```
- фильтрация по VLAN ID на уровне моста

**STP**     
```
sudo ip link set br0 type bridge stp_state 1
```
- проверка статуса:
```
bridge link show dev br0
```
- поддерживаемые протоколы: STP, RSTP



**ПРИМЕР НАСТРОЙКИ**
1. Создать мост
```
sudo ip link add name br-vlan type bridge
sudo ip link set br-vlan up
```
2. Добавить VLAN интерфейсы
```
sudo ip link add link eth0 name eth0.100 type vlan id 100
sudo ip link set eth0.100 master br-vlan
```
3. Включить STP
```
sudo ip link set br-vlan type bridge stp_state 1
```
4. Проверить
```
bridge vlan show && bridge stp show
```

# Выполнение домашнего задания на тему архитектура сетей.
1. VagrantFile
```
MACHINES = {
  :inetRouter => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "inetRouter",
        #:public => {:ip => "10.10.10.1", :adapter => 1},
        :net => [   
                    #ip, adpter, netmask, virtualbox__intnet
                    ["192.168.255.1", 2, "255.255.255.252",  "router-net"], 
                    ["192.168.50.10", 8, "255.255.255.0"],
                ]
  },

  :centralRouter => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralRouter",
        :net => [
                   ["192.168.255.2",  2, "255.255.255.252",  "router-net"],
                   ["192.168.0.1",    3, "255.255.255.240",  "dir-net"],
                   ["192.168.0.33",   4, "255.255.255.240",  "hw-net"],
                   ["192.168.0.65",   5, "255.255.255.192",  "mgt-net"],
                   ["192.168.255.9",  6, "255.255.255.252",  "office1-central"],
                   ["192.168.255.5",  7, "255.255.255.252",  "office2-central"],
                   ["192.168.50.11",  8, "255.255.255.0"],
                ]
  },

  :centralServer => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralServer",
        :net => [
                   ["192.168.0.2",    2, "255.255.255.240",  "dir-net"],
                   ["192.168.50.12",  8, "255.255.255.0"],
                ]
  },

  :office1Router => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Router",
        :net => [
                   ["192.168.255.10",  2,  "255.255.255.252",  "office1-central"],
                   ["192.168.2.1",     3,  "255.255.255.192",  "dev1-net"],
                   ["192.168.2.65",    4,  "255.255.255.192",  "test1-net"],
                   ["192.168.2.129",   5,  "255.255.255.192",  "managers-net"],
                   ["192.168.2.193",   6,  "255.255.255.192",  "office1-net"],
                   ["192.168.50.20",   8,  "255.255.255.0"],
                ]
  },

  :office1Server => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Server",
        :net => [
                   ["192.168.2.130",  2,  "255.255.255.192",  "managers-net"],
                   ["192.168.50.21",  8,  "255.255.255.0"],
                ]
  },

  :office2Router => {
       :box_name => "ubuntu/jammy64",
       :vm_name => "office2Router",
       :net => [
                   ["192.168.255.6",  2,  "255.255.255.252",  "office2-central"],
                   ["192.168.1.1",    3,  "255.255.255.128",  "dev2-net"],
                   ["192.168.1.129",  4,  "255.255.255.192",  "test2-net"],
                   ["192.168.1.193",  5,  "255.255.255.192",  "office2-net"],
                   ["192.168.50.30",  8,  "255.255.255.0"],
               ]
  },

  :office2Server => {
       :box_name => "ubuntu/jammy64",
       :vm_name => "office2Server",
       :net => [
                  ["192.168.1.2",    2,  "255.255.255.128",  "dev2-net"],
                  ["192.168.50.31",  8,  "255.255.255.0"],
               ]
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
       end

      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL
    end
  end
end
```
2. Cхема сети

 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/Shema_seti.PNG">
</p>

3. Список серверов     

<div align="center">

| Server         | IP and Bitmask                          |
|----------------|------------------------------------------|
| inetRouter     | Default-NAT address VirtualBox          |
|                | 192.168.255.1/30                        |
| centralRouter  | 192.168.255.2/30                        |
|                | 192.168.0.1/28                          |
|                | 192.168.0.33/28                         |
|                | 192.168.0.65/26                         |
|                | 192.168.255.9/30                        |
|                | 192.168.255.5/30                        |
| centralServer  | 192.168.0.2/28                          |
| office1Router  | 192.168.255.10/30                       |
|                | 192.168.2.1/26                          |
|                | 192.168.2.65/26                         |
|                | 192.168.2.129/26                        |
|                | 192.168.2.193/26                        |
| office1Server  | 192.168.2.130/26                        |
| office2Router  | 192.168.255.6/30                        |
|                | 192.168.1.1/26                          |
|                | 192.168.1.129/26                        |
|                | 192.168.1.193/26                        |
| office2Server  | 192.168.1.2/26                          |

</div>

4. Список сетей     
Central Network

| Name            | Network         | Netmask            | N  | Hostmin       | Hostmax       | Broadcast      |
|-----------------|------------------|---------------------|----|----------------|----------------|----------------|
| Directors       | 192.168.0.0/28    | 255.255.255.240     | 14 | 192.168.0.1   | 192.168.0.14  | 192.168.0.15   |
| Office hardware | 192.168.0.32/28   | 255.255.255.240     | 14 | 192.168.0.33  | 192.168.0.46  | 192.168.0.47   |
| Wifi (mgt)      | 192.168.0.64/26   | 255.255.255.192     | 62 | 192.168.0.65  | 192.168.0.126 | 192.168.0.127  |

---

 Office 1 Network

| Name | Network         | Netmask            | N  | Hostmin       | Hostmax       | Broadcast      |
|------|------------------|---------------------|----|----------------|----------------|----------------|
| Dev  | 192.168.2.0/26    | 255.255.255.192     | 62 | 192.168.2.1   | 192.168.2.62  | 192.168.2.63   |
| Test | 192.168.2.64/26   | 255.255.255.192     | 62 | 192.168.2.65  | 192.168.2.126 | 192.168.2.127  |
| Managers | 192.168.2.128/26 | 255.255.255.192  | 62 | 192.168.2.129 | 192.168.2.190 | 192.168.2.191  |
| Office hardware | 192.168.2.192/26 | 255.255.255.192  | 62 | 192.168.2.193 | 192.168.2.254 | 192.168.2.255  |

---

 Office 2 Network

| Name   | Network           | Netmask            | N   | Hostmin        | Hostmax        | Broadcast       |
|--------|--------------------|---------------------|-----|-----------------|-----------------|-----------------|
| Dev    | 192.168.1.0/25      | 255.255.255.128     | 126 | 192.168.1.1    | 192.168.1.126  | 192.168.1.127   |
| Test   | 192.168.1.128/26    | 255.255.255.192     | 62  | 192.168.1.129  | 192.168.1.190  | 192.168.1.191   |
| Office | 192.168.1.192/26    | 255.255.255.192     | 62  | 192.168.1.193  | 192.168.1.254  | 192.168.1.255   |

---

 InetRouter – CentralRouter Network

| Name          | Network            | Netmask            | N  | Hostmin        | Hostmax        | Broadcast       |
|---------------|---------------------|---------------------|----|-----------------|-----------------|-----------------|
| Inet – central | 192.168.255.0/30    | 255.255.255.252     | 2  | 192.168.255.1  | 192.168.255.2  | 192.168.255.3   |


### Настройка NAT на сервере inetRouter
1. Отключаем firewall ufw
```
systemctl stop ufw
systemctl disable ufw
```
2. Создаём файл /etc/iptables_rules.ipv4:
```
nano /etc/iptables_rules.ipv4

*filter
:INPUT ACCEPT [90:8713]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [54:7429]
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
COMMIT

*nat
:PREROUTING ACCEPT [1:44]
:INPUT ACCEPT [1:44]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT
```
3.  Создаём файл, в который добавим скрипт автоматического восстановления правил при перезапуске системы:
```
nano /etc/network/if-pre-up.d/iptables

#!/bin/sh
/sbin/iptables-restore < /etc/iptables_rules.ipv4
```
4. Добавляем права на выполнение файла /etc/network/if-pre-up.d/iptables
```
chmod +x /etc/network/if-pre-up.d/iptables
```
5. Перезагружаем сервер
```
shutdown -r now
```
### Настройка маршрутизации транзитных пакетов
1. Проверяем статус форвардинга пакетов на сервере
```
vagrant@inetRouter:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```
0 - не активирован
2. Активируем
```
echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
```
Данную операцию проделываем на всех маршрутизаторах в сети

### Отключение маршрута по умолчанию на интерфейсе enp0s3
При разворачивании нашего стенда Vagrant создает в каждом сервере свой интерфейс, через который у сервера появляется доступ в интернет. Отключить данный порт нельзя, так как через него Vagrant подключается к серверам. Обычно маршрут по умолчанию прописан как раз на этот интерфейс, данный маршрут нужно отключить:
```
nano 50-cloud-init.yaml

network:
    ethernets:
        enp0s3:
            dhcp4: true
            dhcp4-overrides:
                use-routes: false
            dhcp6: false
            match:
                macaddress: 02:a7:b1:fd:e7:e2
            set-name: enp0s3
    version: 2
```
```
netplan try
```
Данную операцию проделываем на всех маршрутизаторах/cерверах кроме inetRouter

### Настройка статических маршрутов
1. Так как у нас нет маршгрута по умолчанию, то нет сетевой связанности с хостами в сети
```
root@office1Server:/home/vagrant# ping 192.168.1.2
ping: connect: Network is unreachable
```
Нам нужно настроить маршруты по умолчанию, делаем на  office1Server default маршрут в сторону office1Router
```
nano /etc/netplan/50-vagrant.yaml

---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 192.168.2.130/26
      routes:
      - to: 0.0.0.0/0
        via: 192.168.2.129
    enp0s19:
      addresses:
      - 192.168.50.21/24
```
2. Делаем на office1Router
```
ip route add default via 192.168.255.10 dev enp0s8
```
3. Делаем на centralRouter
```
ip route add default via 192.168.255.1 dev enp0s8
ip route add 192.168.2.0/24 via 192.168.255.10 dev enp0s17
ip route add 192.168.1.0/24 via 192.168.255.6 dev enp0s18
```

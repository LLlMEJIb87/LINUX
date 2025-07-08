# Туннели
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Сеть/picture/otus_vpn.PNG">
</p>

**L2 Туннели**    
Ethernet поверх других протоколов. Технологии:
- VXLAN
- L2TP
- Ethernet over GRE   

Пример Создание VXLAN-туннеля:
```
sudo ip link add vxlan0 type vxlan id 42 dstport 4789 remote 10.0.0.2 
```

**L3 Туннели**   

IP-in-IP инкапсуляция
Протоколы:
- GRE (Generic Routing Encapsulation)
- IPIP (IP over IP)
- WireGuard (L3+ с шифрованием)


Пример GRE:
```
sudo ip tunnel add gre0 mode gre remote 203.0.113.5 local 198.51.100.1
sudo ip link set gre0 up
```

**L4 Туннели**    
Туннелирование через TCP/UDP     
Методы:
- SSH Port Forwarding ( -L , -R )
- WireGuard (использует UDP)
- socat для проброса портов   

Пример SSH:
```
ssh -L 8080:localhost:80 user@gateway.example.com
```

**L7 Туннели**    
Трафик поверх прикладных протоколов     
Подходы:
- HTTP-туннели (httptunnel, ncat)
- DNS-туннели (iodine)
- SSLH (мультиплексирование через HTTPS)    


Пример SSLH:
```
sslh -p 0.0.0.0:443 --ssh 127.0.0.1:22 --ssl 127.0.0.1:443
```

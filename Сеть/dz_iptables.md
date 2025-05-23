# Выполненее домашнего задания на тему IPTABLES
Тополгия сети берется из прошлого задания   https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/dz_arhitektura_seti.md    
## Задание №1. Реализовать knocking port
centralRouter может попасть на ssh inetrRouter через knock.     
Делаем на inetRouter.
1. Настраиваем IPTABLES
```
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -P INPUT DROP
netfilter-persistent save
```
2. Устанавливаем и конфигурим knockd
```
sudo apt install knockd

#правим /etc/default/knockd
START_KNOCKD=1
KNOCKD_OPTS="-i enp0s8"

#конфигурим /etc/knockd.conf
[options]
        UseSyslog

[openSSH]
        sequence      = 1234,2345,3456
        seq_timeout   = 15
        command       = /usr/sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        cmd_timeout   = 60
        stop_command  = /usr/sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags      = syn
```
Перезапускаем
```
sudo systemctl daemon-reexec
sudo systemctl restart knockd
sudo systemctl enable knockd
```
3. Идем на centalRouter и проверяем
```
#Открываем порт
root@centralRouter:/home/vagrant# knock 192.168.255.1 1234 2345 3456
#Подключаемся по ssh
root@centralRouter:/home/vagrant# ssh vagrant@192.168.255.1
```
## Задание 2. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост

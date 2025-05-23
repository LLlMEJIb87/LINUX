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
## Задание 2. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост. Запустить nginx на centralServer и пробросить 80й порт на inetRouter2 8080.
1. Создаем inetRouter2
```
MACHINES = {
  :inetRouter2 => {
    :box_name => "ubuntu/jammy64",
    :vm_name => "inetRouter",
    :net => [
      # ip, adapter, netmask
      ["192.168.50.50", 8, "255.255.255.0"]
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
        box.vm.network("private_network",
          ip: ipconf[0],
          adapter: ipconf[1],
          netmask: ipconf[2]
        )
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
2. Устанавливаем Nginx на centralServer
```
sudo apt install nginx
```

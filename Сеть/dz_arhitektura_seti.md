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



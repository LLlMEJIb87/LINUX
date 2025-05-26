# Выполнение домашнего задания на тему OSPF
1. Поднимаем стенд
```
MACHINES = {
  :router1 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "router1",
        :net => [
                   {ip: '10.0.10.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
                   {ip: '10.0.12.1', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
                   {ip: '192.168.10.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net1"},
                   {ip: '192.168.50.10', adapter: 5},
                ]
  },

  :router2 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "router2",
        :net => [
                   {ip: '10.0.10.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
                   {ip: '10.0.11.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
                   {ip: '192.168.20.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
                   {ip: '192.168.50.11', adapter: 5},
                ]
  },

  :router3 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "router3",
        :net => [
                   {ip: '10.0.11.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
                   {ip: '10.0.12.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
                   {ip: '192.168.30.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net3"},
                   {ip: '192.168.50.12', adapter: 5},
                ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
    
    config.vm.define boxname do |box|
   
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]

      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", **ipconf
      end

    end
  end
end
```

  <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%A1%D0%B5%D1%82%D1%8C/picture/ospf_topology.PNG">
</p>    

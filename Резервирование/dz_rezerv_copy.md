# Выполнение домашней работы на тему рещервное копирование
1. Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. 
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.ssh.insert_key = false

  # Common VirtualBox provider settings
  config.vm.provider "virtualbox" do |vb|
    vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga']
    vb.cpus = 2
    vb.memory = 2048
    vb.gui = false
  end

  # Backup Server VM
  config.vm.define "backup_server" do |va|
    va.vm.provider "virtualbox" do |vb|
      vb.name = "backup_server"
    end
    va.vm.hostname = "backup-server"
    va.vm.network "public_network", ip: "192.168.1.210", bridge: "Intel(R) Dual Band Wireless-AC 7260"
    va.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "add_ssh_user.yml"
    end
  end

  # Client VM
  config.vm.define "client" do |client|
    client.vm.provider "virtualbox" do |vb|
      vb.name = "client"
    end
    client.vm.hostname = "client"
    client.vm.network "public_network", ip: "192.168.1.211", bridge: "Intel(R) Dual Band Wireless-AC 7260"
    client.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "add_ssh_user.yml"
    end
  end
end
```
```
- hosts: all
  become: true
  tasks:
    - name: create shmel user
      ansible.builtin.user:
        name: shmel
        password: $6$nQPdUpPsRnBKuxxM$MTiQBKKWJ/EUevz4JX0nSEEO1B6AKtzuhQSRxDkEfJ52wDH1jxEKD.MI4a3HNFsEZfU4MlVWnpES7k9IhbFBC.
        shell: /bin/bash
        group: sudo

    - name: add ssh key for user shmel
      ansible.posix.authorized_key:
        user: shmel
        state: present
        key: "{{ lookup('file', 'shmel.pub') }}"
        exclusive: true
```

2. Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup.

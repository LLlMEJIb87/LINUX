# Выполнение домашнего задания на тему Pluggable Authentication Modules (PAM) 
1. Создаем Vagrantfile и запускаем VM
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64" 
  config.ssh.insert_key = false 
  config.vm.provider "virtualbox" do |vb|
    vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga'] 
    vb.cpus = 2	
    vb.memory = 2048 
    vb.gui = false 
    vb.name = "pam" 
  end
  
#VM for dz OTUS
  config.vm.define "pam" do |va| 
  va.vm.hostname = "pam-host" 
  va.vm.network "public_network", ip: "192.168.1.220", bridge: "Intel(R) Dual Band Wireless-AC 7260" 
  va.vm.provision "ansible_local" do |ansible| 
    ansible.playbook = "password_yes.yaml" 
     end
  end
end
```
playbook
```
---
- name: Configure SSH for password authentication
  hosts: all
  become: true  
  tasks:
    - name: Ensure PasswordAuthentication is set to yes
      lineinfile:
        path: /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
        regexp: '^PasswordAuthentication\s+no'
        line: 'PasswordAuthentication yes'
        state: present  
      notify:
        - Restart SSH

  handlers:
    - name: Restart SSH
      service:
        name: sshd
        state: restarted
```

2. Подключаемся к VM
3. Создаем пользователей и пароли к ним
```
useradd otusadm && passwd otusadm
useradd otus && passwd otus
```
4. Cоздаем группу
```
groupadd -f admin
```
5. Добавляем пользователей vagrant,root и otusadm в группу admin
```
usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```

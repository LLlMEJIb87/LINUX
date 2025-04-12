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
6. Проверяем подключени по SSH
```
C:\Users\RocketJump\Documents\Vagrant_job>ssh otus@192.168.1.220
otus@192.168.1.220's password:
$ whoami
otusadm
$ exit
Connection to 192.168.1.220 closed.

C:\Users\RocketJump\Documents\Vagrant_job>ssh otusadm@192.168.1.220
otusadm@192.168.1.220's password:
$ whoami
otusadm
```
7. Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
```
root@pam-host:/# cat /etc/group | grep admin
admin:x:118:otusadm,root,vagrant
```
8. Выберем метод PAM-аутентификации, так как у нас используется только ограничение по времени, то было бы логично использовать метод pam_time, однако, данный метод не работает с локальными группами пользователей, и, получается, что использование данного метода добавит нам большое количество однообразных строк с разными пользователями. В текущей ситуации лучше написать небольшой скрипт контроля и использовать модуль pam_exec: 
```
touch  /usr/local/bin/login.sh
``` 
```
#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
```
```
chmod +x /usr/local/bin/login.sh
```

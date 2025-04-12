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
В скрипте подписаны все условия. Скрипт работает по принципу: 
Если сегодня суббота или воскресенье, то нужно проверить, входит ли пользователь в группу admin, если не входит — то подключение запрещено. При любых других вариантах подключение разрешено.     

9. Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт
```
nano /etc/pam.d/sshd 

# PAM configuration for the Secure Shell service

# Вставляем скрипт ДО всех auth-правил
auth       required     pam_exec.so debug /usr/local/bin/login.sh

# Standard Un*x authentication.
@include common-auth

# Disallow non-root logins when /etc/nologin exists.
account    required     pam_nologin.so

# Uncomment and edit /etc/security/access.conf if you need to set complex
# access limits that are hard to express in sshd_config.
# account  required     pam_access.so

# Standard Un*x authorization.
@include common-account

# SELinux needs to be the first session rule.
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so close

# Set the loginuid process attribute.
session    required     pam_loginuid.so

# Create a new session keyring.
session    optional     pam_keyinit.so force revoke

# Standard Un*x session setup and teardown.
@include common-session

# Print MOTD
session    optional     pam_motd.so  motd=/run/motd.dynamic
session    optional     pam_motd.so noupdate

# Mail notification
session    optional     pam_mail.so standard noenv

# Limits
session    required     pam_limits.so

# Read environment vars
session    required     pam_env.so
session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale

# SELinux open
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so open

# Password updates
@include common-password
```
10. Проверяем
```
otus
Apr 12 07:01:21 pam-host sshd[1238]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Apr 12 07:01:21 pam-host sshd[1235]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Apr 12 07:01:23 pam-host sshd[1235]: Failed password for otus from 192.168.1.88 port 51750 ssh2
otusadm
Apr 12 07:01:27 pam-host sshd[1235]: Connection reset by authenticating user otus 192.168.1.88 port 51750 [preauth]
Apr 12 07:01:34 pam-host sshd[1244]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Apr 12 07:01:34 pam-host sshd[1242]: Accepted password for otusadm from 192.168.1.88 port 51787 ssh2
Apr 12 07:01:34 pam-host sshd[1242]: pam_unix(sshd:session): session opened for user otusadm(uid=1002) by (uid=0)
```


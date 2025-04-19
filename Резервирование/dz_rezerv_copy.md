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
2.1 Устанавливаем ПО borgbackup (на клиенте и сервере)
```
apt install borgbackup
```
2.2  На сервере backup создаем пользователя и каталог /var/backup и назначаем на него права пользователя borg	
```
sudo useradd --system --no-create-home --shell /usr/sbin/nologin borg
mkdir /var/backup
mount /dev/sdc1 /var/backup
chown borg:borg /var/backup/
```
2.3  На сервер backup создаем каталог ~/.ssh/authorized_keys в каталоге /home/borg
```
mkdir /home/borg
chown borg:borg /home/borg
chmod 700 /home/borg
mkdir /home/borg/.ssh
touch /home/borg/.ssh/authorized_keys
chown -R borg:borg /home/borg/.ssh
chmod 700 /home/borg/.ssh
chmod 600 /home/borg/.ssh/authorized_keys
```
2.4 На client создаем ключи
```
ssh-keygen
```
2.5 На backup-server
В authorized_keys добавляем ключ client с ограничением:
```
command="borg serve --restrict-to-path /srv/borg/repos",no-port-forwarding,no-pty,no-agent-forwarding,no-X11-forwarding ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDfacR6Rt9CklF+rjey6xwnirWUwlGU9UnUxvPjcx/D43GNIHwZlhmlIrxrRbDiUWqEGSJP6/ZeFR+hEbrBXDblNRfRo5NPwemi4kPwxfgTtsXr550Qcn8szTtI44C5jXMCAfzaDPHLO+H8HAFsMnWDq81TP6ughKwez4tKF/Q/UkbUbMwtwo+4N4zAqjRz6GPwTMY7L+aOheFpPqK04fcjaEwdoq9lbh3dkPdQtAqb8ikR2jfzsMuu51U2ytkYh/h6p8rWj+KkMb2Okd8GkDjcMcskyZjQX1FZ22uSqaTZIDHtm9QZ83khgl7CQ3p/yI+f1ewi40k8wmRbJr6idOoPFQRRPNIHguCVOwUmgi0uc8NwDWojXLEpoT/vVjmt9qxdm/p8Cy0vkAOIiUI0LYpj9+pY6t2k4O7Jc4Pz0jqOAKZgzmjknEi24bTHCs6r/IQZgI/V60QktwXWljlr33MKPHLzWV03FQ5k1UeJDS1yP3RANphADXPJX2MrCYMixbs= root@client
```
2.6 Все дальнейшие действия будут проходить на client сервере.     
Инициализируем репозиторий borg на backup сервере с client сервера
```
root@client:~/.ssh# BORG_RSH="ssh -i borg_ssh" borg init --encryption=repokey borg@192.168.1.210:/var/backup/
```
2.7 Запускаем для проверки создания бэкапа
```
BORG_RSH="ssh -i borg_ssh" borg create --stats --list borg@192.168.1.210:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc
```
2.8 Смотрим, что получилось
```
root@client:~/.ssh# BORG_RSH="ssh -i borg_ssh" borg list borg@192.168.1.210:/var/backup/
Enter passphrase for key ssh://borg@192.168.1.210/var/backup: 
etc-2025-04-19_10:26:43              Sat, 2025-04-19 10:26:50 [e01144c9b68eeb6e21b3a86de250c66330341a4008df3c889fc3e8b671821e70]
```
2.9 Смотрим список файлов
```
BORG_RSH="ssh -i borg_ssh" borg list borg@192.168.1.210:/var/backup/::etc-2025-04-19_10:26:43
```
2.10 Достаем файл из бекапа
```
root@client:~/.ssh# BORG_RSH="ssh -i borg_ssh" borg extract borg@192.168.1.210:/var/backup/::etc-2025-04-19_10:26:43 etc/hostname
```
3. Автоматизируем создание бэкапов с помощью systemd
3.1 Создаем сервис и таймер в каталоге /etc/systemd/system/
```
nano /etc/systemd/system/borg-backup.service
                                                                      
[Unit]
Description=Borg Backup

[Service]
Type=oneshot

# Парольная фраза
Environment="BORG_PASSPHRASE=k3115007s"
# Репозиторий
Environment=REPO=borg@192.168.1.210:/var/backup/
# Что бэкапим
Environment=BACKUP_TARGET=/etc

# Создание бэкапа
ExecStart=/bin/borg create 
    --stats                
    ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}

# Проверка бэкапа
ExecStart=/bin/borg check ${REPO}

# Очистка старых бэкапов
ExecStart=/bin/borg prune 
    --keep-daily  90      
    --keep-monthly 12     
    --keep-yearly  1       
    ${REPO}
```
```

nano /etc/systemd/system/borg-backup.timer
[Unit]
Description=Borg Backup

[Timer]
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```
3.2 Включаем и запускаем службу таймера
```
systemctl enable borg-backup.timer 
systemctl start borg-backup.timer
```

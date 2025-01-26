# Домашняя работа объединяющая темы Vagrant и Ansible.
Цель - автоматизировать процесс установки/запуска виртуальной машины с установленным/настроенным дополнительным ПО при помощи средств автоматизации таких как Vagrant и Ansible

1. Vagrant file:
```
Vagrant.configure("2") do |config| #Вместо Vagrant.configure подставляем переменную config
  config.vm.box = "ubuntu/jammy64" #Указываем дестрибутив для установки
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |vb| #Вместо config.vm.provider "virtualbox" подставляем переменную vb
    vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga'] #Указываем какой графический контроллер будем использовать в virtualbox
    vb.cpus = 1	#Указываем сколько ядер выделить для виртуальной машины
    vb.memory = 2048 #Указываем сколько памяти выделить для виртуальной машины
    vb.gui = false #Отключаем вэб интерфейс
    vb.name = "vm_nginx_1.210" #Указываем имя виртуальной машины в менеджере vitrualbox
  end
  
#Nginx vm vagrant
  config.vm.define "vm_nginx" do |va| #Указываем имя виртуальной машины в vagrant
  va.vm.hostname = "vm-nginx" #Указываем имя хоста
# va.vm.network "private_network", ip: "172.16.0.5"
  va.vm.network "public_network", ip: "192.168.1.210", bridge: "Intel(R) Dual Band Wireless-AC 7260" #Добавляем сетевой интерфейс с статическим ip, устанавливаем режим моста
  va.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "add_user_and_upgrade_repo.yaml" #Cценрай добавления пользователя и создания ключа
#   ansible.inventory_path = "inventory"
     end
     va.vm.provision "ansible_local" do |ansible|
       ansible.playbook = "nginx.yml"                   #Cценарий установки nginx
    end
  end
end
```
2. Сценарий который добавляет пользователя,ключ и обновляет ядро до последней версии
```
---
- hosts: all   #Указываем на каких хостах запустить playbook
  become: true #Задачи будут выполняться с правами администратора
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

  - name: Update & upgrade repo
    ansible.builtin.apt:
      update_cache: true
      upgrade: full
  ```

3. Cценарий который утсанавливает nginx
```
- name: Setup Nginx server
  hosts: all
  become: yes
  vars_files:
    - become_password
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: add nginx official gpg key
      ansible.builtin.get_url:
        url: https://nginx.org/keys/nginx_signing.key
        dest: /usr/share/keyrings/nginx_key.asc

    - name: add nginx repository
      ansible.builtin.apt_repository:
        repo: "deb [signed-by=/usr/share/keyrings/nginx_key.asc] http://nginx.org/packages/ubuntu {{ ansible_distribution_release }} nginx"
        state: present

    - name: set nginx package priority
      ansible.builtin.copy:
        dest: /etc/apt/preferences.d/99nginx
        content: |
          Package: *
          Pin: origin nginx.org
          Pin: release o=nginx
          Pin-Priority: 900

    - name: update apt cache
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 3600
      tags:
        - update apt

    - name: install nginx
      ansible.builtin.apt:
        pkg:
          - nginx
        state: latest
      notify:
        - restart nginx
      tags:
        - nginx-package

    - name: Create nginx config file from template
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
      tags:
        - nginx-configuration

    - name: enable nginx service
      ansible.builtin.systemd:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: restart nginx
      ansible.builtin.systemd:
        name: nginx
        state: restarted

    - name: reload nginx
      ansible.builtin.systemd:
        name: nginx
        state: reloaded
```


__Как итог, при использование команды vagrant UP через несколько минут получаю виртуальную машину с настроенным пользователем,доступом по ключу, и установленным/настроенным nginx__

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%90%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_vagant%2Bansible.PNG">
</p>


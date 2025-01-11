# Домашняя работа по теме Vagrant.
_ _ _
__1) Запустить ВМ с помощью Vagrant.__
Создаю Vagrantfile:
```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64" #Указываем box для установки
  config.ssh.insert_key = false #Отключаем автоматическое добавление ключей в гостевую систему.
  config.vm.provider "virtualbox" do |vb|
    vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga'] #Указываем какой графический контроллер будем использовать в virtualbox
    vb.cpus = 1	#Указываем сколько ядер выделить для виртуальной машины
    vb.memory = 2048 #Указываем сколько памяти выделить для виртуальной машины
    vb.gui = false #Отключаем вэб интерфейс
    vb.name = "dz_upgrade" #Указываем имя виртуальной машины в менеджере vitrualbox
  end
  
#VM for dz OTUS
  config.vm.define "vm_dz_upgrade" do |va| #Указываем имя виртуальной машины в vagrant
  va.vm.hostname = "vm-dz-upgrade" #Указываем имя хоста
  va.vm.network "public_network", ip: "192.168.1.210", bridge: "Intel(R) Dual Band Wireless-AC 7260" #Добавляем сетевой интерфейс с статическим ip, устанавливаем режим моста
  va.vm.provision "ansible_local" do |ansible| #Подготавливаем гостевую систему с помощью сценария Ansible
    ansible.playbook = "add_user_and_upgrade_repo.yaml" #Указываем какой сценарий использовать
     end
  end
end
```
__2. Обновить ядро OS__
Для обнавления ядра OS был создан сценарий Ansible
```
---
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

  - name: Update & upgrade repo
    ansible.builtin.apt:
      update_cache: true
      upgrade: full
```

**Итого:** При использование команды vagrant up получаю ВМ с обновленным репозиторем и ядром, с добавленным пользователем с возможностью входа по ключу. Остается только перезагрузить машину

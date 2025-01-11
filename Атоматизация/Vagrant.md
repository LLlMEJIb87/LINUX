# VAGRANT
_ _ _
Vagrant (с англ. — «бродяга») — свободное и открытое программное обеспечение для создания и конфигурирования виртуальных машин.    

https://developer.hashicorp.com/vagrant/docs/vagrantfile
## Установка и настройка на WINDOWS
1. Скачиваем и устанавливаем VirtualBox для Windows - https://www.virtualbox.org/wiki/Downloads
2. Скачиваем и устанавливаем Vagrant - https://www.vagrantup.com/downloads.html
3. Проверяем установился ли Vagrant (используем командную строку CMD)
```
vagrant -v #Команда
Vagrant 2.4.3 # Результат
```

## Начало работы с Vagrant
1. Создадим директорию для нашего  проекта и зайдем в неё:
```
mkdir vagrant_job
cd vagrant_job
```
2. Начинаем новый проект:
```
vagrant init
```
Команда vagrant init создает Vagrantfile который обозначает root директорию нашего проекта и описывает конфигурацию виртуальной машины.    
3. Открываем Vagrantfile в текстовом редакторе, удаляем дефолтные настройки и добавляем следующий конфиг:
```
Vagrant.configure("2") do |config| #Вместо Vagrant.configure подставляем переменную config
  config.vm.box = "ubuntu/jammy64" #Указываем дестрибутив для установки
  config.vm.provider "virtualbox" do |vb| #Вместо config.vm.provider "virtualbox" подставляем переменную vb
    vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga'] #Указываем какой графический контроллер будем использовать в virtualbox
    vb.cpus = 1	#Указываем сколько ядер выделить для виртуальной машины
    vb.memory = 2048 #Указываем сколько памяти выделить для виртуальной машины
    vb.gui = false #Отключаем вэб интерфейс ОС
	vb.name = "vm_vagrant_ansible" #Указываем имя виртуальной машины в менеджере vitrualbox
  end
  
#Test vm vagrant-ansible
  config.vm.define "vb_vagrant_ansible" do |va| #Указываем имя виртуальной машины в vagrant
  va.vm.hostname = "vagrant-ansible" #Указываем имя хоста
  va.vm.network "public_network", ip: "192.168.1.210", bridge: "Intel(R) Dual Band Wireless-AC 7260" #Добавляем сетевой интерфейс с статическим ip, устанавливаем режим моста
  end 
end
```
4. Запускаем Vagrant
```
vagrant up
```
### Список команд Vagrant:
```
vagrant up #Запуск VM в соответсвие с vagrant file

vagrant ssh #Подключиться к VM по SSH

vagrant halt #Выключает машину

vagrant destroy #Удаление VM

vagrant reload #Перезагружает VM

vagrant suspend #Сохраняет состояние VM и останавливает её

vagrant resume #Возобнавляет работу VM

vagrant global-status #Посмотреть информацию обо всех ВМ

vagrant snapshot #Управление снапшотами

#Cделать снапшот
vagrant snapshot test
vagrant snaphot list
vagran snapshot restore test
```
__Логин и пароль по умолчанию vagrant vagrant__

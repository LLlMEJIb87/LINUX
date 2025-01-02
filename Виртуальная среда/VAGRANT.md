# VAGRANT
_ _ _
Vagrant (с англ. — «бродяга») — свободное и открытое программное обеспечение для создания и конфигурирования виртуальной среды разработки.

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
Vagrant.configure(2) do |config|

  # Vagrant box - образ ОС из которой будет создана виртуальная машина.
  # Список образов на https://app.vagrantup.com/boxes/search
  # Здесь используется Ubuntu Server 20.04 
  config.vm.box = "ubuntu/focal64"
  

  # Отключает автоматическое обновление образов.
  config.vm.box_check_update = false
  

  # Создает приватную сеть  с доступом только из хоста и присваивает виртуальной машине IP.
  config.vm.network "private_network", ip: "192.168.88.2"
  

  # Директории которые будут синхронизироватьсямежду локальным компьютером и виртуалкой.
  config.vm.synced_folder "C:/Users/RocketJump/Documents/Vagrant_job", "/var/www/vagrant_job"
  

  # Настройки виртуальной машины
  config.vm.provider "virtualbox" do |vb|
	 # Показывать или нет интерфейс VirtualBox при загрузке.
	 vb.gui = false
	 # Выделяемая память для виртуалки.
	 vb.memory = "2048"
	 # Название в интерфейсе VirtualBox.
	 vb.name = "vagrant_job"
  end  

end
```
4. Запускаем Vagrant
```
vagrant up
```
5. Список команд Vagrant:
```
# Запуск виртуальной машины
vagrant up
# Выключает машину
vagrant halt
# Перезапуск машины
vagrant reload
# Запускает заново скрипт конфигурации виртуальной машины
# Нужно выполнять при изменении файла конфигурации
vagrant provision
# Удаление файлов виртуалки
vagrant destroy
```

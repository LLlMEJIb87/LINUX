# Домашняя работа по теме Ansible
1. Подготовил и проверил удаленный хост
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%90%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_ansible.PNG">
</p>    

Хост был создан при поомощи ПО Vagrant:
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
    ansible.playbook = "add_user_and_upgrade_repo.yaml"
#   ansible.inventory_path = "inventory"
     end
  end
end
```

2. Создал роль nginx
```
pyenv exec ansible-galaxy init roles/nginx
```
3. Зашифровал пароль sudo с помощью ansible-vault
```
pyenv exec ansible-vault enrypt become_password
```
4. Написал playbook nginx.yml
```
- name: Setup Nginx server
  hosts: nginx
  become: yes
  vars_files:
    - become_password
  roles:
    - nginx
```
5. Написал задачи roles/nginx/tasks/main.yml 
```
- name: add nginx official gpg key #Cкачиваем публичный GPG-ключ для проверки подленности пакета
  ansible.builtin.get_url: #Используем модуль для скачивания файла          
    url: https://nginx.org/keys/nginx_signing.key #Откуда скачиваем
    dest: /usr/share/keyrings/nginx_key.asc #Куда положить

- name: add nginx repository #Добавляем репозиторий Nginx
  ansible.builtin.apt_repository: #Модуль для управления репозиториями
    repo: "deb [signed-by=/usr/share/keyrings/nginx_key.asc] http://nginx.org/packages/ubuntu {{ ansible_distribution_release }} nginx" #Пакеты из репозитория nginx будут подписаны ключем скаченным ранее)
    state: present #репозиторий должен быть добавлен(или уже присутсвовать)

- name: set nginx package priority #Указываем приорет для пакетов Nginx (устанавливать пакеты с репозитория Nginx, а не Ubuntu)
  ansible.builtin.copy: #Модуль для копирования, перемещения или создание файлов
    dest: /etc/apt/preferences.d/99nginx #Путь куда мы сохраним созданные ниже нами настройки
    content: | #делаем записи в файл (| -означает построчно)
      Package: *
      Pin: origin nginx.org
      Pin: release o=nginx
      Pin-Priority: 900

- name: update apt cache #Обновляем кэш с приложениями репозитория Ubuntu
  ansible.builtin.apt: Модуль для работы с пакетами
    update_cache: true
    cache_valid_time: 3600 Максимальное время, в течение которого кэш считается действителен
  tags:
    - update apt #Тэг для возможности исполнения конкретно этой задачи

- name: install nginx #Устанавливаем Nginx
  ansible.builtin.apt:
    pkg:               #Указываем какой пакет должен быть установлен
      - nginx
    state: latest      #Пакет должен быть установлен + проверка самой новой версии
  notify:              #Вызываем обработчик
    - restart nginx
  tags:
    - nginx-package

- name: Create nginx config file from template #Указываем, что конфиг nginx должен быть создан из шаблона
  ansible.builtin.template:
    src: nginx.conf.j2 #Откуда берем
    dest: /etc/nginx/nginx.conf #Куда положим на целевом хосте
  notify:
    - reload nginx
  tags:
    - nginx-configuration

- name: enable nginx service  #Делаем nginx в автозагрузку
  ansible.builtin.systemd:    #Модуль для управления сервисами, которые используют systemd для старта
    name: nginx
    state: started            #Cтартует сервис если он не запущен
    enabled: true             #автоматический запуск при старте системы

```
6. Написал handlers roles/nginx/handlers/main.yml
```
- name: restart nginx
  ansible.builtin.systemd:
    name: nginx
    state: restarted

- name: reload nginx
  ansible.builtin.systemd:
    name: nginx
    state: reloaded
```
7. Закинул файл с конфигом в roles/nginx/templates/nginx.conf.j2
```
# {{ ansible_managed }}
events {
    worker_connections 1024;
}

http {
    server {
        listen       {{ nginx_listen_port }} default_server;
        server_name  default_server;
        root         /usr/share/nginx/html;

        location / {
        }
    }
}
```
8. Запустил playbook
```
pyenv exec ansible-playbook ./playbook/nginx.yml
```

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%90%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_ansible2.PNG">
</p>

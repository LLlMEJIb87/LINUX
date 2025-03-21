# ANSIBLE
_ _ _ 
**Ansible** - это инструмент автоматизации процессов администрирования. Он использует простые, понятные человеку скрипты, называемые плейбуками, для автоматизации задач, которые мы пишем в формате YAML.  
Ansible читает построчно наш Playbook (который мы хотим применить) и из него  гинерирует Python скрипт, который по ssh (SCP) копируется на удаленную машину и исполняется на ней.     

__Термины и определения:__
1. Управляющий узел (Control Node): система, на которой установлен Ansible.
2. Целевой узел (Managed Node): система, которой управляют с помощью
Ansible.
3. Плейбук (Playbook): текстовый файл в формате YAML, содержащий список
задач, которые нужно выполнить на целевых узлах.
4. Задача (Task): единичное действие, которое Ansible должен выполнить на
целевых узлах.
5. Модуль (Module): программы, выполняющие определенные задачи на
целевых узлах.
6. Инвентарь (Inventory): файл, содержащий список целевых узлов, с
которыми Ansible должен взаимодействовать.
7. Факт (Fact): данные о целевых узлах, собранные Ansible для использования
в плейбуках.
8. Шаблон (Template): файлы, которые содержат специальные заполнители,
которые Ansible может заменить значениями переменных.
9. Handlers: специальные задачи в плейбуке, которые выполняются только
при наличии триггера.


## pyenv
Инструмент, который позволяет управлять разными версиями Python на машине. Требуется установить на машине где будет запущен ansible master server    

1. Устанавливаем пакет:
```
curl https://pyenv.run | bash
```
2. Настраиваем среду оболочки для Pyenv    
- Сначала добавьте команды, ~/.bashrcвыполнив в терминале следующее:
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc
```
-  Затем, если у вас есть ~/.profile, ~/.bash_profile или ~/.bash_login, добавьте туда команды. Если у вас нет ни одного из них, создайте ~/.profile и добавьте туда команды:
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo 'eval "$(pyenv init - bash)"' >> ~/.profile
```
3. Рестар оболочки
```
exec "$SHELL"
```
4. Установка зависимостей:
```
sudo apt update; sudo apt install build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
5. Устанавливаем менеджер пакетов для Python
```
sudo apt install python3-pip
```

https://github.com/pyenv/pyenv?tab=readme-ov-file#linuxunix


### работа с pyenv
- Посмотреть установленные версии python
```
pyenv versions
```
- Установить последнюю 3 версию
```
pyenv install 3
```
- Установить нужную локальную версию python
```
pyenv local 3.11.9
```
- Посмотреть пакеты на Pythone
```
pyenv exec pip3 list
```
## Установка Ansible
В зависимости от того, какая локальная версия Python выбрана на машине, после ввода команды установится максимальная версия Ansible . Таким образом мы можем одновременно иметь разные версии ansible и переключаться между ними, после установки разных версий Ansible, путем смены локальной версии Pythone.
1. Создаем папку ansible_project, в ней поддиректории с версиями python (которые установили через pyenv)
```
mkdir ansible_project
cd ./asnible_project
mkdir v_3.9.18
mkdir v_3.12.3
mkdir v_3.13.1
```
2. Заходим в папку с нужной версией и меняем локально версию Pythone
```
pyenv local 3.9.18
```
3. Устанавливаем ansible максимальную версию для выбранной нами версии Python
```
pyenv exec pip3 install ansible
```
4. Далее просто меняем версию Pythone через pyenv по необхимости и можем пользовать разными версиями Ansible

Посмотреть версию ansible
```
pyenv exec ansible --version
```

### Конфиг Ansible
1. Конфигурационный файл при установке через pyenv не создается, создаем его в папке ansible_project самостоятельно:
```
touch ansible.cfg
```
2. Вносим несколько команд  для начала работы
```
[defaults]
inventory = ./inventory/hosts #Указываем где находится наш файл inventory
ask_vault_pass = true #Указываем, что playbook нужно запускать сразу с этой опцией

 #Использовать пользователя root по умолчанию
```
### Inventory
Специальный файл где хранится информация об узлах, на которых мы будем выполнять наши сценарии (файл может быть не один, для разных задач можно делать разные файлы)     
1. Создаем файл в директории ansible_project
```
mkdir inventoy
cd inventory
touch hosts
```
2. Пример файла inventroy c названием проекта wazuh
```
---
wazuh:                   #Общая группа
  children:              #Указывает, что в группе есть подгруппы
    wazuh_server:        #Подгруппа
      hosts:
        192.168.1.100:
    wazuh_exporter:      #Подгруппа
      hosts:
        192.168.1.101:

dzupgrade:               #Другая группа
  hosts:
    192.168.1.210:
  vars:                  #Общие переменные для всех групп
    ansible_user: shmel  
    ansible_ssh_private_key_file: /home/shmel/.ssh/id_rsa
```
Посмотреть файл inventory в виде графа:
```
pyenv exec ansible-inventory -i inventory --graph
@all:
  |--@ungrouped:
  |--@wazuh:
  |  |--@wazuh_server:
  |  |  |--192.168.1.100
  |  |--@wazuh_exporter:
  |  |  |--192.168.1.101
```
### Модули
Модули - мини программа выполняющие определенную функцию. Основа для выполнения действий в Ansible. Ниже предствален модуль для првоерки роботоспособности файла конфига и inventory, если в ответ приходит pong, то все в порядке
```
pyenv exec ansible -m ping -i inventory wazuh(группа хостов) (-m указываем что хотим использовать модуль, ping - модуль пинг, котоорый запустит ping до хостов из файла inventory
```
### Playbooks
Playbook - список сценариев для достижения целевого состояния системы с использованием модулей Ansible.    

Пример сценария добавления пользователя, ключа для пользователя и обновления системы
```
- hosts: all #Ссылается на хосты из inventory
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
```                                                                                               
- name: Setup Nginx server #Цель playbook
  hosts: dzupgrade #Указываем целевой хост
  become: yes # Задачи будут выполняться с правами администратора
  vars_files: #C помощью ansible-vaul зашифровали пароль sudo для выполнения задач из под админа
    - become_password #Указываем файл с зашифрованным паролем sudo удаленного хоста
  tasks:
    - name: add nginx official gpg key #Cкачиваем публичный GPG-ключ для проверки подленности пакета
      ansible.builtin.get_url:  #Используем модуль для скачивания файла
        url: https://nginx.org/keys/nginx_signing.key #Откуда скачиваем
        dest: /usr/share/keyrings/nginx_key.asc #Куда положить

    - name: add nginx repository #Добавляем репозиторий Nginx
      ansible.builtin.apt_repository: #Модуль для управления репозитория
        repo: "deb [signed-by=/usr/share/keyrings/nginx_key.asc] http://nginx.org/packages/ubuntu {{ ansible_distribution_release }} nginx" #Пакеты из репозитория nginx будут подписаны ключем скаченным ранее)
        state: present #репозиторий должен быть добавлен(или уже присутсвовать)

    - name: set nginx package priority #Указываем приорет для пакетов Nginx (устанавливать пакеты с репозитория Nginx, а не Ubuntu)
      ansible.builtin.copy: #Модуль для копирования, перемещения или создание файлов
        dest: /etc/apt/preferences.d/99nginx #Путь куда мы сохраним созданные ниже нами настройки
        content: | #делаем записи в файл (| -о значает построчно)
          Package: *
          Pin: origin nginx.org
          Pin: release o=nginx
          Pin-Priority: 900

    - name: update apt cache #Обновляем кэш с приложениями репозитория Ubuntu
      ansible.builtin.apt: #Модуль для работы с пакетами
        update_cache: true
        cache_valid_time: 300 #Максимальное время, в течение которого кэш считается действителен

    - name: install nginx #Устанавливаем Nginx
      ansible.builtin.apt:
        pkg:
          - nginx
        state: present

    - name: enable nginx service #Делаем nginx в автозагрузку
      ansible.builtin.systemd: #Модуль для управления сервисами, которые используют systemd для старта
        name: nginx
        state: started #Cтартует сервис если он не запущен
        enabled: true #автоматический запуск при старте

```
Сценарий выполняется по умолчанию сверху вниз проходя и дожидаясь выполнения на каждом хосте, это не единственная стратегия выполнения, можно выбрать свободную стратегию    
__Проверка написания playook__
```
ansible-playbook site.yml -i inventory --syntax-check
ansible-playbook site.yml -i inventory check #Сухой прогон, задачи не будут применены к хосту, но программа покажет будет ли он выполнен без ошибок
```
Запустить playbook
```
pyenv exec ansible-playbook ./playbook/nginx.yml
```
__Нюансы__
- для использования sudo на удаленном хоcте нам нужно знать пароль, чтобы не держать его в открытом виде на мастер сервере ншифруем пароль при помощи ansible-vault encrypt и кладем его в файл, далее в playbook указываем ссылку на файл
- зашифрованный файл должен быть в дериктории с playbook 

#### ROLES 
Роли в Ansible — это способ организации и структурирования вашего кода, чтобы его было легче поддерживать и повторно использовать. Cейчас мы переделаем playbook, который был выше используя роли:  

Каждая роль в Ansible состоит из определенного набора директорий и файлов, например:

- tasks/ — содержит список задач, которые выполняет роль.
- defaults/ — здесь хранятся значения переменных по умолчанию.
- vars/ — для переменных, которые можно переопределять.
- files/ — для файлов, которые могут быть переданы на хосты.
- templates/ — для шаблонов конфигурационных файлов, которые будут подставляться на хосты.
- handlers/ — для задач, которые выполняются, если что-то изменилось (например, перезапуск службы).

__Роль Common__    
Роль common в Ansible — это базовая роль, которая используется для выполнения стандартных или общих задач на серверах. Она обычно применяется ко всем серверам в инфраструктуре, независимо от их назначения (веб-сервера, базы данных, кэш-сервера и т.д.)    
Для автоматического создания директорий роли common используем следующую команду:
```
ansible-galaxy init roles/common
```
В директории tasks лежит main файл, наполняем его
```
- name: Update and upgrade apt package
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600
    upgrade: safe #Обновляются только те пакеты, которые могут быть обновлены без удаления или установки новых пакетов.
```
__Роль nginx_install__    
Для автоматического создания директорий роли nginx_install используем следующую команду:
```
ansible-galaxy init roles/ nginx_install
```
В директории tasks лежит main файл, наполняем его
```
- name: add nginx official gpg key #Cкачиваем публичный GPG-ключ для проверки подленности пакета
  ansible.builtin.get_url:  #Используем модуль для скачивания файла
    url: https://nginx.org/keys/nginx_signing.key #Откуда скачиваем
    dest: /usr/share/keyrings/nginx_key.asc #Куда положить

- name: add nginx repository #Добавляем репозиторий Nginx
  ansible.builtin.apt_repository: #Модуль для управления репозитория
    repo: "deb [signed-by=/usr/share/keyrings/nginx_key.asc] http://nginx.org/packages/ubuntu {{ ansible_distribution_release }} nginx" #Пакеты из репозитория nginx будут подписаны ключем скаченным ранее)
    state: present #репозиторий должен быть добавлен(или уже присутсвовать)
  notify: Update apt cache #Выполняется в случае изменения, Указывает, что если задача изменила состояние, то нужно запустить update cache

- name: set nginx package priority #Указываем приорет для пакетов Nginx (устанавливать пакеты с репозитория Nginx, а не Ubuntu)
  ansible.builtin.copy: #Модуль для копирования, перемещения или создание файлов
    dest: /etc/apt/preferences.d/99nginx #Путь куда мы сохраним созданные ниже нами настройки
    content: | #делаем записи в файл (| -означает построчно)
          Package: *
          Pin: origin nginx.org
          Pin: release o=nginx
          Pin-Priority: 900

- name: install nginx #Устанавливаем Nginx
  ansible.builtin.apt:
    pkg:
      - nginx
    state: present
  notify:
    - Start Nginx #В случае изменений стартануть Nginx
```
Как видим выше мы использовали тригеры notify, далее создадим их в этой роле в директории handlers:
```
- name: update apt cache
  ansible.builtin.apt: 
    date_cache: true
    cache_valid_time: 3600

- name: Start Nginx
  ansible.builtin.systemd: 
    service: nginx
    state: started 
```
Notify делают отсылку на handlers     

__Роль nginx_deploy__     
Для автоматического создания директорий роли nginx_deploy используем следующую команду:
```
ansible-galaxy init roles/ nginx_deploy
```
В директории tasks лежит main файл, наполняем его
```
- name: Remove default config #Удаляем старую конфигурацию
  ansible.builtin.file:
    path: /etc/nginx/conf.d/default.conf
    state: absent

- name: Create new Nginx default config #Cоздаем новую конфигурацию из шаблона j2
  ansible.builtin.template:
    src: default.j2
    dest: /etc/nginx/conf.d/{{ inventory_hostname }}.conf
    backup: true
  notify:
    - Restart nginx

- name: Create folder for ssl certificates #Cоздается новая директория для ssl сертификатов
  ansible.builtin.file:
    path: /etc/nginx/ssl
    state: directory

- name: Copy ssl certificates #Копируем файлы и закидываем файлы по указанному пути
  ansible.builtin.copy:
    src: "{{ item }}"
    dest: "/etc/nginx/ssl/{{ item }}"
  loop:                       #По сути цикл for
    - fullchain.pem
    - privkey.pem

- name: Change nginx.conf #Меняем конфиг по умолчанию
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    backup: true

- name: Create new Nginx web package #Cоздается новая приветсвенная страничка
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    backup: true
```
Как видим выше мы использовали тригеры notify, далее создадим их в этой роле в директории handlers:
```
- name: Restart Nginx
  ansible.builtin.systemd_service:
    services: nginx
    state: restarted
```
Notify делают отсылку на handlers      

В директорию Defaults, записываем переменные по улмолчанию:
```
nginx:
  http:
    port: 80
  https:
    port: 443
  certs:
    pub: fullchain.pem
    priv: privkey.pem
```

И так далее заполняем нужные директории    

В итоге получаем playbook такого вида:
```
- name: Setup Nginx server
  hosts: dzupgrade 
  become: yes 
  roles:
    - role: common
    - role: nginx_install
    - role: nginx_deploy
```

### Переменные
YAML поддерживает словари и списки. Их можно поддерживать для переменных. Пример использования переменной в playbook:
```
- hosts reddit_app
  vars:
    active: yes
    app_path: "{{ base_dir }}/reddit_app #Переменная
```
#### Машические переменные
**Hostvars** - особая переменная, позволяющая при управление одним хостом получит доступ к значениям переменных другого хоста:
```
{{ hostvars['host1']['admin_user'] }}
{{ hostvars['host1']['admin_facts']['os_family] }}
```
### Facts
В Ansible, помимо явно определенных разработчиком переменных, существует read-only системные переменные, которые также известны как факты. За это отвечает модуль setup. Посмотреть  все факты, которые может собрать setup модуль из системы можно командой:
```
pyenv exec ansible -m setup -i inventory wazuh_server
```
Где (как пример) Переменная из фактов
```
"ansible_service_mgr": "systemd" #Переменная из фактов
```

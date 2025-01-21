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
remote_user = root #Использовать пользователя root по умолчанию
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
Модули - Небольшие библиотеки для выполнения и отслеживания задач. Основа для выполнения действий в Ansible. Ниже предствален модуль для првоерки роботоспособности файла конфига и inventory, если в ответ приходит pong, то все в порядке
```
pyenv exec ansible -m ping -i inventory wazuh(группа хостов) (-m указываем что хотим использовать модуль, ping - модуль пинг, котоорый запустит ping до хостов из файла inventory
```
### Playbooks
Playbook - список сценариев для достижения целевого состояния системы с использованием модулей Ansible.    

Пример сценария добавления пользователя, ключа для пользователя и обновления системы
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

  - name: Update & upgrade repo
    ansible.builtin.apt:
      update_cache: true
      upgrade: full
```
Сценарий выполняется по умолчанию сверху вниз проходя и дожидаясь выполнения на каждом хосте, это не единственная стратегия выполнения, можно выбрать свободную стратегию    
__Проверка написания playook__
```
ansible-playbook site.yml -i inventory --syntax-check
```
### Переменные
YAML поддерживает словари и списки. Их можно поддерживать для переменных. Пример использования переменной в playbook:
```
- hosts reddit_app
  vars:
    active: yes
    app_path: "{{ base_dir }}/reddit_app
```
### Facts
В Ansible, помимо явно определенных разработчиком переменных, существует read-only системные переменные, которые также известны как факты. За это отвечает модуль setuo. Посмотреть  все факты, которые может собрать setup модуль из системы можно командой:
```
ansible -m setup
```

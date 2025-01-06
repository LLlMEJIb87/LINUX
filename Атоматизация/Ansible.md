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
позволяет легко переключаться между несколькими версиями Python. Требуется установить на машине где будет запущен ansible master server
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
sudo apt update; sudo apt install build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
5. Устанавливаем менеджер пакетов для Python
```
sudo apt install python3-pip
```

https://github.com/pyenv/pyenv?tab=readme-ov-file#linuxunix


### работа с pyenv
- Посмотреть установленные версии python
```
sudo su - shmel #Получаем права суперпользователя
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
```
pyenv exec pip3 install ansible
```

Посмотреть версию ansible
```
pyenv exec ansible --version
```

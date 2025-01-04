# ANSIBLE
_ _ _ 
**Ansible** - это инструмент автоматизации процессов администрирования. Он использует простые, понятные человеку скрипты, называемые плейбуками, для автоматизации задач

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
В зависимости от того, какая локальная версия Python выбрана на машине, после ввода команды установится максимальная версия Ansible . Установка происходит в папку  в которой находимся при использование команд, для разных версий ansible нужно создать соответсвующие папки. Таким образом мы можем одновременно иметь разные версии ansible и переключаться между ними.
```
pyenv exec pip3 install ansible
```

Посмотреть версию ansible
```
pyenv exec ansible --version
```

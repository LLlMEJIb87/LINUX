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

https://github.com/pyenv/pyenv?tab=readme-ov-file#linuxunix

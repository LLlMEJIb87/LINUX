# Домашняя работа по теме Ansible
1. Подготовил и проверил удаленный хост
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%90%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/dz_ansible.PNG">
</p>
2. Создаль роль nginx
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

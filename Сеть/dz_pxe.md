# Выполнение домашней работы на тему PXE
```
agrant.configure("2") do |config|
  config.vm.define "pxeserver" do |server|
    server.vm.box = 'Ubuntu/jammy64' # Обновили образ
    server.vm.host_name = 'pxeserver'
    server.vm.network "forwarded_port", guest: 80, host: 8080
    server.vm.network :private_network,
      ip: "10.0.0.20",
      virtualbox__intnet: 'pxenet',
      adapter: 2
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    server.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "ansible_dz_network.yml" # Обновили playbook
      ansible.limit = "pxeserver"
      ansible.install = true
      ansible.install_mode = "default"
      ansible.compatibility_mode = "2.0"
    end
  end

  config.vm.define "pxeclient" do |pxeclient|
    pxeclient.vm.box = 'Ubuntu/jammy64'
    pxeclient.vm.host_name = 'pxeclient'
    pxeclient.vm.network :private_network,
      ip: "10.0.0.21",
      virtualbox__intnet: 'pxenet',
      adapter: 2,
      auto_config: false
    pxeclient.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize [
        'modifyvm', :id,
        '--boot1', 'net',
        '--boot2', 'none',
        '--boot3', 'none',
        '--boot4', 'none'
      ]
    end
  end
end
```

```
---
- name: Настройка PXE-сервера для автоматической установки Ubuntu 24.04
  hosts: pxeserver
  become: yes
  tasks:
    - name: Отключение файрвола ufw
      systemd:
        name: ufw
        state: stopped
        enabled: no

    - name: Обновление кэша пакетов
      apt:
        update_cache: yes

    - name: Установка dnsmasq
      apt:
        name: dnsmasq
        state: present

    - name: Создание конфигурации dnsmasq для PXE
      copy:
        dest: /etc/dnsmasq.d/pxe.conf
        content: |
          interface=enp0s8
          bind-interfaces
          dhcp-range=enp0s8,10.0.0.100,10.0.0.120,12h
          dhcp-boot=pxelinux.0
          enable-tftp
          tftp-root=/srv/tftp/amd64
          dhcp-no-override
        mode: '0644'
      notify: Перезапуск dnsmasq

    - name: Создание директории для TFTP
      file:
        path: /srv/tftp
        state: directory
        mode: '0755'

    - name: Скачивание и распаковка netboot-архива Ubuntu 24.04
      unarchive:
        src: http://releases.ubuntu.com/24.04/ubuntu-24.04.2-netboot-amd64.tar.gz
        dest: /srv/tftp
        remote_src: yes
        creates: /srv/tftp/amd64/pxelinux.0

    - name: Установка Apache2
      apt:
        name: apache2
        state: present

    - name: Создание директории для ISO-образов
      file:
        path: /srv/images
        state: directory
        mode: '0755'

    - name: Скачивание ISO-образа Ubuntu 24.04
      get_url:
        url: http://releases.Ubuntu.com/24.04/ubuntu-24.04.2-live-server-amd64.iso
        dest: /srv/images/ubuntu-24.04.2-live-server-amd64.iso
        mode: '0644'

    - name: Создание конфигурации Apache для PXE
      copy:
        dest: /etc/apache2/sites-available/ks-server.conf
        content: |
          <VirtualHost 10.0.0.20:80>
              DocumentRoot /
              <Directory /srv/images>
                  Options Indexes MultiViews
                  AllowOverride All
                  Require all granted
              </Directory>
              <Directory /srv/ks>
                  Options Indexes MultiViews
                  AllowOverride All
                  Require all granted
              </Directory>
          </VirtualHost>
        mode: '0644'
      notify: Перезапуск Apache

    - name: Активация сайта ks-server
      command: a2ensite ks-server.conf
      args:
        creates: /etc/apache2/sites-enabled/ks-server.conf
      notify: Перезапуск Apache

    - name: Настройка pxelinux для загрузки ISO
      copy:
        dest: /srv/tftp/amd64/pxelinux.cfg/default
        content: |
          DEFAULT install
          LABEL install
          KERNEL linux
          INITRD initrd
          APPEND root=/dev/ram0 ramdisk_size=3000000 ip=dhcp iso-url=http://10.0.0.20/srv/images/ubuntu-24.04.2-live-server-amd64.iso autoinstall ds=nocloud-net;s=http://10.0.0.20/srv/ks/
        mode: '0644'

    - name: Создание директории для файлов автоматической установки
      file:
        path: /srv/ks
        state: directory
        mode: '0755'

    - name: Создание файла user-data для автоматической установки
      copy:
        dest: /srv/ks/user-data
        content: |
          #cloud-config
          autoinstall:
            apt:
              disable_components: []
              geoip: true
              preserve_sources_list: false
              primary:
              - arches:
                - amd64
                - i386
                uri: http://us.archive.Ubuntu.com/ubuntu
              - arches:
                - default
                uri: http://ports.Ubuntu.com/ubuntu-ports
            drivers:
              install: false
            identity:
              hostname: linux
              password: $6$sJgo6Hg5zXBwkkI8$btrEoWAb5FxKhajagWR49XM4EAOfO/Dr5bMrLOkGe3KkMYdsh7T3MU5mYwY2TIMJpVKckAwnZFs2ltUJ1abOZ.
              realname: otus
              username: otus
            kernel:
              package: linux-generic
            keyboard:
              layout: us
              toggle: null
              variant: ''
            locale: en_US.UTF-8
            network:
              ethernets:
                enp0s3:
                  dhcp4: true
                enp0s8:
                  dhcp4: true
              version: 2
            ssh:
              allow-pw: true
              authorized-keys: []
              install-server: true
            updates: security
            version: 1
        mode: '0644'

    - name: Создание пустого файла meta-data
      file:
        path: /srv/ks/meta-data
        state: touch
        mode: '0644'

  handlers:
    - name: Перезапуск dnsmasq
      systemd:
        name: dnsmasq
        state: restarted

    - name: Перезапуск Apache
      systemd:
        name: apache2
        state: restarted
```

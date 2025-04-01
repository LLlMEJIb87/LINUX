# ПРОМИФИУС
_ _ _
  <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3/Picture/prometheus.PNG">
</p>        

  
**Prometheus** — это система мониторинга и оповещения с открытым исходным кодом, которая используется для сбора и хранения метрик из различных источников. Она создана для мониторинга высокодинамичных сред, таких как облачные приложения или микросервисы. С помощью Prometheus можно отслеживать и анализировать производительность и состояние ваших приложений и инфраструктуры.    

Prometheus может быть использован для мониторинга использования ресурсов на серверах, отслеживания времени ответа веб-сервисов, сбора метрик с баз данных и измерения производительности контейнерных приложений.      
    
https://prometheus.io/docs/introduction/overview/
   
Prometheus работает по принципу Pull модели      

 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3/Picture/Prometheus_pull.PNG">
</p>      
     

**Варианты сбора метрик** 
__1. Instrumentation__  - научить приложение из коробки отдавать метрики    
__2. Exporter__    

 <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3/Picture/Prometheus_exporter.PNG">
</p>      

Node Exporter — это экспортер Prometheus (один из), специально разработанный для сбора системных показателей с целевого устройства. Он работает на машине, которую вы хотите мониторить, и предоставляет различные показатели, такие как использование CPU, использование памяти, использование диска, статистика сети и другие. Эти метрики затем собираются Prometheus для дальнейшего анализа.    

Node Exporter обычно применяется для мониторинга состояния и производительности отдельных серверов или узлов в кластере. Это помогает выявить узкие места в ресурсах, обнаружить отказы оборудования и оптимизировать распределение ресурсов.     

https://prometheus.io/docs/instrumenting/exporters/    

__3. Pushgateway__ - на стороне сервиса пишем скрипт, который будет пушить нужные метрики на gateway откуда будет забирать метркии промифиус.


### Установка Prometheus

```
# Cкачиваем Prometheus
$ wget https://github.com/prometheus/prometheus/releases/download/v2.44.0/prometheus-2.44.0.linux-amd64.tar.gz

# Создаем пользователя и нужные каталоги, настраиваем для них владельцев
$ useradd --no-create-home --shell /bin/false prometheus
$ mkdir /etc/prometheus
$ mkdir /var/lib/prometheus
$ chown prometheus:prometheus /etc/prometheus
$ chown prometheus:prometheus /var/lib/prometheus

# Распаковываем архив, для удобства переименовываем директорию и копируем бинарники в /usr/local/bin
$ tar -xvzf prometheus-2.44.0.linux-amd64.tar.gz
$ mv prometheus-2.44.0.linux-amd64 prometheuspackage
$ cp prometheuspackage/prometheus /usr/local/bin/
$ cp prometheuspackage/promtool /usr/local/bin/

# Меняем владельцев у бинарников
$ chown prometheus:prometheus /usr/local/bin/prometheus
$ chown prometheus:prometheus /usr/local/bin/promtool

# По аналогии копируем библиотеки
$ cp -r prometheuspackage/consoles /etc/prometheus
$ cp -r prometheuspackage/console_libraries /etc/prometheus
$ chown -R prometheus:prometheus /etc/prometheus/consoles
$ chown -R prometheus:prometheus /etc/prometheus/console_libraries
```
```
# Создаем файл конфигурации
$ touch /etc/prometheus/prometheus.yml

global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus_master'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

$ chown prometheus:prometheus /etc/prometheus/prometheus.yml
```
```
#Настраиваем сервис
$ nano /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
$ systemctl daemon-reload
$ systemctl start prometheus
$ systemctl status prometheus 
```    
### Установка AlertManager

  <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3/Picture/Prometheus_Alert_Manager.PNG">
</p>   
     
```
# Скачиваем и распаковываем AlertManager
$ wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
$ tar zxf alertmanager-0.25.0.linux-amd64.tar.gz

#Создаем пользователя и нужные директории
$ useradd --no-create-home --shell /bin/false alertmanager
$ usermod --home /var/lib/alertmanager alertmanager
$ mkdir /etc/alertmanager
$ mkdir /var/lib/alertmanager

#Копируем бинарники из архива в /usr/local/bin и меняем владельца
$ cp alertmanager-0.25.0.linux-amd64/amtool /usr/local/bin/
$ cp alertmanager-0.25.0.linux-amd64/alertmanager /usr/local/bin/
$ cp alertmanager-0.25.0.linux-amd64/alertmanager.yml /etc/alertmanager/
$ chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
$ chown alertmanager:alertmanager /usr/local/bin/{alertmanager,amtool}
$ echo "ALERTMANAGER_OPTS=\"\"" > /etc/default/alertmanager
$ chown alertmanager:alertmanager /etc/default/alertmanager
$ chown -R prometheus:prometheus /var/lib/prometheus/alertmanager
```

```
#Настраиваем сервис
$ touch /etc/systemd/system/alertmanager.service

[Unit]
Description=Alertmanager Service
After=network.target prometheus.service
[Service]
EnvironmentFile=-/etc/default/alertmanager
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
 --config.file=/etc/alertmanager/alertmanager.yml \
 --storage.path=/var/lib/prometheus/alertmanager \
 $ALERTMANAGER_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
Restart=always
[Install]
WantedBy=multi-user.target

#Запускаем сервис
$ systemctl daemon-reload
$ systemctl start alertmanager
```
```
# Настраиваем правила
$ vim /etc/prometheus/rules.yml
groups:
- name: alert.rules
 rules:
 - alert: InstanceDown
 expr: up == 0
 for: 1m
 labels:
 severity: critical
 annotations:
 description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.'
 summary: Instance {{ $labels.instance }} down

# Проверяем валидность
$ /usr/local/bin/promtool check rules /etc/prometheus/rules.yml
```

```
# Обновляем конфиг prometheus
$ nano /etc/prometheus/prometheus.yml
global:
 scrape_interval: 10s
scrape_configs:
 - job_name: 'prometheus_master'
 scrape_interval: 5s
 static_configs:
 - targets: ['localhost:9090']
 - job_name: 'node_exporter_centos'
 scrape_interval: 5s
 static_configs:
 - targets: ['localhost:9100']

rule_files:
 - "rules.yml"
alerting:
 alertmanagers:
 - static_configs:
 - targets:
 - localhost:9093
# Рестартуем сервисы
$ systemctl restart prometheus
$ systemctl restart alertmanager
```
### Установка Node Exporter
```
#Скачиваем и распаковываем Node Exporter
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
$ tar xzfv node_exporter-1.5.0.linux-amd64.tar.gz
#Создаем пользователя, перемещаем бинарник в /usr/local/bin
$ useradd -rs /bin/false nodeusr
$ mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
#Создаем сервис
$ touch /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
#Запускаем сервис
$ systemctl daemon-reload
$ systemctl start node_exporter
$ 
```
```
#Обновляем конфигурацию Prometheus (делаем на хосте где установлен  Prometheus)
$ nano /etc/prometheus/prometheus.yml
global:
 scrape_interval: 10s
scrape_configs:
 - job_name: 'prometheus_master'
 scrape_interval: 5s
 static_configs:
 - targets: ['localhost:9090']
 - job_name: 'node_exporter_ubuntusrv'
 scrape_interval: 5s
 static_configs:
 - targets: ['192.168.1.211:9100']
#Перезапускаем сервис
$ systemctl restart prometheus
```

## GRAFANA
Для визуализации метрик можно использовать Grafana. Существует множество различных источников данных, которые поддерживает Grafana, один из них — Prometheus.   

### Установка
https://grafana.com/grafana/download    

```
sudo apt-get install -y adduser libfontconfig1 musl  - устанавливаем зависимости
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_11.2.2_amd64.deb - скачиваем пакет
sudo dpkg -i grafana-enterprise_11.2.2_amd64.deb  - устанавливаем
```
Grafana не стартует автоматически, после установки делаем следующее:   
systemctl daemon-reload   
systemctl enable grafana-server   - добавляем в автозагрузку   
systemctl start grafana-server



### Настройка
**Grafana** - слушает 3000 порт

1. Идем в web браузер и подключаемся к ip:3000 
2. Логин admin пароль admin
3. Home --> Connection --> Data Source --> add Data
4. Выбираем Prometheus, вводим ip где установлен prometheus ---> save&test
5. Идем в dashboard и создаем новый dashboard если нужно сделать его с нуля.

С Нуля тяжело создать новый dashboard, гораздо проще импортировать уже готовый, нажимаем import и переходим по ссылке https://grafana.com/grafana/dashboards/
1. Ищем нужный дашборд, к примеру мы хотим использовать Node exporter Full
2. Вводим его id и нажимаем load
3. В поле источник данных выбираем промифиус и нажимаем import

# ПРОМИФИУС
_ _ _
  <p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9C%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3/Picture/prometheus.PNG">
</p>        

  
**Prometheus** — это система мониторинга и оповещения с открытым исходным кодом, которая используется для сбора и хранения метрик из различных источников. Она создана для мониторинга высокодинамичных сред, таких как облачные приложения или микросервисы. С помощью Prometheus можно отслеживать и анализировать производительность и состояние ваших приложений и инфраструктуры.    

Prometheus может быть использован для мониторинга использования ресурсов на серверах, отслеживания времени ответа веб-сервисов, сбора метрик с баз данных и измерения производительности контейнерных приложений. 

**Node Exporter**
Node Exporter — это экспортер Prometheus, специально разработанный для сбора системных показателей с целевого устройства. Он работает на машине, которую вы хотите мониторить, и предоставляет различные показатели, такие как использование CPU, использование памяти, использование диска, статистика сети и другие. Эти метрики затем собираются Prometheus для дальнейшего анализа.    

Node Exporter обычно применяется для мониторинга состояния и производительности отдельных серверов или узлов в кластере. Это помогает выявить узкие места в ресурсах, обнаружить отказы оборудования и оптимизировать распределение ресурсов.


### Установка
```
sudo apt install prometheus prometheus-node-exporter
```
### Настройка
Конфиг находится /etc/prometheus/prometheus.yml  

По умолчанию Промифиус слушает порт 9090, Node-exporter 9100
```
 - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100'] - в эту строку мы добавляем хосты, которые планиурем мониторить
````

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
sudo /bin/systemctl daemon-reload   
shmel@test:~$ sudo /bin/systemctl enable grafana-server   

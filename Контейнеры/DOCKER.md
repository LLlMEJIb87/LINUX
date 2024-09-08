# DOCKER
_ _ _
Докер это программная платформа для контейнеризации приложений в OS.

## Архитектура
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/Docker_architektura.PNG">
</p>


**Компоненты** **Docker**
- Сервер (демон dockerd)
- Клиент (команда docker). Команды по операциям с образами, сетями, томами и контейнерами
- Клиент и сервер взаимодействует по REST API
   

**Объекты** **Docker**
1. Образы (images) — дистрибутивы для контейнеров
- Есть открытый реестр (registry) Docker Hub https://hub.docker.com/
- Можно собираты собственные образы на базе существующих
- Состоят из слоёв (layers)
<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/%D0%9A%D0%BE%D0%BD%D1%82%D0%B5%D0%B9%D0%BD%D0%B5%D1%80%D1%8B/%D0%9A%D0%B0%D1%80%D1%82%D0%B8%D0%BD%D0%BA%D0%B8/sloi.PNG">
</p>

2. Контейнеры (containers) — развёрнутые, рабочие образы
3. Сети (networks) — передача данных
4. Тома (volumes) — хранение данных

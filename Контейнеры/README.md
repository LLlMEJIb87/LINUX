# КОНТЕЙНЕРИЗАЦИЯ
_ _ _
- Метод легковесной виртуализации процессов
- В основе — изоляция процессов через cgroups, namespaces и другие механизмы
- Используется только одно ядро хостовой ОС
- Возможны ограничениā ресурсов по CPU, RAM, сети и т.д.
- Реализации:    
○ Docker    
○ LXC    
○ OpenVZ (Virtuozzo)    
    

Обыкновенно в контейнер записывают не более одного приложения и всё, что нужно для его  работы - библиотеки (зависимости). Из этого вытекают все плюсы контейнеризации - приложение "обособленно" от ОС, если в ОС происходят изменения, обновления и т.д то в контейнере ничего не изменится, все зависимости останутся прежними.

## Средства изоляции
1.Пространства имён (namespaces) - с помощью namespaces мы изолируем процессы в различных областях
- PID PID процессов
- NETWORK Сетевые устройства, стеки, порты и т.п.
- USER ID пользователей и групп
- MOUNT Точки монтирования
- IPC System V IPC, очереди сообщений POSIX
- UTS Имя хоста и доменное имя NIS
2. Контрольные группы (cgroups) ограничение ресурсов - контролируем выделяемы ресурс
- использование памяти, в том числе виртуальной
- приоритизация: разным группам можно выделить разное количество процессорного ресурса
- пропускной способности подсистемы ввода-вывода

## Инструменты
1. __Chroot (Changeroot)__ - это подмена корня файловой системы для процесса (системный вызов) подменяет местонахождение корня файловой системы, заключая программу в специально созданное ограниченное окружение (jail) Любые обращения к файлам за пределами этой системы будут запрещены.
    
- разбирает аргументы командной строки
- выполняет системный вызов chroot
- выполняет переход в другой каталог
- изменяет пользователя, от имени которого необходимо выполнить действия
- выполняет заданную команду, а если команды нет, то запускает интерпретатор командной строки по умолчанию

__Недостатки Chroot__
- Общее пространство процессов - процесс запущенный в chroot видит все остальные процессы
- Общая сеть - невозможно запустить изолированные серверы в разных chroot
- Нельзя назначить лимиты по ресурсам - в любой момент процесс из chroot может занять все ресурсы

__Где используется сейчас__
- Тестирование программ - chroot можно использовать для тестирования программ в безопасной среде, где они не могут повредить хост-систему
- Восстановление системы - chroot можно использовать для восстановления системы после сбоя или вредоносной атаки
- Управление разрешениями - сhroot можно использовать для ограничения доступа пользователей к определнным файлам и каталогам на сервере

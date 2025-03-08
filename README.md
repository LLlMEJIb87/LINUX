# ВВЕДЕНИЕ
__Операционная система__ - это программа которая отвечает за разделение времени между пользователями, за контроль оборудования и за загрузку новых программ в область памяти.   

__Ядро ОС__ - это центральная часть ОС (работает с оборудованием), которая обеспечивает приложениям координированный доступ до ресурсов компьютера ( Приложения не имеют прямой доступ к ресурсам). Linux имеет монолитное ядро (Драйвера работают в пространстве ядра) это увеличиваем производительность, но имеет недостаток - сбой в драйвере ведет к сбою в ядре   


__Структура__    

<p align="center">
<image src="https://github.com/LLlMEJIb87/LINUX/blob/main/Picture/struktura.PNG">
</p>     

**/** - корень    
Это главный каталог в системе Linux. По сути, это и есть файловая система Linux. Здесь нет дисков или чего-то подобного, как в Windows. Вместо этого, адреса всех файлов начинаются с корня, а дополнительные разделы, флешки или оптические диски подключаются в папки корневого каталога.     
Только пользователь root имеет право читать и изменять файлы в этом каталоге. Обратите внимание, что у пользователя root домашний каталог /root, но не сам /.    

__/bin__ - (binaries) бинарные файлы пользователя   
Этот каталог содержит исполняемые файлы. Здесь расположены программы, которые можно использовать в однопользовательском режиме или режиме восстановления. Одним словом, те утилиты, которые могут использоваться пока еще не подключен каталог /usr/. Это такие общие команды, как cat, ls, tail, ps и т д.    

__/sbin__ - (system binaries) системные исполняемые файлы    
Так же как и /bin, содержит двоичные исполняемые файлы, которые доступны на ранних этапах загрузки, когда не примонтирован каталог /usr. Но здесь находятся программы, которые можно выполнять только с правами суперпользователя. Это разные утилиты для обслуживания системы. Например, iptables, reboot, fdisk, ifconfig,swapon и т д.    

__/etc__ - (etcetera) конфигурационные файлы    
В этой папке содержатся конфигурационные файлы всех программ, установленных в системе. Кроме конфигурационных файлов, в системе инициализации Init Scripts, здесь находятся скрипты запуска и завершения системных демонов, монтирования файловых систем и автозагрузки программ. Структура каталогов linux в этой папке может быть немного запутанной, но предназначение всех их - настройка и конфигурация.    

__/dev__ - (devices) файлы устройств    
В Linux все, в том числе внешние устройства являются файлами. Таким образом, все подключенные флешки, клавиатуры, микрофоны, камеры - это просто файлы в каталоге /dev/. Этот каталог содержит не совсем обычную файловую систему. Структура файловой системы Linux и содержащиеся в папке /dev файлы инициализируются при загрузке системы, сервисом udev. Выполняется сканирование всех подключенных устройств и создание для них специальных файлов. Это такие устройства, как: /dev/sda, /dev/sr0, /dev/tty1, /dev/usbmon0 и т д.     

__/proc__ - (proccess) информация о процессах    
Это тоже не обычная файловая система, а виртуальная подсистема, динамически создаваемая ядром. Здесь содержится вся информация о запущенных процессах в реальном времени. По сути, это псевдофайловая система, содержащая подробную информацию о каждом процессе, его Pid, имя исполняемого файла, параметры запуска, доступ к оперативной памяти и так далее. Также здесь можно найти информацию об использовании системных ресурсов, например, /proc/cpuinfo, /proc/meminfo или /proc/uptime. Кроме файлов в этом каталоге есть большая структура папок linux, из которых можно узнать достаточно много информации о системе.    

__/var__ (variable) - Переменные файлы    
Название каталога /var говорит само за себя, он должен содержать файлы, которые часто изменяются. Размер этих файлов постоянно увеличивается. Здесь содержатся файлы системных журналов, различные кеши, базы данных и так далее. Дальше рассмотрим назначение каталогов Linux в папке /var/.     

__/var/log__ - Файлы логов     
Здесь содержатся большинство файлов логов всех программ, установленных в операционной системе. У многих программ есть свои подкаталоги в этой папке, например, /var/log/apache - логи веб-сервера, /var/log/squid - файлы журналов кеширующего сервера squid. Если в системе что-либо сломалось, скорее всего, ответы вы найдете здесь.    

__/var/lib__ - базы данных    
Еще один тип изменяемых файлов - это файлы баз данных, пакеты, сохраненные пакетным менеджером и т д.    

__/var/mail__ - почта    
В эту папку почтовый сервер складывает все полученные или отправленные электронные письма, здесь же могут находиться его логи и файлы конфигурации.    

__/var/spool__ - очереди    
Изначально, эта папка отвечала за очереди печати на принтере и работу набора программ cups.    

__/var/lock__ - файлы блокировок    
Здесь находятся файлы блокировок. Эти файлы означают, что определенный ресурс, файл или устройство занят и не может быть использован другим процессом. Apt-get, например, блокирует свою базу данных, чтобы другие программы не могли ее использовать, пока программа с ней работает.    

__/var/run__ - PID процессов     
Содержит файлы с PID процессов, которые могут быть использованы, для взаимодействия между программами. В отличие от каталога /run данные сохраняются после перезагрузки.    

__/tmp (temp)__ - Временные файлы     
В этом каталоге содержатся временные файлы, созданные системой, любыми программами или пользователями. Все пользователи имеют право записи в эту директорию.    
Файлы удаляются при каждой перезагрузке. Аналогом Windows является папка Windows\Temp, здесь тоже хранятся все временные файлы.     

__/usr__ - (user applications) Программы пользователя     
Это самый большой каталог с большим количеством функций. Тут наиболее большая структура каталогов Linux. Здесь находятся исполняемые файлы, исходники программ, различные ресурсы приложений, картинки, музыку и документацию.     

__/usr/bin/__ - Исполняемые файлы    
Содержит исполняемые файлы различных программ, которые не нужны на первых этапах загрузки системы, например, музыкальные плееры, графические редакторы, браузеры и так далее.    

__/usr/sbin/__    
Содержит двоичные файлы программ для системного администрирования, которые нужно выполнять с правами суперпользователя. Например, таких как Gparted, sshd, useradd, userdel и т д.    

__/usr/lib/__ - Библиотеки    
Содержит библиотеки для программ из /usr/bin или /usr/sbin.    

__/usr/local__ - Файлы пользователя     
Содержит файлы программ, библиотек, и настроек созданные пользователем. Например, здесь могут храниться программы собранные и установленные из исходников и скрипты, написанные вручную.     

__/home__ - Домашняя папка    
В этой папке хранятся домашние каталоги всех пользователей. В них они могут хранить свои личные файлы, настройки программ и т д. Например, /home/sergiy и т д. Если сравнивать с Windows, то это ваша папка пользователя на диске C, но в отличии от WIndows, home как правило размещается на отдельном разделе, поэтому при переустановке системы все ваши данные и настройки программ сохранятся.     

__/boot__ - Файлы загрузчика    
Содержит все файлы, связанные с загрузчиком системы. Это ядро vmlinuz, образ initrd, а также файлы загрузчика, находящие в каталоге /boot/grub.    

__/lib__ (library) - Системные библиотеки    
Содержит файлы системных библиотек, которые используются исполняемыми файлами в каталогах /bin и /sbin.    
Библиотеки имеют имена файлов с расширением *.so и начинаются с префикса lib*. Например, libncurses.so.5.7. Папка /lib64 в 64 битных системах содержит 64 битные версии библиотек из /lib. Эту папку можно сравнить с WIndows\system32, там тоже сгружены все библиотеки системы, только там они лежат смешанные с исполняемыми файлами, а здесь все отдельно.    

__/opt__ (Optional applications) - Дополнительные программы     
В эту папку устанавливаются проприетарные программы, игры или драйвера. Это программы созданные в виде отдельных исполняемых файлов самими производителями. Такие программы устанавливаются в под-каталоги /opt/, они очень похожи на программы Windows, все исполняемые файлы, библиотеки и файлы конфигурации находятся в одной папке.    

__/mnt__ (mount) - Монтирование    
В этот каталог системные администраторы могут монтировать внешние или дополнительные файловые системы.    

__/media__ - Съемные носители    
В этот каталог система монтирует все подключаемые внешние накопители - USB флешки, оптические диски и другие носители информации.     

__/srv__ (server) - Сервер    
В этом каталоге содержатся файлы серверов и сервисов. Например, могут содержаться файлы веб-сервера apache.    

__/run__ - процессы     
Еще один каталог, содержащий PID файлы процессов, похожий на /var/run, но в отличие от него, он размещен в TMPFS, а поэтому после перезагрузки все файлы теряются.     

__/sys (system)__ - Информация о системе    
Назначение каталогов Linux из этой папки - получение информации о системе непосредственно от ядра. Это еще одна файловая система организуемая ядром и позволяющая просматривать и изменить многие параметры работы системы, например, работу swap, контролировать вентиляторы и многое другое.    


## ПОТОКИ
В UNIX универсальный интерфейс взаимодействия между ПО это потоки текстовых данных. Каждая программа в LINUX имеет три стандартных потока данных:    
1. STDIN (0) - стандартный поток ввода. Любая программа даолжна работать с аргументами, эти аргументы она получает на свой стандартный поток ввода.
2. STDOUT (1) - стандартный поток вывода. После обработки аргументов, программа выведет все корректно отработанные данные в стандартный поток вывода
3. STDERR (2) - стандартный поток ошибок. Если в процессе работы программы произошли ошибки то они выводятся в стандартный поток ошибок

__Перенаправление потоков в файл__    
**>** - перезаписывает информацию    
**>>** - дополняет информацию     

- Перенаправление стандартного потока вывода в файл:
```
ls -l 1> stdout.txt  1> - перенаправляет поток только stdout
```
- Перенаправление стандартного потока ошибок в файл:
```
ls -l aafsf 2> stderr.txt 2> - перенапрвляет поток только stderr
```
- Перенаправление обоих потоков 
```
ls -l adadafaf ./ 2> stderr 1> stdout
```

- Перенаправление обоих потоков в один файл
```
ls -l asdfy ./ 1> file.txt 2>&1 - перенаправляем стандартный поток вывода в file и поток ошибок туда же & куда и поток вывода
```

__Перенаправление потока вывода одной программы на поток ввода другой__       
```
сat stdout | grep ^-  где | - перенаправляет поток , ^ начало строки, - все файлы
```
```
ls -l asdf ./ 2>&1 | grep file.txt cначала перенаправить поток 1 на поток ввода команды grep, далее поток 2 перенаправить туда же куда смотрит поток 1
```
```
ls -l asdf ./ 2>&1 1> /dev/null | grep asdf - стандартный поток ошибок отфильтрован grep, стандартный поток вывод перенаправляется  в dev/null - специальное устройтсва в linux которое подавляет вывод
```
## Операторы

__код возврата__ если код возврата 0 - то программа выполнена успешна, все что не 0 - ошибка
```
&? - посмотреть код возрата последней выполненой команды
```

**&&** - логическое И ( И то И то)
```
ls -l && echo OK - в начале сработае ls далее выполнится команда echo 
```
**||** - логическое ИЛИ ( команды будут выполняться жо тех пор покак одна из них не сработает
```
ls -l sdfgkdg || echo ERR - так как команда ls с ошибкой, то выполнится следующая команада
```
**;** - НЕ ИМЕЕТ ЗНАЧЕНИЯ ( программы будут выполняться вне зависимости от кода возврата
```
ls -l sdfgkdg ;  echo noOK 
```

## ПРОЦЕССЫ
Процесс это программа, которая выполняется в данный момент.   
Процесс это ресурс, который предоставляем нам ядро OS

В LINUX можнос осздать процесс исключительно из другого процесса. Процесс создается путем копирования существующего процесса.    

__Жизненный цикл процесса, системные вызовы для управления процессами__    

1. **fork**  - системный вызов, который создаёт новый процесс потомок из процесса родителя
- Стартует в одном процессе, завершается в двух
- процессу родителю возвращает код возврата равный PID дочернего процесса
- дочернему процессу возвращает код 0
2. **exec** - системный вызов заменяющий исполняемый код в процессе, после этого системного вызова мы получаем нужный нам процесс (способ запустить программу в Linux)
- Работает в существующем процессе
- Заменяет программу процесса на ту, что была передана exec() в качестве аргумента
- exec() - семейство системных вызовов (execl, execle, execlp, execv, execcve, execvp)
3. **exit** - системный вызвов завершающий работу программы 
- приводит к обычному завершению программы
- после завершения программы ее переменная status будет доступна родительскому процессу( status - код возврата)
- exit() - не завершает процесс
4.  **wait** - системный вызов, который считывает код возврата дочернего процесса в Linux (только родитель может считать код возврата)
- приостанавливает выполнение текущего процесса до тех пор, пока дочерний процесс не преркатит выполнение
- получает код возврата дочернего процесса
- после вызова wait() дочерний процесс полностью исчезает из системы


  __PID PPID__
  
  **PID** - уникальный номер процесса

  **PPID** - уникальный номер процесса родителя



  Исключения:
- **init** (systemd) PID 1 PPID 0 - процесс в пространстве пользователя
- **kthreadd** (Демон потоков ядра) PID 2 PPID 0 - процесс в пространстве ядра     

__ПРОЦЕСС СИРОТА__  
Процесс у которого по какой то причине не оказалось процесса родителя, ядро следит за такими процессами, их усыновит процесс init (1)

__ПРОЦЕСС ЗОМБИ__    

Это состояние процесса между системный вызовом exit и системный вызовом wait ( он не занимает каких-либо ресурсов). Образуются в том случае,когда процесс родитель не следит за процессами потомками ( не считывает код возврата)    

Процесс зомби можно убить сделав его сиротой.(т.е убить процесс родитель или заставить родителя послать wait)   

__ПРОЦЕСС ДЕМОН__    
Процесс работающий в фоновом режиме без прямого взаимодействия с пользователем.



__СИГНАЛЫ МЕЖПРОЦЕССНОГО ВЗАИМОДЕЙСТВИЯ__    

 **kill** - посылает указанный сигнал процессу. kill -signal pid
- SIGINT -2 (Сигнал прерывания с терминала ctrl-c)
- SIGTERM -15 (сигнал завершения, сигнал по умолчанию для утилиты kill)
- SIGKILL - 9 (безусловное завершение)

  ## ЗАГРУЗКА ОПЕРАЦИОННОЙ СИСТЕМЫ
1. Загрузка BIOS
2. BIOS смотрит в специальную область диска MBR (главная загрузочная запись). В современных системах используется структура GPT
- содержит код загрузчика и таблицы разделов
3. BIOS считывает загрузчик GRUB
4. Начинается загрузка ядра
5. Cистема инициализации (получаем самый первый процесс)
- определяет порядок загрузки служб

  ## ПОЛЬЗОВАТЕЛИ
Имя пользователя для Linux это строка с максимальным ограничением символов -32 содержащая псевдоним. OS идентифицирует пользователя по UID.    

В OS существуют 2 вида пользователей:
1. ROOT (UID 0)
2. Все остальные ( UID не 0 )
- служебные пользователи (apache, nginx и тп)
- обычные пользователи

/etc/passwd - здесь хранится информация о пользователях в системе linux и имеет вид   
root:x:0:0 - где root имя пользователя, x - не используется, 0 - uid, 0 - gid ( идентификатор группы)    

/etc/group - здесь хранится информация о группах пользователей и имеет вид:    
wheel:x:10:username   

**id** username - покажет информацию о пользователе (uid, gid)   

**usermod** -aG groupname username - добавить пользвателя username в обозначенную группу   

В OS LINUX есть 3 типа прав на использование файлов:
1. r - readable (чтение) - цифровое обозначение **4**
2. w - writeable (запись) - цифровое обозначение **2**
3. x - executeable (исполнение) - цифровое обозначение **1**

И 3 типа прав доступа к файлам:
1. u - user
2. g - group
3. o - other

**chmod** u-w filename or /dir - убрать правона запись для user    
**chmod** g-w filename or /dir - убрать право на запись для group    
**chmod** o+w filename or /dir - добавить право на запись для other    
**chmod** г+rwx,g+rwx,0-rwx filename or /dir - добавили все права пользователю, группе и убрали все права для всех остальных    
**chmod** -R a+rwx  /dir - добавить все права для директории и её содержимого

## Стандартные команды

pwd - посмотреть в какой директории мы находимся     

cat - вывести содержимое файла на экран      

ls  - посмотреть файлы в директории      
ls -l - посмотреть,что еть дирктория а что есть файл      
ls -l / - посмотреть директорию находясь в другой директории     
ls -la  - посмотреть скрытые файлы     
ls -li - посмотреть номера inode     
ls -lh - информация о размере файла будет отображена в килобайтах       

touch - cоздать файл       
mkdir test - cоздать директорию       
mkdir test/test1 - создать поддиректорию в директории (которая уже создана)       
mkdir -p tset2/test3/test4 - создать директорию с поддиректориями одновременно       

cd - сменить директорию      

rm - удалить файл       
rm *2 - удалить все что имеет цифру  2 на конце      
rm test* - удалить все с наименованием test       
rm -r - удаляет файлы вместе с катологом      

rmdir - удалить пустую директорию      

cp test dir/ - скопировать файл в директорию      
cp -r dir dir2/ - скопировать директорию      

mv test ./dir2/ - переместить файл     
mv dir2/ ./dir/ - переместить директорию      
mv test test_new - переименовать файл     

type ls - обзор программа ( внешняя или внутренняя)      
which ls - покажет путь к программе      
who - кто работает на сервере      
man rm - открыть мануал ( более информативно)     
rm --help - открыть мануал    

tail -n 50 - покажет последние 50 строк файла

## Процессы
**ss** -ntlpe - посмотреть какие процессы слушают какие порты    
**ps** (от англ. process status) -afx  - покажет запущенные процессы


## Полезные утилиты для доп установки
**tree** -покажет структру папок и их вложения

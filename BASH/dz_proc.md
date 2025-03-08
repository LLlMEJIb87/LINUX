# Домашняя работа на тему "Управление процессами"
## Задание №1 Написать свою реализацию ps ax используя анализ /proc
1. Команда ps ax в Linux выводит список всех активных процессов в системе. Она показывает такие данные, как:

- PID (идентификатор процесса),
- TTY (терминал, если есть),
- STAT (состояние процесса),
- TIME (время выполнения),
- CMD (команда, которая запустила процесс).

Наша цель — получить похожую информацию, анализируя /proc

2. /proc — это виртуальная файловая система в Linux, которая содержит информацию о состоянии системы и запущенных процессах. Для каждого процесса в /proc есть своя папка, названная по его PID (например, /proc/1234). Внутри этих папок находятся файлы с данными о процессе, такие как:

- /proc/[PID]/stat — основная информация о процессе (PID, имя, состояние и т.д.),
- /proc/[PID]/cmdline — команда, которая запустила процесс.

 __Пишем скрипт__      

Чтобы написать скрипт, нам нужно:

1. Считать список всех процессов из /proc.
2. Извлечь нужные данные ( PID, имя команды, состояние).
3. Вывести их в удобном формате.

```
#!/bin/bash
for pid in /proc/[0-9]* #Запускаем цикл, который ищет папки в /proc, названия которых начинаются с цифр(PID процессов)
    if [ -d "$pid" ]; then #Убеждаемся, что процесс еще жив(Есть директория)
        pid_num=$(basename "$pid") #Получаем только номер PID бещ пути к директории (обрезаем путь)
        if [ -f "$pid/cmdline" ]; then #Проверяем наличие файла cmdline
            cmd=$(tr '\0' ' ' < "$pid/cmdline") #Cохраняем содержимое файла в переменную cmd
            if [ -z "$cmd" ]; then #Проверяем, что файл не пустой
                cmd=$(awk '{print $2}' "$pid/stat" | tr -d '()') #Если cmdline пустой, то берем имя процесса из /proc/[PID]/stat
            fi
            state=$(awk '{print $3}' "$pid/stat") #Извлекам поле с состоянием процесса
            ppid=$(awk '{print $4}' "$pid/stat")  #Извлекам поле с PPID
            tty_nr=$(awk '{print $7}' "$pid/stat") #Извлекам поле с TTY
            if [ "$tty_nr" -eq 0 ]; then #Проверяем равен ли TTY 0
                tty="?" # Если равен то выводим ?
            else
                tty="tty$tty_nr" #если не равен, то выводим идентификатор терминала
            fi
            echo "PID: $pid_num  PPID: $ppid  STAT: $state  TTY: $tty  CMD: $cmd" #выводим все собранные данные в одной строке
        fi
    fi
done
```
Результат работы скрипта:
```
root@lvm:~# ./ps.sh
PID: 1  PPID: 0  STAT: S  TTY: ?  CMD: /sbin/init 
PID: 10  PPID: 2  STAT: I  TTY: ?  CMD: kworker/0:0H-events_highpri
PID: 1104  PPID: 2  STAT: S  TTY: ?  CMD: z_null_iss
PID: 1105  PPID: 2  STAT: S  TTY: ?  CMD: z_null_int
PID: 1106  PPID: 2  STAT: S  TTY: ?  CMD: z_rd_iss
PID: 1107  PPID: 2  STAT: S  TTY: ?  CMD: z_rd_int
PID: 1108  PPID: 2  STAT: S  TTY: ?  CMD: z_wr_iss
PID: 1109  PPID: 2  STAT: S  TTY: ?  CMD: z_wr_iss_h
PID: 1110  PPID: 2  STAT: S  TTY: ?  CMD: z_wr_int
PID: 1111  PPID: 2  STAT: S  TTY: ?  CMD: z_wr_int_h
PID: 1112  PPID: 2  STAT: S  TTY: ?  CMD: z_fr_iss
PID: 1113  PPID: 2  STAT: S  TTY: ?  CMD: z_fr_int
PID: 1114  PPID: 2  STAT: S  TTY: ?  CMD: z_cl_iss
...
```

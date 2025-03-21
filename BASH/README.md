# BASH
**Bash** (Bourne Again Shell) — это командный интерпретатор (shell) и язык сценариев (скриптов) в UNIX-подобных операционных системах, таких как Linux.    
По сути, это удобный инструмент для управления компьютером с помощью текста, без графического интерфейса.    

https://www.gnu.org/software/bash/manual/bash.html#What-is-Bash_003f

## Конфигурационные файлы оболочки
● ~/.bashrc — переменные конкретного пользователя (non-login shell)     
● ~/.bash_profile — файл переменных для login shell    
● ~/.profile — настройки для пользователя (для любых оболочек)    
● /etc/environment — переменные окружения на системном уровне     
● /etc/bashrc — системные настройки Bash для локальных пользователей    
● /etc/profile — системные настройки оболочек, в том числе при удалённом входе      

## Командs в Bash
**Команда в Bash** — это строка текста, которую оболочка (shell) анализирует и выполняет.    
Команда может быть:

1. Встроенной (built-in) — выполняется прямо внутри Bash без создания нового процесса.
- Примеры: cd, echo, exit.
- Они работают быстрее, потому что выполняются в том же процессе, что и Bash.
2. Внешней (external) — это отдельная программа (исполняемый файл в системе).
- Примеры: ls, cat, ping.
- Bash создаёт новый процесс с помощью fork(), а затем запускает программу через execve().
3. Алиасом — это просто короткое название для другой команды.     
- Например, alias ll='ls -l'.
4. Функцией Bash — это набор команд, который можно переиспользовать.
```
myfunc() {
    echo "Привет, $1!"
}
myfunc Дмитрий  # Выведет: Привет, Дмитрий!
```

Как Bash выполняет команду?
1. Сначала он проверяет, встроенная ли это команда.
2. Если нет, ищет среди алиасов и функций.
3. Если ничего не найдено, ищет исполняемый файл в $PATH.
4. Если это внешний файл, создаёт новый процесс (fork + execve).     
Если сказать просто: если команда выполняется без создания нового процесса, это встроенная функция Bash, а если создаётся новый процесс — это внешняя программа.

## Переменные в Bash
Переменные в Bash — это именованные области памяти, в которых можно хранить данные (текст, числа, команды и другие значения) для использования в скриптах и командной строке.   
__Как работают переменные в Bash?__    
1. Создание переменной   
- В Bash переменная создаётся просто через = (без пробелов вокруг =):
```
name="Dmitry"
age=30
```
Значение хранится как строка, даже если это число.    
2. Использование переменной    
- Чтобы получить значение переменной, нужно использовать $:
```
echo "Имя: $name, Возраст: $age"
```
Выведет:
```
Имя: Dmitry, Возраст: 30
```
3. Изменение переменной
- Просто присваиваем новое значение:
```
name="Alex"
echo "Новое имя: $name"
```
4. Переменные окружения
- Если переменная нужна не только в текущем сеансе Bash, её можно сделать глобальной с помощью export:
```
export myvar="Hello"
```
Теперь она будет доступна и в дочерних процессах.    
5. Удаление переменной
```
unset myvar
```
6. Чтение ввода от пользователя
```
echo "Введите имя:"
read user_name
echo "Привет, $user_name!"
```
__Типы переменных в Bash__
- Обычные переменные — доступны только в текущем сеансе или скрипте.
- Переменные окружения — доступны во всех дочерних процессах.
- Специальные переменные (например, $?, $#, $@) — содержат информацию о выполнении команд.

Примеры переменных    
● $EDITOR — текстовый редактор по умолчанию    
● $UID — содержит реальный идентификатор, который устанавливается при логине.     
● $НОМЕ — Домашний каталог пользователя    
● $HОЅTTYPE — архитектура машины    
● $LC_CTYPE — внутренняя переменная, которая определяет кодировку символов $ОЅTYPE- тип ОС     
● $РАТН — путь поиска программ     
● $PPID — идентификатор родительского процесса    
● $SECONDS — время работы скрипта (в секундах)     
● $@ — параметры скрипта (столбик)     
● $* — все параметры скрипта (строка)     
● $0 — имя скрипта     
● $1, $2, $3, ... — параметры скрипта, по одному      
● $# — количество параметров     
● $? — статус выхода последней выполненной команды    
● $$ — PID оболочки     
● $! — PID последней выполненной в фоновом режиме команды    


Переменные в Bash помогают передавать данные между командами и делать скрипты динамическими.

## Подстановки в Bash
Подстановки в Bash — это механизм, позволяющий автоматически подставлять результат выполнения команд в другие команды или строки. Это очень полезно для динамического получения данных и использования их в других операциях.    

1. Подстановка команд (Command Substitution)
Подстановка команд позволяет вставить результат выполнения команды в команду или строку.    
```
current_time=$(date)
echo "Текущее время: $current_time"
```
Здесь выполняется команда date, и её результат (текущее время) сохраняется в переменную current_time, а затем выводится    
2. Подстановка параметров (Parameter Substitution)
- Использование переменной:
```
name="Dmitry"
echo "Привет, $name!"
```
- Условная подстановка:
```
echo ${name:-"Неизвестно"}  # Если переменная не пуста, выведет её значение, иначе — "Неизвестно"
```
- Удаление части строки:
```
filename="file.txt"
echo ${filename%.txt}  # Удаляет расширение .txt, выведет "file"
```
- Замена части строки:
```
path="/home/user/file.txt"
echo ${path/file.txt/document.txt}  # Заменит "file.txt" на "document.txt"
```
3. Подстановка арифметических выражений (Arithmetic Substitution)     
Подстановка арифметических выражений позволяет вычислять математические выражения и подставлять их результат. Для этого используется синтаксис $(( )).
```
x=5
y=10
sum=$((x + y))
echo "Сумма: $sum"  # Выведет "Сумма: 15"
```
4. Подстановка путей (Filename Expansion)     
Bash может автоматически расширять пути, заменяя символы *, ?, и [] на соответствующие файлы в директории. Это называется "глоббинг".
```
echo *.txt  # Выведет все файлы с расширением .txt в текущей директории
```
5. Подстановка процессов (Process Substitution)
Подстановка процессов позволяет использовать вывод процесса как файл.
```
diff <(ls /dir1) <(ls /dir2)
```
Здесь вывод команд ls /dir1 и ls /dir2 подставляется как временные файлы для команды diff.

## Условия в Bash
Условия в Bash позволяют выполнять определённые команды или блоки кода в зависимости от выполнения заданных условий. Они играют ключевую роль в управлении потоком выполнения скриптов, позволяя реализовывать ветвления и принимать решения на основе различных проверок.    
1. Оператор if: Основная конструкция для проверки условий.
```
if [ условие ]; then
    # команды, выполняемые если условие истинно
elif [ другое_условие ]; then
    # команды, выполняемые если другое_условие истинно
else
    # команды, выполняемые если ни одно из условий не истинно
fi
```
Пример:
```
#!/bin/bash
if [ -f "$1" ]; then
    echo "Файл $1 существует."
else
    echo "Файл $1 не найден."
fi
```
В этом примере проверяется существование файла, переданного в качестве первого аргумента скрипта.   

2. Оператор case: Используется для сравнения значения переменной с несколькими шаблонами.
```
case $variable in
    pattern1)
        # команды для pattern1
        ;;
    pattern2)
        # команды для pattern2
        ;;
    *)
        # команды по умолчанию
        ;;
esac
```
Пример:
```
#!/bin/bash
case "$1" in
    start)
        echo "Запуск сервиса..."
        ;;
    stop)
        echo "Остановка сервиса..."
        ;;
    restart)
        echo "Перезапуск сервиса..."
        ;;
    *)
        echo "Использование: $0 {start|stop|restart}"
        ;;
esac
```
Здесь в зависимости от первого аргумента ($1) выполняется соответствующая команда.    

__Проверка условий__    
Для проверки условий в Bash используются квадратные скобки [ ] или двойные квадратные скобки [[ ]]. Они позволяют выполнять различные проверки:
- Сравнение чисел:
```
  [ "$a" -eq "$b" ]  # true, если $a равно $b
  [ "$a" -ne "$b" ]  # true, если $a не равно $b
  [ "$a" -lt "$b" ]  # true, если $a меньше $b
  [ "$a" -le "$b" ]  # true, если $a меньше или равно $b
  [ "$a" -gt "$b" ]  # true, если $a больше $b
  [ "$a" -ge "$b" ]  # true, если $a больше или равно $b
```
- Сравнение строк:
```
  [ "$a" = "$b" ]    # true, если строки идентичны
  [ "$a" != "$b" ]   # true, если строки различны
  [ -z "$a" ]        # true, если строка пустая
  [ -n "$a" ]        # true, если строка не пустая
```
- Проверка файлов:
```
  [ -e "$file" ]     # true, если файл существует
  [ -f "$file" ]     # true, если это обычный файл
  [ -d "$file" ]     # true, если это каталог
  [ -r "$file" ]     # true, если файл доступен для чтения
  [ -w "$file" ]     # true, если файл доступен для записи
  [ -x "$file" ]     # true, если файл исполняемый
```
Для более сложных условий можно использовать логические операторы:

- Логическое "И": [ "$a" -gt 0 ] && [ "$a" -lt 10 ]
- Логическое "ИЛИ": [ "$a" -lt 0 ] || [ "$a" -gt 10 ]
- Логическое "НЕ": [ ! -f "$file" ]
## Циклы в Bash
В Bash доступны несколько типов циклов, позволяющих повторять выполнение команд или блоков кода:     
1. Цикл **for**    
Цикл for используется для итерации по списку значений или диапазону чисел.    
```
for элемент in список
do
    # команды
done
```
Пример:
```
#!/bin/bash
for i in {1..5}
do
    echo "Число: $i"
done

```
Вывод:
```
Число: 1
Число: 2
Число: 3
Число: 4
Число: 5
```
В этом примере цикл for перебирает числа от 1 до 5 и выводит их на экран.    

2. Цикл **while**      
Цикл while выполняет команды до тех пор, пока заданное условие истинно.   
```
while условие
do
    # команды
done
```
Пример:
```
#!/bin/bash
counter=1
while [ $counter -le 5 ]
do
    echo "Счётчик: $counter"
    ((counter++))
done
```
Вывод:
```
Счётчик: 1
Счётчик: 2
Счётчик: 3
Счётчик: 4
Счётчик: 5
```
Здесь цикл while продолжает выполняться, пока значение переменной counter не превысит 5.   

3. Цикл **until**    
Цикл until работает противоположно циклу while: он выполняет команды до тех пор, пока условие не станет истинным.   
```
until условие
do
    # команды
done
```
Пример:
```
#!/bin/bash
counter=1
until [ $counter -gt 5 ]
do
    echo "Счётчик: $counter"
    ((counter++))
done
```
Вывод:
```
Счётчик: 1
Счётчик: 2
Счётчик: 3
Счётчик: 4
Счётчик: 5
```
В этом примере цикл until выполняется до тех пор, пока counter не станет больше 5.

__Управление выполнением циклов__    
- break: Прерывает выполнение текущего цикла.
- continue: Пропускает текущую итерацию цикла и переходит к следующей.
```
#!/bin/bash
for i in {1..5}
do
    if [ $i -eq 3 ]; then
        continue  # пропустить число 3
    elif [ $i -eq 5 ]; then
        break  # выйти из цикла после числа 4
    fi
    echo "Число: $i"
done
```
Вывод:
```
Число: 1
Число: 2
Число: 4
```     
В этом примере число 3 пропускается, а цикл завершается после числа 4.

4. Цикл **Select**     
Цикл select используется для создания меню с выбором пользователя.

## Разделители полей
В Bash разделители полей играют ключевую роль в обработке строк и данных, позволяя разделять текст на отдельные компоненты для дальнейшей обработки. Основным механизмом для управления разделителями является переменная IFS (Internal Field Separator).    

__Переменная IFS__    
IFS определяет символы, которые Bash использует для разделения строки на слова или поля. По умолчанию, в IFS содержатся пробел, табуляция и символ новой строки (\n), что означает, что при чтении строки Bash будет разделять её на части, используя эти символы.    
Пример:
```
#!/bin/bash
# Устанавливаем IFS в запятую
IFS=','

# Читаем строку и разделяем её на массив
read -r -a array <<< "яблоко,банан,вишня"

# Выводим элементы массива
for item in "${array[@]}"
do
    echo "$item"
done
```
Вывод:
```
яблоко
банан
вишня
```

## Функции
Функции в Bash позволяют объединять блоки кода, выполняющие определённые задачи, под общим именем, что способствует улучшению структуры и читаемости скриптов. Они работают аналогично функциям в других языках программирования, позволяя повторно использовать код без необходимости его дублирования.    
Синтаксис объявления функции:    
- С использованием ключевого слова function:
```
function имя_функции {
    команды
}
```
- Без использования ключевого слова function:
```
имя_функции() {
    команды
}
```

## Массивы

Массивы в Bash позволяют хранить наборы значений в одной переменной, что упрощает работу с группами данных и делает скрипты более эффективными. Bash поддерживает два типа массивов: индексированные и ассоциативные.

## Математические операции
- let, expr — встроенные функции bash для арифметических операций
- Двойные скобки — позволяют проводить вычисления
- Операторы     
○ +,-, \*, / — сложение, вычитание, умножение, деление    
○ var++, var-- инкремент, декремент (на 1)    
○ % — остаток от деления    
- Примеры:     
○ let c=$a+$b    
○ a=$(( 4 + 5 ))     
○ a=$( expr 10 - 3 )    

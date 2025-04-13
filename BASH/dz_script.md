# Выполнение домашнего задания на тему BASH скрипты
1. Написать скрипт для CRON, который раз в час будет формировать письмо и отправлять на заданную почту.

```
#!/bin/bash

# === Настройки ===
LOGFILE="/var/log/nginx/access.log"
ERRORLOG="/var/log/nginx/error.log"
TMPDIR="/tmp/log_report"
LOCKFILE="/var/lock/log_report.lock"
STATEFILE="/var/tmp/log_report_last_run"
EMAIL="vagrant"
SUBJECT="Log Report from $(hostname)"

# === Предотвращение одновременного запуска ===
exec 200>"$LOCKFILE"
flock -n 200 || {
    echo "Скрипт уже выполняется." >&2
    exit 1
}

trap 'rm -rf "$TMPDIR"' EXIT
mkdir -p "$TMPDIR"

# === Временной диапазон ===
NOW=$(date +%s)
NOW_HUMAN=$(date "+%Y-%m-%d %H:%M:%S")

if [ -f "$STATEFILE" ]; then
    LAST_RUN=$(cat "$STATEFILE")
else
    # Если нет файла состояния, берем последние 60 минут
    LAST_RUN=$((NOW - 3600))
fi
LAST_RUN_HUMAN=$(date -d "@$LAST_RUN" "+%Y-%m-%d %H:%M:%S")
echo "$NOW" > "$STATEFILE"

# === Функции ===

get_log_lines_since_last_run() {
    # Используем временную метку и фильтруем log по дате
    find "$LOGFILE" -type f -exec sed -n "/$(date -d @$LAST_RUN "+%d\/%b\/%Y:%H")/,\$p" {} \;
}

get_error_lines_since_last_run() {
    find "$ERRORLOG" -type f -exec sed -n "/$(date -d @$LAST_RUN "+%d\/%b\/%Y:%H")/,\$p" {} \;
}

parse_top_ips() {
    awk '{print $1}' >> "$TMPDIR/ips.txt"
    sort "$TMPDIR/ips.txt" | uniq -c | sort -nr | head -10
}

parse_top_urls() {
    awk '{print $7}' >> "$TMPDIR/urls.txt"
    sort "$TMPDIR/urls.txt" | uniq -c | sort -nr | head -10
}

parse_http_codes() {
    awk '{print $9}' >> "$TMPDIR/codes.txt"
    sort "$TMPDIR/codes.txt" | uniq -c | sort -n
}
# === Обработка логов ===

get_log_lines_since_last_run > "$TMPDIR/access.log"
get_error_lines_since_last_run > "$TMPDIR/errors.log"

echo -e "Отчёт за период: $LAST_RUN_HUMAN — $NOW_HUMAN\n" > "$TMPDIR/report.txt"

echo -e "\n== ТОП IP адресов ==" >> "$TMPDIR/report.txt"
parse_top_ips < "$TMPDIR/access.log" >> "$TMPDIR/report.txt"

echo -e "\n== ТОП URL ==" >> "$TMPDIR/report.txt"
parse_top_urls < "$TMPDIR/access.log" >> "$TMPDIR/report.txt"

echo -e "\n== Ошибки (error.log) ==" >> "$TMPDIR/report.txt"
cat "$TMPDIR/errors.log" >> "$TMPDIR/report.txt"

echo -e "\n== HTTP коды ==" >> "$TMPDIR/report.txt"
parse_http_codes < "$TMPDIR/access.log" >> "$TMPDIR/report.txt"

# === Отправка письма ===
mail -s "$SUBJECT" "$EMAIL" < "$TMPDIR/report.txt"
```
2. Добавляем в cron
```
crontab -e
0 * * * * /home/vagrant/email.sh
```
3. Проверяем
```
vagrant@crone-host:~$ mail

"/var/mail/vagrant": 1 message 1 new
>N   1 root               Sat Apr 12 23:57  26/628   Log Report from crone-host
? 
Return-Path: <root@crone-host>
X-Original-To: vagrant
Delivered-To: vagrant@crone-host
Received: by crone-host (Postfix, from userid 0)
        id 5EEBE7DB1C; Sat, 12 Apr 2025 23:57:36 +0000 (UTC)
Subject: Log Report from crone-host
To: vagrant@crone-host
User-Agent: mail (GNU Mailutils 3.14)
Date: Sat, 12 Apr 2025 23:57:36 +0000
Message-Id: <20250412235736.5EEBE7DB1C@crone-host>
From: root <root@crone-host>

Отчёт за период: 2025-04-12 23:00:01 — 2025-04-12 23:57:36


== ТОП IP адресов ==
     12 192.168.1.88

== ТОП URL ==
     12 /

== Ошибки (error.log) ==

== HTTP коды ==
     12 304
```

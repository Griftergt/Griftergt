#!/bin/bash

# --- защита от одновременного запуска ---
exec 200> /tmp/web_report.lock
if ! flock -n 200; then
    echo "Скрипт уже запущен, выходим."
    exit 1
fi

ACCESS_LOG="/var/log/nginx/access.log"
ERROR_LOG="/var/log/nginx/error.log"
STATE_FILE="/tmp/web_report.last_run"

# --- временной диапазон ---
if [ -f "$STATE_FILE" ]; then
    START_TIME=$(cat "$STATE_FILE")
else
    START_TIME=$(date -d "1 hour ago" "+%Y-%m-%d %H:%M:%S")
fi
END_TIME=$(date "+%Y-%m-%d %H:%M:%S")
echo "$END_TIME" > "$STATE_FILE"

START_EPOCH=$(date -d "$START_TIME" +%s)
END_EPOCH=$(date -d "$END_TIME" +%s)

# --- функции ---
filter_access_by_time() {
    awk -v start="$START_EPOCH" -v end="$END_EPOCH" '
    {
        split($4, a, ":")
        gsub(/^\[/, "", a[1])
        gsub("/", " ", a[1])
        t = a[1] " " a[2] ":" a[3] ":" a[4]
        cmd = "date -d \"" t "\" +%s"
        cmd | getline epoch
        close(cmd)
        if (epoch >= start && epoch <= end) print $0
    }'
}

filter_errors_by_time() {
    awk -v start="$START_EPOCH" -v end="$END_EPOCH" '
    {
        d = $1
        gsub("/", "-", d)
        t = d " " $2
        cmd = "date -d \"" t "\" +%s"
        cmd | getline epoch
        close(cmd)
        if (epoch >= start && epoch <= end) print $0
    }'
}

# --- отчёт ---
echo "=== Отчёт о работе веб-сервера ==="
echo "Временной диапазон: с $START_TIME по $END_TIME"
echo ""

echo "=== Топ IP-адресов ==="
filter_access_by_time < "$ACCESS_LOG" | awk '{print $1}' | sort | uniq -c | sort -rn | head -5 | sed 's/^/    /'

echo ""
echo "=== Топ URL ==="
filter_access_by_time < "$ACCESS_LOG" | awk '{print $7}' | sort | uniq -c | sort -rn | head -5 | sed 's/^/    /'

echo ""
echo "=== HTTP-коды ответов ==="
filter_access_by_time < "$ACCESS_LOG" | awk '{print $9}' | sort | uniq -c | sort -rn | head -5 | sed 's/^/    /'

echo ""
echo "=== Ошибки веб-сервера (последние 10) ==="
filter_errors_by_time < "$ERROR_LOG" | tail -n 10 | sed 's/^/    /'
[web_report.sh](https://github.com/user-attachments/files/34861554/web_report.sh)
# Домашнее задание. 9
bash-скрипт

## Цель задания
- Написать bash-скрипт, который ежечасно формирует и отправляет на email отчёт о работе веб-сервера.

## Задание
1. Написать скрипт для CRON, который раз в час формирует отчёт и отправляет его на заданную почту;
2. Отчёт должен содержать:
- IP-адреса с наибольшим числом запросов (с момента последнего запуска);
- Запрашиваемые URL с наибольшим числом запросов (с момента последнего запуска);
- Ошибки веб-сервера/приложения (с момента последнего запуска);
- HTTP-коды ответов с указанием их количества (с момента последнего запуска).
3. Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.
4. В письме должен быть прописан обрабатываемый временной диапазон.

Письмо заменю файлом.
## Выполнение
Для большей наглядности я выставил тестовую виртуалку портом 80 наружу.
IP-адреса, URL, HTTP-коды, показываются одной командой с разницей лишь в номере поля.

```bash
root@ubuntu-server:/home/padmin# awk '{print $1}' "/var/log/nginx/access.log" | sort | uniq -c | sort -rn | head -5
      6 10.1.1.7
      2 31.173.87.149
      2 10.1.2.104
      1 95.108.213.206
      1 95.108.213.146
root@ubuntu-server:/home/padmin# awk '{print $1}' "/var/log/nginx/access.log" | sort | uniq -c | sort -rn | head -5
      6 10.1.1.7
      2 31.173.87.149
      2 10.1.2.104
      1 95.108.213.206
      1 95.108.213.146
root@ubuntu-server:/home/padmin# awk '{print $7}' "/var/log/nginx/access.log" | sort | uniq -c | sort -rn | head -5
      8 /
      3 /favicon.ico
      1 /robots.txt
      1 /boaform/admin/formLogin?username=user&psd=user
root@ubuntu-server:/home/padmin# awk '{print $9}' "/var/log/nginx/access.log" | sort | uniq -c | sort -rn | head -5
      5 404
      4 304
      4 200

```
Ошибки веб-сервера/приложения сгенерировал вручную и вывел через cat (потом заменл на tail)
```bash
root@ubuntu-server:/home/padmin# cat /var/log/nginx/error.log | grep error
2026/11/14 23:14:41 [error] 206618#206618: *11 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
2026/11/14 23:15:07 [error] 206619#206619: *12 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
```

Оберул в скрипт
```bash
root@ubuntu-server:/home/padmin# cat web_report.sh
#!/bin/bash

ACCESS_LOG="/var/log/nginx/access.log"
ERROR_LOG="/var/log/nginx/error.log"

echo "=== Топ IP-адресов ==="
awk '{print $1}' "$ACCESS_LOG" | sort | uniq -c | sort -rn | head -5

echo ""
echo "=== Топ URL ==="
awk '{print $7}' "$ACCESS_LOG" | sort | uniq -c | sort -rn | head -5

echo ""
echo "=== HTTP-коды ответов ==="
awk '{print $9}' "$ACCESS_LOG" | sort | uniq -c | sort -rn | head -5

echo ""
echo "=== Ошибки веб-сервера (последние 10) ==="
cat "$ERROR_LOG" | grep error
root@ubuntu-server:/home/padmin# ./web_report.sh
=== Топ IP-адресов ===
      6 10.1.1.7
      2 31.173.87.149
      2 10.1.2.104
      2 ::1
      1 95.108.213.206

=== Топ URL ===
      8 /
      3 /favicon.ico
      2 /secret.html
      1 /robots.txt
      1 /boaform/admin/formLogin?username=user&psd=user

=== HTTP-коды ответов ===
      5 404
      4 304
      4 200
      2 403

=== Ошибки веб-сервера (последние 10) ===
2026/11/14 23:14:41 [error] 206618#206618: *11 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
2026/11/14 23:15:07 [error] 206619#206619: *12 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"

```
Добавил блокировку одновременного запуска в начало скрипта и обратный временной диапазон.
```bash
root@ubuntu-server:/home/padmin# ./web_report.sh
Скрипт уже запущен, выходим.
root@ubuntu-server:/home/padmin# ./web_report.sh
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-14 23:35:42 по 2026-11-14 23:37:35

=== Топ IP-адресов ===
      6 10.1.1.7
      2 31.173.87.149
      2 10.1.2.104
      2 ::1
      1 95.108.213.206

=== Топ URL ===
      8 /
      3 /favicon.ico
      2 /secret.html
      1 /robots.txt
      1 /boaform/admin/formLogin?username=user&psd=user

=== HTTP-коды ответов ===
      5 404
      4 304
      4 200
      2 403

=== Ошибки веб-сервера (последние 10) ===
2026/11/14 23:14:41 [error] 206618#206618: *11 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
2026/11/14 23:15:07 [error] 206619#206619: *12 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"

```
Реализовал вывод отчета с момента последнего запуска.
```bash
root@ubuntu-server:/home/padmin# ./web_report.sh
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-14 23:42:17 по 2026-11-14 23:53:47

=== Топ IP-адресов ===
         10 10.1.1.7
          1 10.1.2.104

=== Топ URL ===
         11 /

=== HTTP-коды ответов ===
         11 304

=== Ошибки веб-сервера (последние 10) ===
root@ubuntu-server:/home/padmin# curl -s http://localhost/secret.html > /dev/null
root@ubuntu-server:/home/padmin# curl -s http://localhost/secret.html > /dev/null
root@ubuntu-server:/home/padmin# curl -s http://localhost/secret.html > /dev/null
root@ubuntu-server:/home/padmin# ./web_report.sh
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-14 23:53:47 по 2026-11-14 23:55:26

=== Топ IP-адресов ===
          3 ::1
          1 31.173.81.87

=== Топ URL ===
          3 /secret.html
          1 /

=== HTTP-коды ответов ===
          3 403
          1 304

=== Ошибки веб-сервера (последние 10) ===
    2026/11/14 23:54:42 [error] 206621#206621: *16 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/14 23:54:43 [error] 206622#206622: *17 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/14 23:54:44 [error] 206625#206625: *18 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"

```
Добавил запись в крон таким образом, что каждую минуту в файл отчета пишутся требуемые данные. Для наглядности.
```bash
root@ubuntu-server:/home/padmin# cat /var/log/web_report/report_20261115.log
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-14 23:55:26 по 2026-11-15 00:12:01

=== Топ IP-адресов ===
          1 5.255.231.181
          1 5.187.38.30
          1 213.180.203.122

=== Топ URL ===
          1 /SDK/webLanguage
          1 /robots.txt
          1 /

=== HTTP-коды ответов ===
          2 404
          1 200

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:12:01 по 2026-11-15 00:13:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:13:01 по 2026-11-15 00:14:01

=== Топ IP-адресов ===
         12 ::1

=== Топ URL ===
         12 /secret.html

=== HTTP-коды ответов ===
         12 403

=== Ошибки веб-сервера (последние 10) ===
    2026/11/15 00:13:25 [error] 206623#206623: *26 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:25 [error] 206618#206618: *27 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:26 [error] 206618#206618: *28 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:26 [error] 206618#206618: *29 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:27 [error] 206618#206618: *30 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:27 [error] 206618#206618: *31 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:27 [error] 206618#206618: *32 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:28 [error] 206618#206618: *33 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:28 [error] 206618#206618: *34 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
    2026/11/15 00:13:28 [error] 206618#206618: *35 open() "/var/www/html/secret.html" failed (13: Permission denied), client: ::1, server: _, request: "GET /secret.html HTTP/1.1", host: "localhost"
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:14:01 по 2026-11-15 00:15:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:15:01 по 2026-11-15 00:16:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:16:01 по 2026-11-15 00:17:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:17:01 по 2026-11-15 00:18:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:18:01 по 2026-11-15 00:19:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:19:01 по 2026-11-15 00:20:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===
=== Отчёт о работе веб-сервера ===
Временной диапазон: с 2026-11-15 00:20:01 по 2026-11-15 00:21:01

=== Топ IP-адресов ===

=== Топ URL ===

=== HTTP-коды ответов ===

=== Ошибки веб-сервера (последние 10) ===


```

```bash

```

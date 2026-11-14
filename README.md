[web_report.sh](https://github.com/user-attachments/files/34861580/web_report.sh)
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

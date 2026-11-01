# Домашнее задание. 8
Systemd - создание unit-файла

## Цель задания
- Научиться редактировать существующие и создавать новые unit-файлы.
## Задание
- 1 Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).;
- 2 Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).;
- 3 Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.
cat
## Выполнение
 

### 1. Написать service
Cоздаём файл с конфигурацией для сервиса
```bash
root@ubuntu-server:/home/padmin# cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log

```
Cоздаем лог
```bash
root@ubuntu-server:/home/padmin# cat /var/log/watchlog.log
#Log file for my watchlog service
‘ALERT’
```
Cоздаем скрипт
```bash
root@ubuntu-server:/home/padmin# cat /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi
```
Создаём юнит для сервиса
```bash
root@ubuntu-server:/home/padmin# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```
Создаём юнит для таймера
```bash
root@ubuntu-server:/home/padmin# cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
```
Запускаем timer
```bash
root@ubuntu-server:/home/padmin# systemctl start watchlog.timer
root@ubuntu-server:/home/padmin# systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
     Loaded: loaded (/etc/systemd/system/watchlog.timer; disabled; preset: enabled)
     Active: active (elapsed) since Sun 2026-11-01 18:19:17 UTC; 17s ago
...
```
Проверяем результат
```bash
root@ubuntu-server:/home/padmin# systemctl start watchlog.service
root@ubuntu-server:/home/padmin# systemctl restart watchlog.timer
root@ubuntu-server:/home/padmin# tail -n 1000 /var/log/syslog  | grep word
2026-11-01T18:28:58.894734+00:00 ubuntu-server root: Вс 01 ноя 2026 18:28:58 UTC: I foundword, Master!

```



### 2. Установить spawn-fcgi и создать unit-файл
Устанавливаем spawn-fcgi и необходимые для него пакеты
```bash
root@ubuntu-server:/home/padmin# apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
...

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

```
Создаём файл с настройками для будущего сервиса

```bash
root@ubuntu-server:/home/padmin/script# cat /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

```
Создаём юнит-файл
```bash
root@ubuntu-server:/home/padmin/script# cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

```
Убеждаемся, что все успешно работает
```bash
root@ubuntu-server:/home/padmin/script# systemctl start spawn-fcgi
root@ubuntu-server:/home/padmin/script# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; preset: enabled)
     Active: active (running) since Sun 2026-11-01 18:46:25 UTC; 5s ago
   Main PID: 27425 (php-cgi)
      Tasks: 33 (limit: 19092)
     Memory: 19.3M (peak: 19.9M)
        CPU: 30ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─27425 /usr/bin/php-cgi
             ├─27426 /usr/bin/php-cgi
             ├─27427 /usr/bin/php-cgi

...
 ├─27456 /usr/bin/php-cgi
             └─27457 /usr/bin/php-cgi

ноя 01 18:46:25 ubuntu-server systemd[1]: Started spawn-fcgi.service - Spawn-fcgi startu>

```
Получается unit-файл — это отображение init-скрипта на язык systemd.

```bash
root@ubuntu-server:/home/padmin/script# cat php_cgi | wc -l
74
```
### 3. Доработка unit-файлa
Создаём /etc/systemd/system/nginx@.service

```bash
root@ubuntu-server:/home/padmin/script# cat /etc/systemd/system/nginx@.service
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target

```


Создаём 2 доп. конфига nginx
- первы
```bash
root@ubuntu-server:/home/padmin/script# cat /etc/nginx/nginx-first.conf
# /etc/nginx/nginx-first.conf

user www-data;
worker_processes auto;
pid /run/nginx-first.pid;              # <-- УНИКАЛЬНЫЙ PID ДЛЯ ПЕРВОГО ИНСТАНСА

events {
    worker_connections 768;
}

http {
    # Базовые настройки, которые обычно есть в стандартном nginx.conf
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Логи (можно оставить стандартные или развести по файлам)
    access_log /var/log/nginx/access-first.log;
    error_log /var/log/nginx/error-first.log;

    # Ваш уникальный server-блок с портом 9001
    server {
        listen 9001;
        server_name localhost;

        location / {
            root /var/www/html;
            index index.html index.htm;
        }
    }

    # Если хотите, чтобы работали стандартные сайты из sites-enabled
    # (но тогда нужно быть осторожным, чтобы там не было listen 80)
    # include /etc/nginx/sites-enabled/*;
}

```
- Второй
```bash
root@ubuntu-server:/home/padmin/script# cat /etc/nginx/nginx-second.conf
# /etc/nginx/nginx-second.conf

user www-data;
worker_processes auto;
pid /run/nginx-second.pid;             # <-- УНИКАЛЬНЫЙ PID ДЛЯ ВТОРОГО ИНСТАНСА

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access-second.log;
    error_log /var/log/nginx/error-second.log;

    server {
        listen 9002;                    # <-- ДРУГОЙ ПОРТ (например, 9002)
        server_name localhost;

        location / {
            root /var/www/html;
            index index.html index.htm;
        }
    }
}


```
Проверка портов

```bash
root@ubuntu-server:/home/padmin/script# ss -tnulp | grep nginx
tcp   LISTEN 0      511          0.0.0.0:9001      0.0.0.0:*    users:(("nginx",pid=43147,fd=5),("nginx",pid=43146,fd=5),("nginx",pid=43145,fd=5),("nginx",pid=43144,fd=5),("nginx",pid=43143,fd=5),("nginx",pid=43142,fd=5),("nginx",pid=43141,fd=5),("nginx",pid=43140,fd=5),("nginx",pid=43139,fd=5))
tcp   LISTEN 0      511          0.0.0.0:9002      0.0.0.0:*    users:(("nginx",pid=43384,fd=5),("nginx",pid=43383,fd=5),("nginx",pid=43382,fd=5),("nginx",pid=43381,fd=5),("nginx",pid=43380,fd=5),("nginx",pid=43379,fd=5),("nginx",pid=43378,fd=5),("nginx",pid=43377,fd=5),("nginx",pid=43376,fd=5))
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=41421,fd=5),("nginx",pid=41420,fd=5),("nginx",pid=41419,fd=5),("nginx",pid=41418,fd=5),("nginx",pid=41416,fd=5),("nginx",pid=41415,fd=5),("nginx",pid=41414,fd=5),("nginx",pid=41413,fd=5),("nginx",pid=41412,fd=5))
tcp   LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=41421,fd=6),("nginx",pid=41420,fd=6),("nginx",pid=41419,fd=6),("nginx",pid=41418,fd=6),("nginx",pid=41416,fd=6),("nginx",pid=41415,fd=6),("nginx",pid=41414,fd=6),("nginx",pid=41413,fd=6),("nginx",pid=41412,fd=6))

```
Проверка процессов
```bash
root@ubuntu-server:/home/padmin/script# ps afx | grep nginx
  44993 pts/1    S+     0:00  |       |                   \_ grep --color=auto nginx
  41412 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
  41413 ?        S      0:00  \_ nginx: worker process
  41414 ?        S      0:00  \_ nginx: worker process
  41415 ?        S      0:00  \_ nginx: worker process
  41416 ?        S      0:00  \_ nginx: worker process
  41418 ?        S      0:00  \_ nginx: worker process
  41419 ?        S      0:00  \_ nginx: worker process
  41420 ?        S      0:00  \_ nginx: worker process
  41421 ?        S      0:00  \_ nginx: worker process
  43139 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf -g daemon on; master_process on;
  43140 ?        S      0:00  \_ nginx: worker process
  43141 ?        S      0:00  \_ nginx: worker process
  43142 ?        S      0:00  \_ nginx: worker process
  43143 ?        S      0:00  \_ nginx: worker process
  43144 ?        S      0:00  \_ nginx: worker process
  43145 ?        S      0:00  \_ nginx: worker process
  43146 ?        S      0:00  \_ nginx: worker process
  43147 ?        S      0:00  \_ nginx: worker process
  43376 ?        Ss     0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf -g daemon on; master_process on;
  43377 ?        S      0:00  \_ nginx: worker process
  43378 ?        S      0:00  \_ nginx: worker process
  43379 ?        S      0:00  \_ nginx: worker process
  43380 ?        S      0:00  \_ nginx: worker process
  43381 ?        S      0:00  \_ nginx: worker process
  43382 ?        S      0:00  \_ nginx: worker process
  43383 ?        S      0:00  \_ nginx: worker process
  43384 ?        S      0:00  \_ nginx: worker process

```



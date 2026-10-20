# Домашнее задание. Занятие 4. Практические навыки работы с ZFS

## Цель задания
научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS;

## Задание
1. Определить алгоритм с наилучшим сжатием:
- определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
- создать 4 файловых системы, на каждой применить свой алгоритм сжатия;
- для сжатия использовать либо текстовый файл, либо группу файлов.
2. Определить настройки пула.
- С помощью команды zfs import собрать pool ZFS.
- Командами zfs определить настройки:
3. Работа со снапшотами
- скопировать файл из удаленной директории;
- восстановить файл локально. zfs receive;
- найти зашифрованное сообщение в файле secret_message.

## Выполнение

### 1. Определить алгоритм с наилучшим сжатием:
определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы, на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.

Установка ZFS
```bash
apt install zfsutils-linux
```
 
Создание 4 пулов mirror с разными алгоритмами сжатиея
```bash
# Создание 4 пулов 
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi

# Разные алгоритмы сжатия
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4

# Проверка
zfs get all | grep compression
```
Результат проверки сжатия

```bash
root@ubuntu-24-04-live-server:/home/padmin# zfs list
NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  22.6M   329M  22.1M  /otus1
otus2  18.6M   333M  18.2M  /otus2
otus3  11.8M   340M  11.3M  /otus3
otus4  40.0M   312M  39.5M  /otus4
root@ubuntu-24-04-live-server:/home/padmin# zfs get all | grep compressratio | grep -v ref
otus1  compressratio         1.78x                      -
otus2  compressratio         2.16x                      -
otus3  compressratio         3.47x                      -
otus4  compressratio         1.00x                      -

```
лучший результат сжатия — gzip-9 (otus3, compressratio 3.47x)

### 2. Определить настройки пула

Импортировал готовый пул из архива и определил его параметры.

```bash
root@ubuntu-24-04-live-server:/home/padmin# wget -O archive.tar.gz --no-check-certificate 'https:                         //drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
--2026-10-19 16:54:37--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRa                         lPAwbNN1Bb&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 173.194.220.132, 2a00:14                         50:4010:c0a::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|173.194.220.132|:443...                          connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6,9M) [application/octet-stream]
Saving to: ‘archive.tar.gz’

archive.tar.gz           100%[===============================>]   6,94M  14,1MB/s    in 0,5s

2026-10-19 16:54:43 (14,1 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]

root@ubuntu-24-04-live-server:/home/padmin# ll
total 7144
drwxr-x--- 4 padmin padmin    4096 окт 19 16:54 ./
drwxr-xr-x 3 root   root      4096 окт 13 05:44 ../
-rw-r--r-- 1 root   root   7275140 дек  6  2023 archive.tar.gz
-rw------- 1 padmin padmin     158 окт 18 19:06 .bash_history
-rw-r--r-- 1 padmin padmin     220 мар 31  2024 .bash_logout
-rw-r--r-- 1 padmin padmin    3771 мар 31  2024 .bashrc
drwx------ 2 padmin padmin    4096 окт 13 05:45 .cache/
-rw-r--r-- 1 padmin padmin     807 мар 31  2024 .profile
drwx------ 2 padmin padmin    4096 окт 13 05:45 .ssh/
-rw-r--r-- 1 padmin padmin       0 окт 13 05:46 .sudo_as_admin_successful
-rw------- 1 padmin padmin     140 окт 19 16:41 .Xauthority
root@ubuntu-24-04-live-server:/home/padmin# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
root@ubuntu-24-04-live-server:/home/padmin# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
        (Note that they may be intentionally disabled if the
        'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
        some features will not be available without an explicit 'zpool upgrade'.
 config:

        otus                                ONLINE
          mirror-0                          ONLINE
            /home/padmin/zpoolexport/filea  ONLINE
            /home/padmin/zpoolexport/fileb  ONLINE
root@ubuntu-24-04-live-server:/home/padmin# zpool import -d zpoolexport/ otus

```

Определение настроек
```bash
root@ubuntu-24-04-live-server:/home/padmin# zfs get available otus # размер хранилища
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
root@ubuntu-24-04-live-server:/home/padmin# zfs get readonly otus  # тип (read-write)
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
root@ubuntu-24-04-live-server:/home/padmin# zfs get recordsize otus  # recordsize
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
root@ubuntu-24-04-live-server:/home/padmin# zfs get compression otus # сжатие
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
root@ubuntu-24-04-live-server:/home/padmin# zfs get checksum otus	# контрольная сумма
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```

 
### 3. Работа со снапшотами

  скопировал файл из удаленной директории
```bash
padmin@ubuntu-24-04-live-server:~/otus_task2.file1$ wget -O otus_task2.file --no-check-certificate 'https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download'
--2026-10-19 18:50:15--  https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 173.194.220.132, 2a00:1450:4010:c05::84
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|173.194.220.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5432736 (5,2M) [application/octet-stream]
Saving to: ‘otus_task2.file’

otus_task2.file                100%[==================================================>]   5,18M  16,5MB/s    in 0,3s

2026-10-19 18:50:17 (16,5 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]

padmin@ubuntu-24-04-live-server:~/otus_task2.file1$

```
Восстановление из снапшота
```bash
padmin@ubuntu-24-04-live-server:~/otus_task2.file1$ sudo zfs receive otus1/test@today < otus_task2.file
[sudo] password for padmin:
```
Зашифрованное сообщение в файле secret_message
```bash
padmin@ubuntu-24-04-live-server:~/otus_task2.file1$ find /otus1/test -name "secret_message"
/otus1/test/task1/file_mess/secret_message
padmin@ubuntu-24-04-live-server:~/otus_task2.file1$ cat /otus1/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
in

```



```


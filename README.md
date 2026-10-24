# Домашнее задание. 5. Работа с NFS

## Цель задания
Научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов.

## Задание
- запустить 2 виртуальных машины (сервер NFS и клиента);
- на сервере NFS должна быть подготовлена и экспортирована директория;
- в экспортированной директории должна быть поддиректория с именем `upload` с правами на запись в неё;
- экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);
- монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.

## Выполнение

### 1. Подготовка стенда
ВМ:
- клиент 10.1.1.71
- сервер 10.1.1.70

Установлено:
```bash
nfs-kernel-server 	#на сервер
nfs-common 		#на клиент
```
 
Сделать, чтобы сервер слушал udp
```bash
sudo nano /etc/nfs.conf

[nfsd]
udp=y
```
Дальше прошёл по методичке, и проверил что всё работает.
Этого оказалось достаточно для написания скрипта

### 2. Применение скрипта

На сервере 

```bash
padmin@ubuntu-server24-nfss:~$ chmod +x nfss_script.sh
padmin@ubuntu-server24-nfss:~$ sudo su
[sudo] password for padmin:
root@ubuntu-server24-nfss:/home/padmin# ls -l
total 4
-rwxrwxr-x 1 padmin padmin 1995 окт 24 16:07 nfss_script.sh
root@ubuntu-server24-nfss:/home/padmin# ./nfss_script.sh
==> 1. Установка nfs-kernel-server
    nfs-kernel-server уже установлен, пропускаю.
==> 2. Создание и настройка экспортируемой директории
drwxr-xr-x 3 nobody nogroup 4096 окт 24 16:08 /srv/share
drwxrwxrwx 2 nobody nogroup 4096 окт 24 16:08 /srv/share/upload
==> 3. Формирование /etc/exports
/srv/share 10.1.1.71/32(rw,sync,root_squash,no_subtree_check)
==> 4. Применение экспорта
==> 5. Включение и перезапуск nfs-server
==> 6. Проверка экспорта
/srv/share  10.1.1.71/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
==> 7. Проверка слушающих портов 2049 (tcp/udp) и 111 (tcp/udp)
udp   UNCONN 0      0            0.0.0.0:2049       0.0.0.0:*                                                             
udp   UNCONN 0      0            0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1650,fd=5),("systemd",pid=1,fd=103))
udp   UNCONN 0      0               [::]:2049          [::]:*                                                             
udp   UNCONN 0      0               [::]:111           [::]:*    users:(("rpcbind",pid=1650,fd=7),("systemd",pid=1,fd=105))
tcp   LISTEN 0      4096         0.0.0.0:111        0.0.0.0:*    users:(("rpcbind",pid=1650,fd=4),("systemd",pid=1,fd=102))
tcp   LISTEN 0      64           0.0.0.0:2049       0.0.0.0:*                                                             
tcp   LISTEN 0      4096            [::]:111           [::]:*    users:(("rpcbind",pid=1650,fd=6),("systemd",pid=1,fd=104))
tcp   LISTEN 0      64              [::]:2049          [::]:*                                                             

Готово. NFS-сервер настроен и экспортирует /srv/share для 10.1.1.71.
root@ubuntu-server24-nfss:/home/padmin# cd /srv/share/upload
root@ubuntu-server24-nfss:/srv/share/upload# touch check_file
root@ubuntu-server24-nfss:/srv/share/upload# ls -l
total 0
-rw-r--r-- 1 root root 0 окт 24 16:11 check_file
root@ubuntu-server24-nfss:/srv/share/upload# ls -l /srv/share/upload/
total 0
-rw-r--r-- 1 root   root    0 окт 24 16:11 check_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:12 client_file
root@ubuntu-server24-nfss:/srv/share/upload#
```

На клиенте
```bash
padmin@ubuntu-server24-nfsc:~$ chmod +x nfsc_script.sh
padmin@ubuntu-server24-nfsc:~$ sudo su
[sudo] password for padmin:
Sorry, try again.
[sudo] password for padmin:
root@ubuntu-server24-nfsc:/home/padmin# ls -l
total 4
-rwxr-xr-x 1 padmin padmin 2836 окт 24 16:06 nfsc_script.sh
root@ubuntu-server24-nfsc:/home/padmin# ./nfsc_script.sh
==> 1. Установка nfs-common
    nfs-common уже установлен, пропускаю.
==> 2. Подготовка точки монтирования
==> 3. Настройка /etc/fstab (NFSv3 + x-systemd.automount)
    Добавлена строка:
    10.1.1.70:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0
/dev/disk/by-id/dm-uuid-LVM-3r1jwJefkfGkseE2VeRGgnTKpW0R1rCFaTIb1CxVlsVIZJjfJGMpUFmN4bG75sEb / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/f9861772-d35e-4954-9ac4-aaf6a85319e6 /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
10.1.1.70:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0
==> 4. Перечитывание systemd и перезапуск remote-fs.target
==> 5. Проверка доступности экспорта с сервера (showmount)
Export list for 10.1.1.70:
/srv/share 10.1.1.71/32
==> 6. Инициируем автомонтирование (обращение к точке)
==> 7. Проверка монтирования
    Монтирование активно:
systemd-1 on /mnt type autofs (rw,relatime,fd=70,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=21654)
10.1.1.70:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.1.1.70,mountvers=3,mountport=36031,mountproto=udp,local_lock=none,addr=10.1.1.70)
==> 8. Проверка NFSv3 (vers=3)
    OK: используется NFSv3 (vers=3).

Готово. Клиент настроен на automount 10.1.1.70:/srv/share/ -> /mnt
Проверка: mount | grep /mnt
root@ubuntu-server24-nfsc:/home/padmin#  mount | grep /mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=70,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=21654)
10.1.1.70:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.1.1.70,mountvers=3,mountport=36031,mountproto=udp,local_lock=none,addr=10.1.1.70)
root@ubuntu-server24-nfsc:/home/padmin# cd /srv/share/upload
bash: cd: /srv/share/upload: No such file or directory
root@ubuntu-server24-nfsc:/home/padmin# ls -l /mnt/upload/
total 0
-rw-r--r-- 1 root root 0 окт 24 16:11 check_file
root@ubuntu-server24-nfsc:/home/padmin# touch /mnt/upload/client_file
root@ubuntu-server24-nfsc:/home/padmin# ls -l /mnt/upload/
total 0
-rw-r--r-- 1 root   root    0 окт 24 16:11 check_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:12 client_file

```

 
### 3. Проверка применения скрипта после перезагрузки

На клиенте
```bash
padmin@ubuntu-server24-nfsc:~$ sudo su
[sudo] password for padmin:
root@ubuntu-server24-nfsc:/home/padmin# showmount -a 10.1.1.70
All mount points on 10.1.1.70:
root@ubuntu-server24-nfsc:/home/padmin# ls /mnt/upload/
check_file  client_file
root@ubuntu-server24-nfsc:/home/padmin# mount | grep /mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=54,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=3854)
10.1.1.70:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.1.1.70,mountvers=3,mountport=36031,mountproto=udp,local_lock=none,addr=10.1.1.70)
root@ubuntu-server24-nfsc:/home/padmin# touch /mnt/upload/final_check
root@ubuntu-server24-nfsc:/home/padmin# ls -l /mnt/upload/
total 0
-rw-r--r-- 1 root   root    0 окт 24 16:11 check_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:12 client_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:29 final_check


```
На сервере
```bash
padmin@ubuntu-server24-nfss:~$ ls -l /srv/share/upload/
total 0
-rw-r--r-- 1 root   root    0 окт 24 16:11 check_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:12 client_file
-rw-r--r-- 1 nobody nogroup 0 окт 24 16:29 final_check
padmin@ubuntu-server24-nfss:~$ exportfs -s
exportfs: could not open /var/lib/nfs/.etab.lock for locking: errno 13 (Permission denied)
padmin@ubuntu-server24-nfss:~$ sudo su
[sudo] password for padmin:
root@ubuntu-server24-nfss:/home/padmin# exportfs -s
/srv/share  10.1.1.71/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
root@ubuntu-server24-nfss:/home/padmin# showmount -a 10.1.1.70
All mount points on 10.1.1.70:
10.1.1.71:/srv/share

```

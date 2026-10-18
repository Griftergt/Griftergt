# Домашнее задание. Занятие 3. Работа с LVM

## Цель задания
Создавать и управлять логическими томами в LVM.

## Задание
1. Уменьшить том под / до 8G.
2. Выделить том под /home.
3. Выделить том под /var - сделать в mirror.
4. /home - сделать том для снапшотов.
5. Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
6. Работа со снапшотами:
	сгенерить файлы в /home/;
	снять снапшот;
	удалить часть файлов;
	восстановиться со снапшота.


## Выполнение

### 1-4. Выделить тома.

Из-за огромного вывода при копировании корня начальное состояние терминала ушло в небытие.
Показываю результат. Оказалось, к тому же, что в chroot-окружении история команд Bash по умолчанию не сохраняется автоматически. 


```bash
root@ubuntu-24-04-live-server:/home/padmin# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                          2:0    1    4K  0 disk
sda                          8:0    0   32G  0 disk
├─sda1                       8:1    0    1M  0 part
├─sda2                       8:2    0    2G  0 part /boot
└─sda3                       8:3    0   30G  0 part
  ├─ubuntu--vg-LogVol_Home 252:0    0    2G  0 lvm  /home
  └─ubuntu--vg-ubuntu--lv  252:1    0    8G  0 lvm  /
sdb                          8:16   0   10G  0 disk
sdc                          8:32   0    2G  0 disk
sdd                          8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_0    252:2    0    4M  0 lvm
│ └─vg_var-lv_var          252:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0   252:3    0  952M  0 lvm
  └─vg_var-lv_var          252:6    0  952M  0 lvm  /var
sde                          8:64   0    1G  0 disk
├─vg_var-lv_var_rmeta_1    252:4    0    4M  0 lvm
│ └─vg_var-lv_var          252:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1   252:5    0  952M  0 lvm
  └─vg_var-lv_var          252:6    0  952M  0 lvm  /var
sr0                         11:0    1 1024M  1 rom
```


### 5. Прописать монтирование в fstab

Прописывал как в методичке. 
echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab


```bash
root@ubuntu-24-04-live-server:/home/padmin# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-JcRW36FLBipbHPniGjr0oXQ0G2dlYEikeXKOeUEzXpTc78CZIvPVYOxdwgRaBUS6 / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/6a9c00fa-da8b-495f-b62b-38ed65988d07 /boot ext4 defaults 0 1
UUID="90a1f4ef-dd7a-4f79-b555-240c258e2d26"  /var ext4 defaults 0 0
UUID="5c6958d9-a202-4495-9f76-55780141634c"  /home xfs defaults 0 0

```
В методичке ошибка или намеренно сделали так, чтобы поломать систему после финальной перезагрузки?
```bash
mkfs.ext4 /dev/ubuntu-vg/LogVol_Home
echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab

```
Форматируем изначально в ext4 а автоматически монтируем в xfs

 
### 6. Работа со снапшотами

   71  touch /home/file{1..20}
   72  lvcreate -L 100MB -s -n home_snap  /dev/ubuntu-vg/LogVol_Home
   73  rm -f /home/file{11..20}
   74  ll /home/
   75  umount /home
   76  lvconvert --merge /dev/ubuntu-vg/home_snap
   77  mount /dev/mapper/ubuntu--vg-LogVol_Home /home
   78  ls -al /home


```bash
root@ubuntu-24-04-live-server:/home/padmin# touch /home/file{1..20}
root@ubuntu-24-04-live-server:/home/padmin# lvcreate -L 100MB -s -n home_snap \
 /dev/ubuntu-vg/LogVol_Home
  Logical volume "home_snap" created.
root@ubuntu-24-04-live-server:/home/padmin# rm -f /home/file{11..20}
root@ubuntu-24-04-live-server:/home/padmin# ll /home/
total 28
drwxr-xr-x  4 root   root    4096 окт 18 13:42 ./
drwxr-xr-x 25 root   root    4096 окт 18 12:45 ../
-rw-r--r--  1 root   root       0 окт 18 13:42 file1
-rw-r--r--  1 root   root       0 окт 18 13:42 file10
-rw-r--r--  1 root   root       0 окт 18 13:42 file2
-rw-r--r--  1 root   root       0 окт 18 13:42 file3
-rw-r--r--  1 root   root       0 окт 18 13:42 file4
-rw-r--r--  1 root   root       0 окт 18 13:42 file5
-rw-r--r--  1 root   root       0 окт 18 13:42 file6
-rw-r--r--  1 root   root       0 окт 18 13:42 file7
-rw-r--r--  1 root   root       0 окт 18 13:42 file8
-rw-r--r--  1 root   root       0 окт 18 13:42 file9
drwx------  2 root   root   16384 окт 18 13:38 lost+found/
drwxr-x---  4 padmin padmin  4096 окт 18 13:20 padmin/
root@ubuntu-24-04-live-server:/home/padmin# umount /home
root@ubuntu-24-04-live-server:/home/padmin# lvconvert --merge /dev/ubuntu-vg/home_snap
  Merging of volume ubuntu-vg/home_snap started.
  ubuntu-vg/LogVol_Home: Merged: 100,00%
root@ubuntu-24-04-live-server:/home/padmin# mount /dev/mapper/ubuntu--vg-LogVol_Home /home
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
root@ubuntu-24-04-live-server:/home/padmin# ls -al /home
total 28
drwxr-xr-x  4 root   root    4096 окт 18 13:42 .
drwxr-xr-x 25 root   root    4096 окт 18 12:45 ..
-rw-r--r--  1 root   root       0 окт 18 13:42 file1
-rw-r--r--  1 root   root       0 окт 18 13:42 file10
-rw-r--r--  1 root   root       0 окт 18 13:42 file11
-rw-r--r--  1 root   root       0 окт 18 13:42 file12
-rw-r--r--  1 root   root       0 окт 18 13:42 file13
-rw-r--r--  1 root   root       0 окт 18 13:42 file14
-rw-r--r--  1 root   root       0 окт 18 13:42 file15
-rw-r--r--  1 root   root       0 окт 18 13:42 file16
-rw-r--r--  1 root   root       0 окт 18 13:42 file17
-rw-r--r--  1 root   root       0 окт 18 13:42 file18
-rw-r--r--  1 root   root       0 окт 18 13:42 file19
-rw-r--r--  1 root   root       0 окт 18 13:42 file2
-rw-r--r--  1 root   root       0 окт 18 13:42 file20
-rw-r--r--  1 root   root       0 окт 18 13:42 file3
-rw-r--r--  1 root   root       0 окт 18 13:42 file4
-rw-r--r--  1 root   root       0 окт 18 13:42 file5
-rw-r--r--  1 root   root       0 окт 18 13:42 file6
-rw-r--r--  1 root   root       0 окт 18 13:42 file7
-rw-r--r--  1 root   root       0 окт 18 13:42 file8
-rw-r--r--  1 root   root       0 окт 18 13:42 file9
drwx------  2 root   root   16384 окт 18 13:38 lost+found
drwxr-x---  4 padmin padmin  4096 окт 18 13:20 padmin

```


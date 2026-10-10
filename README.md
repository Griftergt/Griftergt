# Домашнее задание. Занятие 2. Работа с mdadm

## Цель задания
Научиться использовать утилиту для управления программными RAID-массивами в Linux.

## Текст задания
1. Добавить в виртуальную машину несколько дисков.
2. Собрать RAID-0/1/5/10 на выбор.
3. Сломать и почините RAID.
4. Создать GPT таблицу, пять разделов и смонтировать их в системе


## Выполнение

### 1. Добавление дисков.

Добавлено 6 дисков по гигабайту.
Пришлось поиграться с номерами устройств в гипервизоре, чтобы первый диск отображался как sda.


```bash
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   38G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk

```

### 2. Сборка RAID6

С первого раза не получилось собрать RAID из-за названия блочного устройства md

```bash
root@ubuntu-server-1:/home/padmin# mdadm --create --verbose /dev/md_r6 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}
To optimize recovery speed, it is recommended to enable write-intent bitmap, do you want to enable it now? [y/N]? y
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: /dev/md_r6 is an invalid name for an md device.  Try /dev/md/md_r6
root@ubuntu-server-1:/home/padmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                         2:0    1    4K  0 disk
loop0                       7:0    0 66.8M  1 loop /snap/core24/1643
loop1                       7:1    0  1.5M  1 loop /snap/hwctl/123
loop2                       7:2    0 50.1M  1 loop /snap/snapd/27710
loop3                       7:3    0  1.5M  1 loop /snap/hwctl/131
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   38G  0 lvm  /
sdb                         8:16   0    1G  0 disk
sdc                         8:32   0    1G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sdf                         8:80   0    1G  0 disk
sdg                         8:96   0    1G  0 disk

```
Собрал RAID из 5 дисков. 

```bash
root@ubuntu-server-1:/home/padmin# mdadm --create --verbose /dev/md6 -l 6 -n 5 /dev/sd{b,c,d,e,f}
To optimize recovery speed, it is recommended to enable write-intent bitmap, do you want to enable it now? [y/N]? y
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md6 started.
root@ubuntu-server-1:/home/padmin# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
fd0                         2:0    1    4K  0 disk
loop0                       7:0    0 66.8M  1 loop  /snap/core24/1643
loop1                       7:1    0  1.5M  1 loop  /snap/hwctl/123
loop2                       7:2    0 50.1M  1 loop  /snap/snapd/27710
loop3                       7:3    0  1.5M  1 loop  /snap/hwctl/131
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part  /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   38G  0 lvm   /
sdb                         8:16   0    1G  0 disk
└─md6                       9:6    0    3G  0 raid6
sdc                         8:32   0    1G  0 disk
└─md6                       9:6    0    3G  0 raid6
sdd                         8:48   0    1G  0 disk
└─md6                       9:6    0    3G  0 raid6
sde                         8:64   0    1G  0 disk
└─md6                       9:6    0    3G  0 raid6
sdf                         8:80   0    1G  0 disk
└─md6                       9:6    0    3G  0 raid6
sdg                         8:96   0    1G  0 disk

```

### 3. Поломка и починка RAID
Сломал

```bash
root@ubuntu-server-1:/home/padmin# mdadm /dev/md6 --fail /dev/sde
root@ubuntu-server-1:/home/padmin# cat /proc/mdstat
Personalities : [raid4] [raid5] [raid6]
md6 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]
      bitmap: 0/1 pages [0KB], 65536KB chunk

unused devices: <none>
root@ubuntu-server-1:/home/padmin# mdadm -D /dev/md6
/dev/md6:
           Version : 1.2
     Creation Time : Sat Oct 10 08:50:38 2026
        Raid Level : raid6
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Oct 10 08:52:12 2026
             State : clean, degraded
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : ubuntu-server-1:6  (local to host ubuntu-server-1)
              UUID : bf677b13:e8f0e3cb:b06514cd:d6647871
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync   /dev/sdf

       3       8       64        -      faulty   /dev/sde

```

Починил, подсунув другой диск.

```bash
root@ubuntu-server-1:/home/padmin# mdadm /dev/md6 --add /dev/sdg
mdadm: added /dev/sdg
root@ubuntu-server-1:/home/padmin# cat /proc/mdstat
Personalities : [raid4] [raid5] [raid6]
md6 : active raid6 sdg[5] sdf[4] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

unused devices: <none>
root@ubuntu-server-1:/home/padmin# mdadm -D /dev/md6
/dev/md6:
           Version : 1.2
     Creation Time : Sat Oct 10 08:50:38 2026
        Raid Level : raid6
        Array Size : 3139584 (2.99 GiB 3.21 GB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Oct 10 08:53:55 2026
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : ubuntu-server-1:6  (local to host ubuntu-server-1)
              UUID : bf677b13:e8f0e3cb:b06514cd:d6647871
            Events : 41

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       96        3      active sync   /dev/sdg
       4       8       80        4      active sync   /dev/sdf

```

### 4. Создание GPT таблицы, монтирование разделов в системе
```bash
root@ubuntu-server-1:/home/padmin# parted -s /dev/md6 mklabel gpt
bash: parted: command not found
root@ubuntu-server-1:/home/padmin# apt install parted
Installing:
  parted
 ...

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@ubuntu-server-1:/home/padmin# parted -s /dev/md6 mklabel gpt
root@ubuntu-server-1:/home/padmin# parted /dev/md6 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.

root@ubuntu-server-1:/home/padmin# parted /dev/md6 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.

root@ubuntu-server-1:/home/padmin# parted /dev/md6 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.

root@ubuntu-server-1:/home/padmin# parted /dev/md6 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

root@ubuntu-server-1:/home/padmin# parted /dev/md6 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

root@ubuntu-server-1:/home/padmin# for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md6p$i; done
mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: d3cb0b63-d0a4-45e6-90db-65b11b687b4b
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: 5c3d44d2-c531-4fcf-a346-53d79c22bbac
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: 52895e60-1576-4181-8add-70e24d13a8ee
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 157056 4k blocks and 39280 inodes
Filesystem UUID: db07b355-f82b-41ef-adeb-81cf38e634f7
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 156672 4k blocks and 39200 inodes
Filesystem UUID: a14403b4-4698-45fd-91e3-af5bbec47d2e
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@ubuntu-server-1:/home/padmin# mkdir -p /raid/part{1,2,3,4,5}
root@ubuntu-server-1:/home/padmin# ls -l /raid/
total 20
drwxr-xr-x 2 root root 4096 Oct 10 08:58 part1
drwxr-xr-x 2 root root 4096 Oct 10 08:58 part2
drwxr-xr-x 2 root root 4096 Oct 10 08:58 part3
drwxr-xr-x 2 root root 4096 Oct 10 08:58 part4
drwxr-xr-x 2 root root 4096 Oct 10 08:58 part5
root@ubuntu-server-1:/home/padmin# for i in $(seq 1 5); do mount /dev/md6p$i /raid/part$i; done
root@ubuntu-server-1:/home/padmin# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
fd0                         2:0    1     4K  0 disk
loop0                       7:0    0  66.8M  1 loop  /snap/core24/1643
loop1                       7:1    0   1.5M  1 loop  /snap/hwctl/123
loop2                       7:2    0  50.1M  1 loop  /snap/snapd/27710
loop3                       7:3    0   1.5M  1 loop  /snap/hwctl/131
sda                         8:0    0    40G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part  /boot
└─sda3                      8:3    0    38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0    38G  0 lvm   /
sdb                         8:16   0     1G  0 disk
└─md6                       9:6    0     3G  0 raid6
  ├─md6p1                 259:0    0   612M  0 part  /raid/part1
  ├─md6p2                 259:1    0 613.5M  0 part  /raid/part2
  ├─md6p3                 259:2    0   612M  0 part  /raid/part3
  ├─md6p4                 259:3    0 613.5M  0 part  /raid/part4
  └─md6p5                 259:4    0   612M  0 part  /raid/part5
sdc                         8:32   0     1G  0 disk
└─md6                       9:6    0     3G  0 raid6
  ├─md6p1                 259:0    0   612M  0 part  /raid/part1
  ├─md6p2                 259:1    0 613.5M  0 part  /raid/part2
  ├─md6p3                 259:2    0   612M  0 part  /raid/part3
  ├─md6p4                 259:3    0 613.5M  0 part  /raid/part4
  └─md6p5                 259:4    0   612M  0 part  /raid/part5
sdd                         8:48   0     1G  0 disk
└─md6                       9:6    0     3G  0 raid6
  ├─md6p1                 259:0    0   612M  0 part  /raid/part1
  ├─md6p2                 259:1    0 613.5M  0 part  /raid/part2
  ├─md6p3                 259:2    0   612M  0 part  /raid/part3
  ├─md6p4                 259:3    0 613.5M  0 part  /raid/part4
  └─md6p5                 259:4    0   612M  0 part  /raid/part5
sde                         8:64   0     1G  0 disk
sdf                         8:80   0     1G  0 disk
└─md6                       9:6    0     3G  0 raid6
  ├─md6p1                 259:0    0   612M  0 part  /raid/part1
  ├─md6p2                 259:1    0 613.5M  0 part  /raid/part2
  ├─md6p3                 259:2    0   612M  0 part  /raid/part3
  ├─md6p4                 259:3    0 613.5M  0 part  /raid/part4
  └─md6p5                 259:4    0   612M  0 part  /raid/part5
sdg                         8:96   0     1G  0 disk
└─md6                       9:6    0     3G  0 raid6
  ├─md6p1                 259:0    0   612M  0 part  /raid/part1
  ├─md6p2                 259:1    0 613.5M  0 part  /raid/part2
  ├─md6p3                 259:2    0   612M  0 part  /raid/part3
  ├─md6p4                 259:3    0 613.5M  0 part  /raid/part4
  └─md6p5                 259:4    0   612M  0 part  /raid/part5

```

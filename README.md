# Домашнее задание. Занятие 1. Обновление ядра системы

## Название задания
Обновление ядра Linux в Ubuntu вручную из mainline-репозитория.

## Текст задания
1. Запустить ВМ с Ubuntu.
2. Обновить ядро ОС на болееновую версию из mainline-репозитория.
3. Оформить отчет в README-файле в GitHub-репозитории.

**Дополнительное задание:** собрать ядро самостоятельно из исходных кодов.

## Выполнение

### 1. Исходное состояние системы

Была установлена Ubuntu 26.04.1 LTS. Проверена текущая версия ядра:

```bash
$ uname -r
7.0.0-34-generic
```
Проверено содержимое /boot:
```bash
$ ls -al /boot
total 130500
-rw-------  1 root root 11018537 Sep  2 09:35 System.map-7.0.0-34-generic
-rw-r--r--  1 root root   308473 Sep  2 09:35 config-7.0.0-34-generic
lrwxrwxrwx  1 root root       27 Sep 30 07:13 initrd.img -> initrd.img-7.0.0-34-generic
lrwxrwxrwx  1 root root       24 Sep 30 07:13 vmlinuz -> vmlinuz-7.0.0-34-generic
...
```
### 2. Выбор версии ядра
На странице https://kernel.ubuntu.com/mainline/ была выбрана версия — v7.1 (сборка 7.1.0-070100.202606141628), которая новее установленной 7.0.0-34.

### 3. Скачивание пакетов
Создана рабочая директория и скачаны четыре .deb-пакета для архитектуры amd64:
```bash
$ mkdir ~/kernel-7.1 && cd ~/kernel-7.1

$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-headers-7.1.0-070100_7.1.0-070100.202606141628_all.deb
$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-image-unsigned-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
$ wget https://kernel.ubuntu.com/mainline/v7.1/amd64/linux-modules-7.1.0-070100-generic_7.1.0-070100.202606141628_amd64.deb
```
### 4. Установка пакетов
```bash
$ sudo dpkg -i *.deb
```
Вывод (сокращённо):
```
Setting up linux-headers-7.1.0-070100 (7.1.0-070100.202606141628) ...
Setting up linux-modules-7.1.0-070100-generic (7.1.0-070100.202606141628) ...
Setting up linux-headers-7.1.0-070100-generic (7.1.0-070100.202606141628) ...
Setting up linux-image-unsigned-7.1.0-070100-generic (7.1.0-070100.202606141628) ...
I: /boot/vmlinuz is now a symlink to vmlinuz-7.1.0-070100-generic
I: /boot/initrd.img is now a symlink to initrd.img-7.1.0-070100-generic
```
<img width="965" height="699" alt="kernel panic" src="https://github.com/user-attachments/assets/9c009001-867a-45c3-89c3-a343e4a13f9e" />
### 5. Устранение неполадки: отсутствие initramfs

После первой перезагрузки система упала в Kernel Panic с ошибкой:
VFS: Unable to mount root fs on unknown-block(0,0)

Причина: при установке mainline-ядра не был автоматически сгенерирован initrd.img для нового ядра. В /boot отсутствовал файл initrd.img-7.1.0-070100-generic, при этом симлинк initrd.img указывал на несуществующий файл.

Решение: загрузился в старое рабочее ядро 7.0.0-34-generic через меню GRUB (Advanced options for Ubuntu), затем вручную сгенерировал initramfs и обновил конфигурацию загрузчика:

```bash
$ sudo update-initramfs -c -k 7.1.0-070100-generic
update-initramfs: Generating /boot/initrd.img-7.1.0-070100-generic

$ sudo update-grub
Found linux image: /boot/vmlinuz-7.1.0-070100-generic
Found initrd image: /boot/initrd.img-7.1.0-070100-generic
Found linux image: /boot/vmlinuz-7.0.0-34-generic
Found initrd image: /boot/initrd.img-7.0.0-34-generic
...
done

$ sudo reboot
```
### 6. Проверка результата

После перезагрузки система успешно загрузилась в новое ядро:

Содержимое /boot после успешной установки:
```bash
$ ls -al /boot
total 188844
-rw-------  1 root root 11899326 Jun 14 16:28 System.map-7.1.0-070100-generic
-rw-r--r--  1 root root   307299 Jun 14 16:28 config-7.1.0-070100-generic
lrwxrwxrwx  1 root root       31 Sep 30 07:59 initrd.img -> initrd.img-7.1.0-070100-generic
-rw-------  1 root root 34750005 Sep 30 08:28 initrd.img-7.1.0-070100-generic
lrwxrwxrwx  1 root root       28 Sep 30 07:59 vmlinuz -> vmlinuz-7.1.0-070100-generic
-rw-------  1 root root 17457664 Jun 14 16:28 vmlinuz-7.1.0-070100-generic
lrwxrwxrwx  1 root root       27 Sep 30 07:59 initrd.img.old -> initrd.img-7.0.0-34-generic
lrwxrwxrwx  1 root root       24 Sep 30 07:59 vmlinuz.old -> vmlinuz-7.0.0-34-generic
...
```
### Задание со звёздочкой выполнил

Башхистори ниже.
```bash
 21  apt update
   22  apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev bc dwarves zstd rsync git wget patch
   23  cd /usr/src
   24  -l -la
   25  -l -l
   26  -ls -l
   27  ls -l
   28  sudo wget https://cdn.kernel.org/pub/linux/kernel/v7.x/linux-7.2.tar.xz
   29  ls -l
   30  sudo tar xf linux-7.2.tar.xz
   31  ls -l
   32  sudo chown -R $(id -u):$(id -g) linux-7.2
   33  top
   34  shutdown - h now
   35  shutdown -h now
   36  top
   37  cd /usr/src/linux-7.2
   38  cp /boot/config-$(uname -r) .config
   39  ls -l
   40  make olddefconfig
   41  scripts/config --disable SYSTEM_TRUSTED_KEYS
   42  scripts/config --disable SYSTEM_REVOCATION_KEYS
   43  make -j$(nproc)
   44  apt update
   45  apt install -y libdw-dev
   46  make -j$(nproc)
   47  ls -l
   48  ls -lф
   49  ls -la
   50  nano .config
   51  vi .config
   52  vi /.config
   53  ls -la .config
   54  apt install nano
   55  nano .config
   56  grep -n "SYSTEM_TRUSTED_KEYS\|SYSTEM_REVOCATION_KEYS" .config
   57  sed -i 's/^CONFIG_SYSTEM_TRUSTED_KEYS=.*/CONFIG_SYSTEM_TRUSTED_KEYS=""/' .config
   58  sed -i 's/^CONFIG_SYSTEM_REVOCATION_KEYS=.*/CONFIG_SYSTEM_REVOCATION_KEYS=""/' .config
   59  grep -n "SYSTEM_TRUSTED_KEYS\|SYSTEM_REVOCATION_KEYS" .config
   60  make -j$(nproc)
   61  df -h
   62  lsblk
   63  vgs
   64  lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
   65  resize2fs /dev/ubuntu-vg/ubuntu-lv
   66  df -h /
   67  lvs
   68  blockdev --getsize64 /dev/ubuntu-vg/ubuntu-lv
   69  blockdev --getsize64 /dev/mapper/ubuntu--vg-ubuntu--lv
   70  apt clean
   71  apt autoremove -y
   72  df -h /
   73  make clean
   74  df -h /
   75  lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
   76  resize2fs /dev/ubuntu-vg/ubuntu-lv
   77  df -h /
   78  make -j$(nproc) 2>&1 | tee build.log
   79  df -h /
   80  ls -l
   81  make clean
   82  df -h
   83  lablk
   84  lsblk
   85  parted /dev/sdb mklabel gpt
   86  apt install parted
   87  parted /dev/sdb mklabel gpt
   88  parted /dev/sdb mkpart primary ext4 0% 100%
   89  mkfs.ext4 /dev/sdb1
   90  lsblk
   91  mkdir -p /mnt/build
   92  mount /dev/sdb1 /mnt/build
   93  df -h /mnt/build
   94  mv /usr/src/linux-7.2 /mnt/build/
   95  ln -s /mnt/build/linux-7.2 /usr/src/linux-7.2
   96  cd /usr/src/linux-7.2
   97  pwd
   98  make -j$(nproc)
   99  ls -l
  100  df -H
  101  cd /usr/src/linux-7.2
  102  sudo make modules_install
  103  sudo make install
  104  uname -r
  105  ls -al /boot
  106  blkid
  107  cat /etc/fstab
  108  nano /etc/fstab
  109  tail -n 3 /etc/fstab
  110  mount -a
  111  df -h /mnt/build
  112  ls /mnt/build
  113  sudo reboot
  114  uname -r
  115  history
  ```
  ### Начало и конец вывода терминала:
```bash
root@ubuntu-server-1:/usr/src/linux-7.2# ls -l
total 1216
-rw-rw-r--   1 root root    496 Aug 16 21:32 COPYING
-rw-rw-r--   1 root root 108983 Aug 16 21:32 CREDITS
drwxrwxr-x  77 root root   4096 Aug 16 21:32 Documentation
-rw-rw-r--   1 root root   3129 Aug 16 21:32 Kbuild
-rw-rw-r--   1 root root    582 Aug 16 21:32 Kconfig
drwxrwxr-x   6 root root   4096 Aug 16 21:32 LICENSES
-rw-rw-r--   1 root root 916464 Aug 16 21:32 MAINTAINERS
-rw-rw-r--   1 root root  79208 Aug 16 21:32 Makefile
-rw-rw-r--   1 root root   6043 Aug 16 21:32 README
drwxrwxr-x  23 root root   4096 Aug 16 21:32 arch
drwxrwxr-x   3 root root   4096 Aug 16 21:32 block
drwxrwxr-x   2 root root   4096 Aug 16 21:32 certs
drwxrwxr-x   5 root root   4096 Aug 16 21:32 crypto
drwxrwxr-x 146 root root   4096 Aug 16 21:32 drivers
drwxrwxr-x  80 root root   4096 Aug 16 21:32 fs
drwxrwxr-x  33 root root   4096 Aug 16 21:32 include
drwxrwxr-x   2 root root   4096 Aug 16 21:32 init
drwxrwxr-x   2 root root   4096 Aug 16 21:32 io_uring
drwxrwxr-x   2 root root   4096 Aug 16 21:32 ipc
drwxrwxr-x  24 root root   4096 Aug 16 21:32 kernel
drwxrwxr-x  22 root root  12288 Aug 16 21:32 lib
drwxrwxr-x   7 root root   4096 Aug 16 21:32 mm
drwxrwxr-x  68 root root   4096 Aug 16 21:32 net
drwxrwxr-x  13 root root   4096 Aug 16 21:32 rust
drwxrwxr-x  47 root root   4096 Aug 16 21:32 samples
drwxrwxr-x  24 root root  12288 Aug 16 21:32 scripts
drwxrwxr-x  15 root root   4096 Aug 16 21:32 security
drwxrwxr-x  27 root root   4096 Aug 16 21:32 sound
drwxrwxr-x  48 root root   4096 Aug 16 21:32 tools
drwxrwxr-x   4 root root   4096 Aug 16 21:32 usr
drwxrwxr-x   4 root root   4096 Aug 16 21:32 virt
root@ubuntu-server-1:/usr/src/linux-7.2# make olddefconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  LEX     scripts/kconfig/lexer.lex.c
  YACC    scripts/kconfig/parser.tab.[ch]
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/conf
.config:1539:warning: symbol value 'm' invalid for NETFILTER_NETLINK
.config:8082:warning: symbol value 'm' invalid for SND_SOC_ACPI_AMD_SDCA_QUIRKS
#
# configuration written to .config
#
root@ubuntu-server-1:/usr/src/linux-7.2# ^C
root@ubuntu-server-1:/usr/src/linux-7.2# scripts/config --disable SYSTEM_TRUSTED_KEYS
root@ubuntu-server-1:/usr/src/linux-7.2# scripts/config --disable SYSTEM_REVOCATION_KEYS
root@ubuntu-server-1:/usr/src/linux-7.2# make -j$(nproc)
  SYNC    include/config/auto.conf
*
* Restart config...
*
*
* Certificates for signature checking
*
File name or PKCS#11 URI of module signing key (MODULE_SIG_KEY) [certs/signing_key.pem] certs/signing_key.pem
Type of module signing key to be generated
> 1. RSA (MODULE_SIG_KEY_TYPE_RSA)
  2. ECDSA (MODULE_SIG_KEY_TYPE_ECDSA)
  3. ML-DSA-44 (MODULE_SIG_KEY_TYPE_MLDSA_44)
  4. ML-DSA-65 (MODULE_SIG_KEY_TYPE_MLDSA_65)
  5. ML-DSA-87 (MODULE_SIG_KEY_TYPE_MLDSA_87)
choice[1-5?]: 1
Provide system-wide ring of trusted keys (SYSTEM_TRUSTED_KEYRING) [Y/?] y


....


root@ubuntu-server-1:/home/padmin# uname -r
7.2.0
```
Если требуется, то моу прислать огромный текстовый файл выводом всего терминала.


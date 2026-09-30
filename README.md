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

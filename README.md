# Домашнее задание. 7
Работа с загрузчиком

## Цель задания
- Научиться попадать в систему без пароля;
- Устанавливать систему с LVM и переименовывать в VG..
## Задание
- 1 Включить отображение меню Grub;
- 2 Попасть в систему без пароля несколькими способами;
- 3 Установить систему с LVM, после чего переименовать VG.

## Выполнение
 

### 1. Включить отображение меню Grub
Меняем 2 параметра в конфигурационном файле.
GRUB_TIMEOUT_STYLE
GRUB_TIMEOUT
```bash
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=15
GRUB_DISTRIBUTOR=`( . /etc/os-release; echo ${NAME:-Ubuntu} ) 2>/dev/null || echo Ubuntu`
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""
```


### 2. Попасть в систему без пароля несколькими способами


1 Способ
Открыл диспетчер Hyper-V и подключился к ВМ
Запустил ВМ и при выборе ядра для загрузки нажал e.
Дописал init=/bin/bash в конце строки начинающейся с linux
Нажал сtrl-x
<img width="1289" height="947" alt="1 Способ" src="https://github.com/user-attachments/assets/8595f096-7789-4453-8d20-0490563b3a96" />

2 Способ
Выбрал в меню пункт root
Система загрузилась. Причём ФС сразу была в rw
<img width="965" height="741" alt="2Способ" src="https://github.com/user-attachments/assets/944710ba-4ea2-4bcd-97c5-7a3da4763a50" />
<img width="971" height="749" alt="2 Способ" src="https://github.com/user-attachments/assets/0aa177c0-cc48-417a-bbf8-5c026fd4334d" />

 
### 3. Установка зависимостей сборки

Переименовал VG
```bash
vgrename ubuntu-vg ubuntu-otus
```
Поправил файл, /boot/grub/grub.cfg
На лекции говорили, что править руками не надо этот файл.
Но система загрузилась. Значит я нигде не ощибся.

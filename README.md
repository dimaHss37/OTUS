### Занятие 1. Обновление ядра системы

Цель:
научиться обновлять ядро в ОС Linux;

* Устанавливаем гостевую систему Ubuntu на виртуальной машине.

* Устанавливаем пакет SSH.
```sh
sudo install ssh
```


* Проверяем статус SSH сервиса.
```sh
systemctl status ssh.service
```

* Подключаемся по SSH к гостевой системе.

* Проверяем текущую версию ядра:
```sh
uname -r
6.17.0-6-generic
```

* Выводим тип процессора
```sh
uname -p
x86_64
```

* На сайте  [https://kernel.ubuntu.com/mainline](https://kernel.ubuntu.com/mainline) находим более свежую версию ядра ОС.

* Выбираем версию 6.18-rc1.

* Создаем новый каталог и скачиваем туда все пакеты ядра.
```sh
mkdir kernel

cd kernel

wget https://kernel.ubuntu.com/mainline/v6.18-rc1/amd64/linux-headers-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.18-rc1/amd64/linux-headers-6.18.0-061800rc1_6.18.0-061800rc1.202510122141_all.deb
wget https://kernel.ubuntu.com/mainline/v6.18-rc1/amd64/linux-image-unsigned-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.18-rc1/amd64/linux-modules-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb
```

* Проверяем скаченные файлы.
```sh
ls -l
-rw-rw-r-- 1 user user  3714398 Oct 13 01:18 linux-headers-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb
-rw-rw-r-- 1 user user 14536696 Oct 13 01:18 linux-headers-6.18.0-061800rc1_6.18.0-061800rc1.202510122141_all.deb
-rw-rw-r-- 1 user user 16691392 Oct 13 01:17 linux-image-unsigned-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb
-rw-rw-r-- 1 user user 97642793 Oct 31 21:10 linux-modules-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb

```

* Устанавливаем все пакеты.
```sh
sudo dpkg -i *.deb

Выбор ранее не выбранного пакета linux-headers-6.18.0-061800rc1.
(Чтение базы данных … на данный момент установлен 186761 файл и каталог.)
Подготовка к распаковке linux-headers-6.18.0-061800rc1_6.18.0-061800rc1.202510122141_all.deb …
Распаковывается linux-headers-6.18.0-061800rc1 (6.18.0-061800rc1.202510122141) …
Выбор ранее не выбранного пакета linux-headers-6.18.0-061800rc1-generic.
Подготовка к распаковке linux-headers-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb …
Распаковывается linux-headers-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
Выбор ранее не выбранного пакета linux-image-unsigned-6.18.0-061800rc1-generic.
Подготовка к распаковке linux-image-unsigned-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb …
Распаковывается linux-image-unsigned-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
Выбор ранее не выбранного пакета linux-modules-6.18.0-061800rc1-generic.
Подготовка к распаковке linux-modules-6.18.0-061800rc1-generic_6.18.0-061800rc1.202510122141_amd64.deb …
Распаковывается linux-modules-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
Настраивается пакет linux-headers-6.18.0-061800rc1 (6.18.0-061800rc1.202510122141) …
Настраивается пакет linux-headers-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
Настраивается пакет linux-modules-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
Настраивается пакет linux-image-unsigned-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
I: /boot/vmlinuz is now a symlink to vmlinuz-6.18.0-061800rc1-generic
I: /boot/initrd.img is now a symlink to initrd.img-6.18.0-061800rc1-generic
Обрабатываются триггеры для linux-image-unsigned-6.18.0-061800rc1-generic (6.18.0-061800rc1.202510122141) …
/etc/kernel/postinst.d/dracut:
dracut: Generating /boot/initrd.img-6.18.0-061800rc1-generic
/etc/kernel/postinst.d/kdump-tools:
kdump-tools: Generating /var/lib/kdump/initrd.img-6.18.0-061800rc1-generic
```

* Проверяем, что ядро появилось в /boot.
```sh
ls -l /boot/ | grep 6.18
-rw------- 1 root root 10196974 Oct 13 00:41 System.map-6.18.0-061800rc1-generic
-rw-r--r-- 1 root root   302364 Oct 13 00:41 config-6.18.0-061800rc1-generic
lrwxrwxrwx 1 root root       35 Oct 31 21:25 initrd.img -> initrd.img-6.18.0-061800rc1-generic
-rw------- 1 root root 45569385 Oct 31 21:25 initrd.img-6.18.0-061800rc1-generic
lrwxrwxrwx 1 root root       32 Oct 31 21:25 vmlinuz -> vmlinuz-6.18.0-061800rc1-generic
-rw------- 1 root root 16654527 Oct 13 00:41 vmlinuz-6.18.0-061800rc1-generic
```


* Перезагружаем ОС и выбираем загрузку с новым ядром 6.18.
<img width="1465" height="927" alt="Снимок экрана от 2025-10-31 21-33-53" src="https://github.com/user-attachments/assets/6baed7dd-1b13-4dee-a713-6ad40dee65b7" />

* Проверяем текущую версию ядра:
<img width="878" height="278" alt="Снимок экрана от 2025-10-31 21-47-37" src="https://github.com/user-attachments/assets/f2ce6465-8475-45b7-a570-4ef146b0a273" />

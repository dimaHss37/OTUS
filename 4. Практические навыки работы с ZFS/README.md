* Создаем виртуальную машину

Создаём виртуальную машину, помимо системного диска добавляем 8 дополнительных дисков по 512 MB.

1. ## Определение алгоритма с наилучшим сжатием

* Смотрим список всех дисков, которые есть в виртуальной машине:
```sh
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
vda    253:0    0   512M  0 disk
vdb    253:16   0    20G  0 disk
├─vdb1 253:17   0     1M  0 part
├─vdb2 253:18   0   1,9G  0 part [SWAP]
└─vdb3 253:19   0  18,1G  0 part /
vdc    253:32   0   512M  0 disk
vdd    253:48   0   512M  0 disk
vde    253:64   0   512M  0 disk
vdf    253:80   0   512M  0 disk
vdg    253:96   0   512M  0 disk
vdh    253:112  0   512M  0 disk
vdi    253:128  0   512M  0 disk
```

* Установим пакет утилит для ZFS:
```sh
apt install zfsutils-linux
```

* Создаём пул из двух дисков в режиме RAID 1:
```sh
zpool create otus1 mirror /dev/vda /dev/vdc
```

* Создадим ещё 3 пула:
```sh
zpool create otus2 mirror /dev/vdd /dev/vde
zpool create otus3 mirror /dev/vdf /dev/vdg
zpool create otus4 mirror /dev/vdh /dev/vdi
```

* Смотрим информацию о пулах:
```sh
zpool list

NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   108K   480M        -         -     3%     0%  1.00x    ONLINE  -
otus2   480M   112K   480M        -         -     3%     0%  1.00x    ONLINE  -
otus3   480M   112K   480M        -         -     3%     0%  1.00x    ONLINE  -
otus4   480M   111K   480M        -         -     3%     0%  1.00x    ONLINE  -
```

* Добавим разные алгоритмы сжатия в каждую файловую систему:

```sh
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```

* Проверим, что все файловые системы имеют разные методы сжатия:
```sh
zfs get all | grep compression
otus1  compression           lzjb                       local
otus2  compression           lz4                        local
otus3  compression           gzip-9                     local
otus4  compression           zle                        local
```

* Скачаем один и тот же текстовый файл во все пулы:
```sh
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```

* Проверим, что файл был скачан во все пулы:
```sh
ll /otus*
/otus1:
total 22120
-rw-r--r--  1 root root 41189447 Nov  2 11:31 pg2600.converter.log

/otus2:
total 18019
-rw-r--r--  1 root root 41189447 Nov  2 11:31 pg2600.converter.log

/otus3:
total 10975
-rw-r--r--  1 root root 41189447 Nov  2 11:31 pg2600.converter.log

/otus4:
total 40259
-rw-r--r--  1 root root 41189447 Nov  2 11:31 pg2600.converter.log
```

* Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:
```sh
zfs list

NAME    USED  AVAIL  REFER  MOUNTPOINT
otus1  21.7M   330M  21.6M  /otus1
otus2  17.7M   334M  17.6M  /otus2
otus3  10.8M   341M  10.7M  /otus3
otus4  39.4M   313M  39.3M  /otus4
```
```sh
zfs get all | grep compressratio | grep -v ref

otus1  compressratio         1.82x                      -
otus2  compressratio         2.23x                      -
otus3  compressratio         3.67x                      -
otus4  compressratio         1.00x                      -
```

* Алгоритм gzip-9 самый эффективный по сжатию.

2.  ## Определение настроек пула.

* Скачиваем архив в домашний каталог:
```sh
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
```

* Разархивируем его:
```sh
tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

* Проверим, возможно ли импортировать данный каталог в пул:
```sh
zpool import -d zpoolexport/
  pool: otus
    id: 6554193320433390805
 state: ONLINE
status: Some supported features are not enabled on the pool.
	(Note that they may be intentionally disabled if the
	'compatibility' property is set.)
action: The pool can be imported using its name or numeric identifier, though
	some features will not be available without an explicit 'zpool upgrade'.
config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE
```

* Сделаем импорт данного пула к нам в ОС:
```sh
zpool import -d zpoolexport/ otus
```

```sh
zpool status

  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
config:

	NAME                         STATE     READ WRITE CKSUM
	otus                         ONLINE       0     0     0
	  mirror-0                   ONLINE       0     0     0
	    /root/zpoolexport/filea  ONLINE       0     0     0
	    /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

  pool: otus1
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	otus1       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vda     ONLINE       0     0     0
	    vdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	otus2       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdd     ONLINE       0     0     0
	    vde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	otus3       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdf     ONLINE       0     0     0
	    vdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
config:

	NAME        STATE     READ WRITE CKSUM
	otus4       ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    vdh     ONLINE       0     0     0
	    vdi     ONLINE       0     0     0

errors: No known data errors
```

* Запрос сразу всех параметром файловой системы:
```sh
zfs get all otus
NAME  PROPERTY              VALUE                      SOURCE
otus  type                  filesystem                 -
otus  creation              Пт мая 15  7:00 2020  -
otus  used                  2.04M                      -
otus  available             350M                       -
otus  referenced            24K                        -
otus  compressratio         1.00x                      -
otus  mounted               yes                        -
otus  quota                 none                       default
otus  reservation           none                       default
otus  recordsize            128K                       local
otus  mountpoint            /otus                      default
otus  sharenfs              off                        default
otus  checksum              sha256                     local
otus  compression           zle                        local
otus  atime                 on                         default
otus  devices               on                         default
otus  exec                  on                         default
otus  setuid                on                         default
otus  readonly              off                        default
otus  zoned                 off                        default
otus  snapdir               hidden                     default
otus  aclmode               discard                    default
otus  aclinherit            restricted                 default
otus  createtxg             1                          -
otus  canmount              on                         default
otus  xattr                 on                         default
otus  copies                1                          default
otus  version               5                          -
otus  utf8only              off                        -
otus  normalization         none                       -
otus  casesensitivity       sensitive                  -
otus  vscan                 off                        default
otus  nbmand                off                        default
otus  sharesmb              off                        default
otus  refquota              none                       default
otus  refreservation        none                       default
otus  guid                  14592242904030363272       -
otus  primarycache          all                        default
otus  secondarycache        all                        default
otus  usedbysnapshots       0B                         -
otus  usedbydataset         24K                        -
otus  usedbychildren        2.01M                      -
otus  usedbyrefreservation  0B                         -
otus  logbias               latency                    default
otus  objsetid              54                         -
otus  dedup                 off                        default
otus  mlslabel              none                       default
otus  sync                  standard                   default
otus  dnodesize             legacy                     default
otus  refcompressratio      1.00x                      -
otus  written               24K                        -
otus  logicalused           1020K                      -
otus  logicalreferenced     12K                        -
otus  volmode               default                    default
otus  filesystem_limit      none                       default
otus  snapshot_limit        none                       default
otus  filesystem_count      none                       default
otus  snapshot_count        none                       default
otus  snapdev               hidden                     default
otus  acltype               off                        default
otus  context               none                       default
otus  fscontext             none                       default
otus  defcontext            none                       default
otus  rootcontext           none                       default
otus  relatime              on                         default
otus  redundant_metadata    all                        default
otus  overlay               on                         default
otus  encryption            off                        default
otus  keylocation           none                       default
otus  keyformat             none                       default
otus  pbkdf2iters           0                          default
otus  special_small_blocks  0                          default
otus  prefetch              all                        default
otus  direct                standard                   default
otus  longname              off                        default
```

* Размер:
```sh
zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
```

* Тип:
```sh
zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
```

* Значение recordsize:
```sh
zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
```

* Тип сжатия (или параметр отключения):
```sh
zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
```

* Тип контрольной суммы:
```sh
zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
```


3. ## Работа со снапшотом, поиск сообщения от преподавателя.
* Скачаем файл, указанный в задании:
```sh
wget -O otus_task2.file --no-check-certificate [https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download](https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download)
```

* Восстановим файловую систему из снапшота:
```sh
zfs receive otus/test@today < otus_task2.file
```

* Ищем в каталоге /otus/test файл с именем “secret_message”:
```sh
find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
```

* Смотрим содержимое найденного файла:
```sh
cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```

ссылка на курс OTUS.
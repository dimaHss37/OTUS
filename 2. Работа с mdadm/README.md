* Добавляем в виртуальную машину диск.
```sh
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0     11:0    1 1024M  0 rom
vda    254:0    0   20G  0 disk
└─vda1 254:1    0   20G  0 part /home
                                /var/cache
                                /var/log
                                /
vdb    254:16   0    5G  0 disk
```
В нашем случае это "vdb".

Создадим разделы на диске "vdb"

```sh
sudo fdisk /dev/vdb
```
```sh
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0     11:0    1 1024M  0 rom
vda    254:0    0   20G  0 disk
└─vda1 254:1    0   20G  0 part /home
                                /var/cache
                                /var/log
                                /
vdb    254:16   0    5G  0 disk
├─vdb1 254:17   0    1G  0 part
├─vdb2 254:18   0    1G  0 part
└─vdb3 254:19   0    1G  0 part
```
Созданы разделы: vdb1, vdb2, vdb3.

* Собираем RAID-1
```sh
sudo mdadm --create /dev/md127 -l 1 -n 2 /dev/vdb1 /dev/vdb2                   
To optimalize recovery speed, it is recommended to enable write-indent bitmap, do you want to enable it now? [y/N]? y
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array [y/N]? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md127 started.
```

Проверим, что RAID собрался нормально:

```sh
cat /proc/mdstat                                                              
Personalities : [raid1]
md127 : active raid1 vdb2[1] vdb1[0]
      1046528 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

unused devices: <none>
```

```sh
lsblk
                                                                    
NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sr0        11:0    1 1024M  0 rom
vda       254:0    0   20G  0 disk
└─vda1    254:1    0   20G  0 part  /home
                                    /var/cache
                                    /var/log
                                    /
vdb       254:16   0    5G  0 disk
├─vdb1    254:17   0    1G  0 part
│ └─md127   9:127  0 1022M  0 raid1
├─vdb2    254:18   0    1G  0 part
│ └─md127   9:127  0 1022M  0 raid1
└─vdb3    254:19   0    1G  0 part
```

```sh
sudo mdadm -D /dev/md127                                                      
/dev/md127:
           Version : 1.2
     Creation Time : Sat Nov  8 14:47:37 2025
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Nov  8 14:47:54 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : dima-standardpc:127  (local to host dima-standardpc)
              UUID : 3b5b67a6:0b411aae:df60c9ac:e64baa66
            Events : 20

    Number   Major   Minor   RaidDevice State
       0     254       17        0      active sync   /dev/vdb1
       1     254       18        1      active sync   /dev/vdb2
```

* Сломать и починить RAID

Искусственно выводим диск "vdb2" из строя:
```sh
sudo mdadm /dev/md127 --fail /dev/vdb2 
```

Проверяем статус массива:

```sh
cat /proc/mdstat                                                             
Personalities : [raid1]
md127 : active raid1 vdb2[1](F) vdb1[0]
      1046528 blocks super 1.2 [2/1] [U_]
      bitmap: 0/1 pages [0KB], 65536KB chunk

unused devices: <none>
```

```sh
sudo mdadm -D /dev/md127   
                                                
/dev/md127:
           Version : 1.2
     Creation Time : Sat Nov  8 14:47:37 2025
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Nov  8 15:23:21 2025
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : bitmap

              Name : dima-standardpc:127  (local to host dima-standardpc)
              UUID : 3b5b67a6:0b411aae:df60c9ac:e64baa66
            Events : 22

    Number   Major   Minor   RaidDevice State
       0     254       17        0      active sync   /dev/vdb1
       -       0        0        1      removed

       1     254       18        -      faulty   /dev/vdb2
```

Удаляем “сломанный” диск из массива:
```sh
sudo mdadm /dev/md127 --remove /dev/vdb2 

mdadm: hot removed /dev/vdb2 from /dev/md127
```

Проверяем статус массива:
```sh
sudo mdadm -D /dev/md127   
                                               
/dev/md127:
           Version : 1.2
     Creation Time : Sat Nov  8 14:47:37 2025
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 1
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Nov  8 15:31:55 2025
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : dima-standardpc:127  (local to host dima-standardpc)
              UUID : 3b5b67a6:0b411aae:df60c9ac:e64baa66
            Events : 23

    Number   Major   Minor   RaidDevice State
       0     254       17        0      active sync   /dev/vdb1
       -       0        0        1      removed
```

Добавим к массиву новый диск "vdb3"
```sh
sudo mdadm /dev/md127 --add /dev/vdb3
                                   
mdadm: added /dev/vdb3
```

Проверяем статус массива:
```sh
sudo mdadm -D /dev/md127
                                                 
/dev/md127:
           Version : 1.2
     Creation Time : Sat Nov  8 14:47:37 2025
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Nov  8 15:36:18 2025
             State : clean, degraded, recovering
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : bitmap

    Rebuild Status : 18% complete

              Name : dima-standardpc:127  (local to host dima-standardpc)
              UUID : 3b5b67a6:0b411aae:df60c9ac:e64baa66
            Events : 27

    Number   Major   Minor   RaidDevice State
       0     254       17        0      active sync   /dev/vdb1
       2     254       19        1      spare rebuilding   /dev/vdb3
```

Идёт процесс восстановления массива.

```sh
sudo mdadm -D /dev/md127
                                                  
/dev/md127:
           Version : 1.2
     Creation Time : Sat Nov  8 14:47:37 2025
        Raid Level : raid1
        Array Size : 1046528 (1022.00 MiB 1071.64 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Sat Nov  8 15:36:33 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : dima-standardpc:127  (local to host dima-standardpc)
              UUID : 3b5b67a6:0b411aae:df60c9ac:e64baa66
            Events : 45

    Number   Major   Minor   RaidDevice State
       0     254       17        0      active sync   /dev/vdb1
       2     254       19        1      active sync   /dev/vdb3
```
После завершения процесса восстановления, массив имеет статус: "clean".

* Создаём GPT таблицу, пять разделов и монтируем их в системе.


```sh
sudo parted -s /dev/md127 mklabel gpt
```

Создаём 4 раздела:

```sh
sudo parted /dev/md127 mkpart primary ext4 0% 20%                         

sudo parted /dev/md127 mkpart primary ext4 20% 40%             

sudo parted /dev/md127 mkpart primary ext4 40% 80%             

sudo parted /dev/md127 mkpart primary ext4 80% 100% 
```

```sh
lsblk  
                                                                   
NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sr0            11:0    1 1024M  0 rom
vda           254:0    0   20G  0 disk
└─vda1        254:1    0   20G  0 part  /home
                                        /var/cache
                                        /var/log
                                        /
vdb           254:16   0    5G  0 disk
├─vdb1        254:17   0    1G  0 part
│ └─md127       9:127  0 1022M  0 raid1
│   ├─md127p3 259:0    0  409M  0 part
│   ├─md127p4 259:1    0  203M  0 part
│   ├─md127p1 259:2    0  203M  0 part
│   └─md127p2 259:3    0  205M  0 part
├─vdb2        254:18   0    1G  0 part
└─vdb3        254:19   0    1G  0 part
  └─md127       9:127  0 1022M  0 raid1
    ├─md127p3 259:0    0  409M  0 part
    ├─md127p4 259:1    0  203M  0 part
    ├─md127p1 259:2    0  203M  0 part
    └─md127p2 259:3    0  205M  0 part
```

Форматируем разделы:

```sh
sudo mkfs -t ext4 /dev/md127p1    
                                        
mke2fs 1.47.3 (8-Jul-2025)
Discarding device blocks: done
Creating filesystem with 207872 1k blocks and 52000 inodes
Filesystem UUID: 2bdc3bb6-c2a4-49ab-b05c-37f1ab748959
Superblock backups stored on blocks:
	8193, 24577, 40961, 57345, 73729, 204801

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
Аналогично для md127p2, md127p3 и md127p4.

Создаём каталоги для монтирования разделов:
```sh
sudo mkdir /mnt/{1,2,3,4}
```
Монтируем разделы в систему:
```sh
sudo mount /dev/md127p4 /mnt/4                                            
sudo mount /dev/md127p3 /mnt/3                                            
sudo mount /dev/md127p2 /mnt/2                                            
sudo mount /dev/md127p1 /mnt/1    
```

```sh
df -hT     
                                                               
Файловая система Тип      Размер Использовано  Дост Использовано% Cмонтировано в
dev              devtmpfs   1,9G            0  1,9G            0% /dev
run              tmpfs      2,0G         1,4M  2,0G            1% /run
/dev/vda1        btrfs       20G         6,0G   14G           31% /
tmpfs            tmpfs      2,0G            0  2,0G            0% /dev/shm
tmpfs            tmpfs      2,0G            0  2,0G            0% /tmp
tmpfs            tmpfs      1,0M            0  1,0M            0% /run/credentials/systemd-journald.service
/dev/vda1        btrfs       20G         6,0G   14G           31% /var/log
/dev/vda1        btrfs       20G         6,0G   14G           31% /var/cache
/dev/vda1        btrfs       20G         6,0G   14G           31% /home
tmpfs            tmpfs      392M         152K  392M            1% /run/user/1000
/dev/md127p4     ext4       185M          64K  171M            1% /mnt/4
/dev/md127p3     ext4       374M         116K  349M            1% /mnt/3
/dev/md127p2     ext4       187M          65K  173M            1% /mnt/2
/dev/md127p1     ext4       185M          64K  171M            1% /mnt/1
```
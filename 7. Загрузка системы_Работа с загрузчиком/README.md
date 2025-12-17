* Включаем отображение меню GRUB
```sh
vim /etc/default/grub
```

* Комментируем строку, скрывающую меню и ставим задержку для выбора пункта меню в 10 секунд.
```sh
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```

* Обновляем конфигурацию загрузчика и перезагружаемся для проверки.

```sh
update-grub
reboot
```

* При загрузке системы видим меню GRUB.
<img width="638" height="479" alt="1" src="https://github.com/user-attachments/assets/0218fc37-6bbe-44ae-9377-8b022f749c30" />


**Попасть в систему без пароля**

***Способ 1. init=/bin/bash***

* Изменим параметры загрузки ядра:
* В конце строки, начинающейся с linux, добавляем "init=/bin/bash", а параметр "ro" меняем на "rw", для монтирования системы в режиме чтения и записи и нажимаем сtrl-x для загрузки в систему.
<img width="634" height="436" alt="2" src="https://github.com/user-attachments/assets/d2997e79-d0ec-4cd6-a0f4-6225f15b7fed" />


***Способ 2. Recovery mode***

* В меню загрузчика (Advanced options…), далее загрузить пункт меню с указанием recovery mode в названии. Получим меню режима восстановления.

<img width="685" height="353" alt="3" src="https://github.com/user-attachments/assets/aa366a08-3d0e-4ab4-b501-82932807426b" />



*  включаем поддержку сети (network) для того, чтобы файловая система перемонтировалась в режим read/write (либо это можно сделать вручную)
* выбираем пункт root и попадаем в консоль с пользователем root
<img width="719" height="400" alt="4" src="https://github.com/user-attachments/assets/0738f1df-6e67-412a-b252-4cadf27e1dec" />


* Установлена система со стандартной разбивкой диска с использованием  LVM.

* Первым делом посмотрим текущее состояние системы (список Volume Group):
```sh
VG        #PV #LV #SN Attr   VSize  VFree

ubuntu-vg   1   1   0 wz--n- 20.01g 10.21g
```

* Переименуем VG:
```sh
vgrename ubuntu-vg ubuntu-otus
Volume group "VolGroup00" successfully renamed to "OtusRoot"
```

* В файле /boot/grub/grub.cfg везде заменяем старое название VG на новое.
* После чего можем перезагружаться
* Успешно грузимся с новым именем Volume Group и проверяем:
```sh
vgs

  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-otus   1   1   0 wz--n- 20.01g 10.21g
```

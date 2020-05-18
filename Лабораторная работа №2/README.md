**Отчет по лабораторной работе №2**
***
* Задание №1

5) Смотрим диски в системе: 2 hhd диска sda(ssd1) и sdb(ssd2).

![disks](images/%D0%BF%D1%83%D0%BD%D0%BA%D1%82%205.png?raw=true)

Затем просматриваем, что RAID-массив проинициализирован корректно: `cat /proc/mdstat`, в этом файле отражается текущее состояние RAID-массива.

![RAID](images/RAID.png?raw=true)

Команда `pvs`: отображает информацию о физическом томе, `lvs`: отображает информацию о логическом томе, `vgs`: отображает информацию о группе физических томов, `mount`: просмотр примонтированых устройств.

В итоге проделанного задания мы создали виртуальную машину с дисками ssd1 и ssd2, настроили LVM(Logical Volume Manager), RAID.

* Задание №2

6) Копируем таблицу разделов со старого диска на новый: `sfdisk -d /dev/sda | sfdisk /dev/sdb`:

![TABLICA](images/TABLICA.png?raw=true)

 Добавляем в RAID-массив новый диск: `mdadm —manage /dev/md0 —add /dev/sdb`:

![RAID2](images/RAID2.png?raw=true)

Cинхронизация разделов, не входящих в RAID, копируя с "живого" диска на новый: `dd if=/dev/sda1 of=/dev/sdb1`:

![noRAID](images/noRAID.png?raw=true)

В итоге проделанного задания мы проэмулировали отказ одного из дисков, удалили диск ssd1, сохранён диск ssd2, добавлен диск ssd3.

* Задание №3

5) Копируем файловую таблицу со старого диска на новый: `sfdisk -d /dev/sda | sfdisk /dev/sdb`:

![TABLICA2](images/TABLICA2.png?raw=true)

При выводе мы замечаем, что в новом диске sdb появились разделы: sdb1, sdb2.

![sdb](images/sdb.png?raw=true)

Устанавливаем загрузчик на новый диск sdb: `grub-install /dev/sdb` - это загрузчик, который загружает нашу операционную систему, и он нам нужен на новом диске после удаления старого.

В итоге у нас установлен новый RAID-массив md63, проверяем при помощи команды `cat /proc/mdstat`:

![md63](images/md63.png?raw=true)

![md632](images/md632.png?raw=true)

6) Настраиваем LVM(Logical Volume Manager)

Создаём новый физический том, включив в него ранее созданный RAID-массив: `pvcreate /dev/md63`

![md633](images/md633.png?raw=true)

Увеличим размер Volume Group system: `vgextend system /dev/sda`

![VG](images/VG.png?raw=true)

LV var,log,root находятся на диске sda:

![LV](images/LV.png?raw=true)

Выполнили перемещение данных со старого диска на новый:

![var](images/var.png?raw=true)

Теперь все данные находятся на одном диске:

![sdb2md63](images/sdb2md63.png?raw=true)

Изменили Volume Group, удалив из него диск старого raid:

Информация о физическом томе: было ![1](images/1.png?raw=true)    стало ![2](images/2.png?raw=true)

Информация о группе физических томов: было ![3](images/3.png?raw=true)    стало ![4](images/4.png?raw=true)

13) Сначала изменили размер второго раздела диска ssd5 (sdb), затем перечитали таблицу разделов и провели результат:

![ssd5](images/ssd5.png?raw=true) 

Добавляем новый диск к текущему RAID-массиву, расширяем количество дисков в нашем массиве до 2-х штук.

14-15) Сначала увеличили размер раздела на диске ssd4 (sda), а затем перечитали таблицу разделов и проверили результат:

![ssd4](images/ssd4.png?raw=true) 

Получили sda2 и sdb2 разделы, которые имеют размер > чем размер RAID-устройство.

16) Расширяем размер RAID 

![R](images/R.png?raw=true) 

Увеличили объём памяти ssd4, ssd5.

18) Завершили миграцию основного массива на новые диски, работа с ssd4 и ssd5 закончена.

![done](images/done.png?raw=true) 

19) Перемещаем `/var/log` на новые диски, для этого создаем новый RAID-массив, создаём логический том:

![var2](images/var2.png?raw=true) 

20) Переносим данные логов со старого раздела на новый:

![log](images/log.png?raw=true) 

21) Правим `/etc/fstab`, поправляем устройство `system-log` на `data-var_log`:

![etc](images/etc.png?raw=true) 

22) Изменяем таблицу разделов командой: `resize2fs`

![resize](images/resize.png?raw=true) 

Завершили установку: 

![final](images/final.png?raw=true) 

Что было сделано: замена дисков на лету, с добавлением новых дисков и переносом разделов.


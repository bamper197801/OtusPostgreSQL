Установка и настройка PostgreSQL

Цель:
создавать дополнительный диск для уже существующей виртуальной машины, размечать его и делать на нем файловую систему
переносить содержимое базы данных PostgreSQL на дополнительный диск
переносить содержимое БД PostgreSQL между виртуальными машинами

создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в ЯО/Virtual Box/докере
поставьте на нее PostgreSQL 15 через sudo apt
## VM сервер ubuntu 24.04
> подключение к виртуальной машине производится через mobaXterm
1. **Автоматизированная настройка репозитория:**\
```sudo apt install -y postgresql-common```\
```sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh```
2. **Устанавливаем нужные модули PostgreSQL 17 и contrib:**\
```sudo apt install postgresql-17```

3. **Запуск сервиса PostgreSQL:**\
```sudo systemctl start postgresql```
4. **Включение сервиса PostgreSQL:**\
```sudo systemctl enable postgresql```

проверьте что кластер запущен через sudo -u postgres pg_lsclusters
```alex@alex:~$ sudo -u postgres pg_lsclusters
[sudo] password for alex:
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
```
зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
postgres=# create table test(c1 text);
postgres=# insert into test values('1');
\q
```postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1

postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | test | table | postgres
(1 row)

postgres=# SELECT * FROM test;
 c1
----
 1
(1 row)
postgres=# \q
```
остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
```
alex@alex:~$ sudo systemctl stop postgresql@17-main
alex@alex:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 down   postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
alex@alex:~$
```
создайте новый диск к ВМ размером 10GB
```
Добавляю диск в VM
```
добавьте свеже-созданный диск к виртуальной машине
Для создания разметки на новом диске вводим команду - sudo fdisk /dev/sdb
```
alex@alex:~$ sudo fdisk /dev/sdb
[sudo] password for alex:

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x5132d348.
```
Далее вводим буку n (обозначает создание нового раздела)
```
Command (m for help): n
```
Затем букву p (создание основного раздела)
Потом системой будет задано три вопроса, везде принимаем настройки по умолчанию, для чего каждый раз на клавиатуре нажимаем клавишу “Enter”
```
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-20971519, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519):

Created a new partition 1 of type 'Linux' and of size 10 GiB.
```
Для сохранения созданных изменений вводим букву w
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
Теперь необходимо отформатировать раздел. Делаем файловую систему ext4, принятую в операционной системе Ubuntu, для чего вводим команду
```
alex@alex:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2621184 4k blocks and 655360 inodes
Filesystem UUID: 98677c9f-f27a-42e7-804d-9cb9302617f9
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```
Далее создаем каталог /mnt/postgresql_data при помощи команды
```
alex@alex:~$ sudo mkdir /mnt/postgresql_data
```
И монтируем (присоединяем к системе) созданный диск в данный каталог командой
```
alex@alex:~$ sudo mount /dev/sdb1 /mnt/postgresql_data
```
Тем не менее, данное подключение будет действительно только до перезагрузки операционной системы. Чтобы создать постоянную точку монтирования для нового диска необходимо внести изменения в конфигурационный файл fstab, для этого в терминале вводим команду
```
alex@alex:~$ sudo nano /etc/fstab

```
В открывшемся файле, в конце дописываем строку, следующего содержания (обратите внимание на пробелы, два последних символа это нули)
```
/dev/sdb1 /mnt/postgresql_data ext4 defaults 0 0
```
перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
```
Диск остался примонтированным после перезагрузки

alex@alex:~$ cd /mnt/postgresql_data/
alex@alex:/mnt/postgresql_data$

```
сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
```
alex@alex:~$ /mnt/postgresql_data - chown -R postgres:postgres /mnt/postgresql_data
-bash: /mnt/postgresql_data: Is a directory
```
перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
```
 mv /var/lib/postgresql/17 /mnt/postgresql_data
```
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
напишите что и почему поменяли
```
кластер не запустилмя т.к. отсуттсвует каталог с данными, который прописан в "/etc/postgresql/17/main/postgresql.conf"
необходимо изменить параметр data_directory в "/etc/postgresql/17/main/postgresql.conf"

data_directory = '/mnt/postgresql_data/17/main' 

```
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
```
Error: Data directory /mnt/postgresql_data/17/main must not be owned by root

```
проверяем права на каталог
```
ls -ld /mnt/postgresql_data/17/

drwxr-xr-x 3 root root 4096 мар 17 13:59 /mnt/postgresql_data/17/

```
повторно назначаю права на присоедененный каталог, проверяю права, смотрю стсатус кластера pg
```

alex@alex:~$ sudo chown -R postgres:postgres /mnt/postgresql_data/
alex@alex:~$ ls -ld /mnt/postgresql_data/17/
drwxr-xr-x 3 postgres postgres 4096 мар 17 13:59 /mnt/postgresql_data/17/
alex@alex:~$ sudo -u postgres pg_ctlcluster 17 main status
pg_ctl: сервер работает (PID: 787)
/usr/lib/postgresql/17/bin/postgres "-D" "/var/lib/postgresql/17/main" "-c" "config_file=/etc/postgresql/17/main/postgresql.conf"
```
перезапускаю кластер - кластер успешно запущен.
```
alex@alex:~$ sudo systemctl stop postgresql@17-main
alex@alex:~$ sudo systemctl start postgresql@17-main
alex@alex:~$ sudo systemctl status postgresql@17-main
● postgresql@17-main.service - PostgreSQL Cluster 17-main
     Loaded: loaded (/usr/lib/systemd/system/postgresql@.service; enabled-runtime; preset: e>
     Active: active (running) since Mon 2025-03-17 14:33:03 UTC; 10min ago
    Process: 1288 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 17-main start (>
   Main PID: 1293 (postgres)
      Tasks: 6 (limit: 2272)
     Memory: 18.8M (peak: 27.2M)
        CPU: 522ms
     CGroup: /system.slice/system-postgresql.slice/postgresql@17-main.service
             ├─1293 /usr/lib/postgresql/17/bin/postgres -D /mnt/postgresql_data/17/main -c c>
             ├─1294 "postgres: 17/main: checkpointer "
             ├─1295 "postgres: 17/main: background writer "
             ├─1297 "postgres: 17/main: walwriter "
             ├─1298 "postgres: 17/main: autovacuum launcher "
             └─1299 "postgres: 17/main: logical replication launcher "

мар 17 14:33:00 alex systemd[1]: Starting postgresql@17-main.service - PostgreSQL Cluster 17>
мар 17 14:33:03 alex systemd[1]: Started postgresql@17-main.service - PostgreSQL Cluster 17->
lines 1-18/18 (END)

```
зайдите через через psql и проверьте содержимое ранее созданной таблицы
```
alex@alex:~$ sudo su - postgres
postgres@alex:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.

postgres=# select * from test;
 c1
----
 1
(1 строка)

```
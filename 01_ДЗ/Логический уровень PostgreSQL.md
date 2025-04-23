## Работа с базами данных, пользователями и правами

Цель:
создание новой базы данных, схемы и таблицы
создание роли для чтения данных из созданной схемы созданной базы данных
создание роли для чтения и записи из созданной схемы созданной базы данных

### создайте новый кластер PostgresSQL 17
```
alex@alex:~$ sudo pg_createcluster -d /var/lib/postgresql/17/main2 17 main2
Creating new PostgreSQL cluster 17/main2 ...
/usr/lib/postgresql/17/bin/initdb -D /var/lib/postgresql/17/main2 --auth-local peer --auth-host scram-sha-256 --no-instructions
Файлы, относящиеся к этой СУБД, будут принадлежать пользователю "postgres".
От его имени также будет запускаться процесс сервера.

Кластер баз данных будет инициализирован с локалью "ru_RU.UTF-8".
Кодировка БД по умолчанию, выбранная в соответствии с настройками: "UTF8".
Выбрана конфигурация текстового поиска по умолчанию "russian".

Контроль целостности страниц данных отключён.

исправление прав для существующего каталога /var/lib/postgresql/17/main2... ок
создание подкаталогов... ок
выбирается реализация динамической разделяемой памяти... posix
выбирается значение "max_connections" по умолчанию... 100
выбирается значение "shared_buffers" по умолчанию... 128MB
выбирается часовой пояс по умолчанию... Etc/UTC
создание конфигурационных файлов... ок
выполняется подготовительный скрипт... ок
выполняется заключительная инициализация... ок
сохранение данных на диске... ок
Ver Cluster Port Status Owner    Data directory               Log file
17  main2   5433 down   postgres /var/lib/postgresql/17/main2 /var/log/postgresql/postgresql-17-main2.log
alex@alex:~$
```
### запуск нового кластера
```
alex@alex:~$ sudo pg_ctlcluster 17 main2 start
alex@alex:~$ sudo pg_ctlcluster 17 main2 status
pg_ctl: сервер работает (PID: 23567)
/usr/lib/postgresql/17/bin/postgres "-D" "/var/lib/postgresql/17/main2" "-c" "config_file=/etc/postgresql/17/main2/postgresql.conf"

```
### зайдите в созданный кластер под пользователем postgres
```
alex@alex:~$ sudo -u postgres psql -p 5433
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.

postgres=#

```
### создайте новую базу данных testdb
```
postgres=# CREATE DATABASE testdb;
CREATE DATABASE
```
### зайдите в созданную базу данных под пользователем postgres
```
postgres=# \c testdb
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=#

```
### создайте новую схему testnm
```
testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA
testdb=#

```
### создайте новую таблицу t1 с одной колонкой c1 типа integer
```
testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE
testdb=#

```
## вставьте строку со значением c1=1
```
testdb=# INSERT INTO t1 values(1);
INSERT 0 1
```
## создайте новую роль readonly
```
testdb=# CREATE role readonly;
CREATE ROLE
```
## дайте новой роли право на подключение к базе данных testdb
```
testdb=# GRANT connect on DATABASE testdb TO readonly;
GRANT
```
## дайте новой роли право на использование схемы testnm
```
testdb=# GRANT usage on SCHEMA testnm to readonly;
GRANT
```
## дайте новой роли право на select для всех таблиц схемы testnm
```
testdb=# GRANT SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
```
## создайте пользователя testread с паролем test123
```
testdb=# CREATE USER testread with password 'test123';
CREATE ROLE
```
## дайте роль readonly пользователю testread
```
testdb=# grant readonly TO testread;
GRANT ROLE
```
## зайдите под пользователем testread в базу данных testdb
```
testdb=# \c testdb testread
Password for user testread:
Вы подключены к базе данных "testdb" как пользователь "testread".
testdb=>

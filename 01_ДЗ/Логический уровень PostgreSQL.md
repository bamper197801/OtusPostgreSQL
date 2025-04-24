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

## сделайте select * from t1;
```
testdb=> SELECT * FROM t1;
ERROR:  permission denied for table t1
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
напишите что именно произошло в тексте домашнего задания
у вас есть идеи почему? ведь права то дали?

*Когда создавали таблицу она создалась в схеме public а не testnm, а пользователю testread мы дали права на select всех таблиц в схеме testnm*
## посмотрите на список таблиц
подсказка в шпаргалке под пунктом 20
```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
```
postgres=# show search_path;
   search_path
-----------------
 "$user", public
(1 row)
```
*Если посмотрим search_path то сначала идет $user, потом public. Так как схемы $user у нас нет, то по умолчанию создается в public, можно поменять эти параметры по умолчанию.*
## вернитесь в базу данных testdb под пользователем postgres
```
postgres=# \c testdb postgres
testdb=#
```
## удалите таблицу t1
```
testdb=# DROP TABLE t1;
DROP TABLE
```
## создайте ее заново но уже с явным указанием имени схемы testnm
```
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
```
## вставьте строку со значением c1=1
```
testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1
testdb=# SELECT * FROM testnm.t1;
 c1
----
  1
(1 row)
```
## зайдите под пользователем testread в базу данных testdb
```
testdb=# \c testdb testread
Password for user testread:
testdb=>
```
## сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
получилось?

*Пишет, что у пользователя нет прав просмотра таблицы t1, но она получается существует в базе*

есть идеи почему? если нет - смотрите шпаргалку

*Потому что мы создали таблицу после раздачи прав на select всех таблиц пользователю testread.*

как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку

*Нужно снова раздать права на чтение всех таблиц в базе пользователю. Нужно использовать ALTER default, который даст права и новым таблицам созданным в базе, а не grant SELECT, который даёт права только на те таблицы, которые на данный момент существуют в базе.*

```
testdb=> \c testdb postgres
Password for user postgres:
testdb=# ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly;
ALTER DEFAULT PRIVILEGES
testdb=# \c testdb testread;
Password for user testread:
testdb=>
```
## сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
получилось?

*А, мы сделали ALTER default, значит надо пересоздать таблицу, она была уже на момент выполнения этой команды, значит для неё надо было повторить grant SELECT или в данный момент просто пересоздать таблицу.*

есть идеи почему? если нет - смотрите шпаргалку

*Пересоздадим таблицу, чтобы сработал ALTER defaul*

```
testdb=> \c testdb postgres
Password for user postgres:
testdb=# DROP TABLE testnm.t1;
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
testdb=# \c testdb testread;
Password for user testread:
testdb=> SELECT * FROM testnm.t1;
 c1
----
(0 rows)

testdb=>
```
## сделайте select * from testnm.t1;
```
testdb=> SELECT * FROM testnm.t1;
 c1
----
(0 rows)

testdb=>
```
получилось?

*Да, все ОК, так как теперь пользователь имеет права на чтение таблицы, ведь мы её пересоздали*

ура!
## теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
```
testdb=> create table t2(c1 integer);
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1
```
а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?

*Выполнив show search_path; мы выяснили что по умолчанию используется "$user", потом public схемы, так как схемы $user у нас нет, то обращаемся к public. Public создается всегда, соответственно при добавлении нового пользователя роль public автоматически ему присваевается, а значит пользователь, которого мы создали имеет право на создание объектов в схеме public в любой базе данных, а ведь мы разрешили данному пользователю доступ на подключение.* 

есть идеи как убрать эти права? если нет - смотрите шпаргалку

*Воспользуемся подсказкой, пункт 36.*

```
postgres=# \c testdb postgres;
testdb=# REVOKE CREATE on SCHEMA public FROM public;
REVOKE
testdb=# REVOKE ALL on DATABASE testdb FROM public;
REVOKE
testdb=# \c testdb testread;
Password for user testread:
testdb=>
```
если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

*Зайдем под пользователем postgres в testdb. REVOKE лишает прав всех пользователей на создание в схеме public. REVOKE отменяет любые привилегии на объекты базы данных для роли. Отзываем права в базе testdb в схеме public.*

## теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
```
postgres=> \c testdb testread;
testdb=> create table t3(c1 integer);
ERROR:  permission denied for schema public
LINE 1: create table t3(c1 integer);
                     ^
testdb=> insert into t2 values (2);
INSERT 0 1
testdb=>
```
расскажите что получилось и почему

*Нет прав на создание таблицы у пользователя, так как ранее мы их отозвали для public. А вот строка в таблицу добавилась? Мы её добавляли в таблицу t2, а её соответвенно создавали в предыдущем шаге данным пользователем в схеме public. Проверим*

```
testdb=> SELECT * FROM t2;
 c1
----
  2
  2
(2 rows)
```
*Значит мы можем добавлять данные в уже созданную таблицу определенным пользователем в схеме паблик, даже если после отозвали для него права.*

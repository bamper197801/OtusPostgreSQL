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
> выключить auto commit

> в первой сессии создаем новую БД, таблицу и наполняем ее данными

```CREATE DATABASE otus;``` -- создание новой БД "otus"\
> postgres=# \c otus -- подключаемся к БД "otus"\

```create table persons(id serial, first_name text, second_name text); ```-- создаем таблицу\
```insert into persons(first_name, second_name) values('ivan', 'ivanov'); ``` -- добавляем в таблицу данные\
```insert into persons(first_name, second_name) values('petr', 'petrov'); commit;```-- добавляем в таблицу данные\

> посмотреть текущий уровень изоляции: show transaction otuslation level

```read committed```

> зайти удаленным ssh (первая сессия), не забывайте про ssh-add

```
alex@alex:~$ sudo su - postgres
[sudo] password for alex:
postgres@alex:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.

```

### зайти вторым ssh (вторая сессия). Запустить везде psql из под пользователя postgres
```

alex@alex::~$ sudo -u postgres
alex@alex::~$ sudo -u postgres psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.
```
*Создание базы данных otus в 1-м подключении*
```
postgres=# CREATE DATABASE otus;
DATABASECREATE DATABASE
postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# SELECT current_database();
 (i serial, amount int) current_database
------------------
 otus
(1 row)
```
*Подключение к созданной базе во втором подключении*
```
alex@alex::~$ sudo -u postgres psql -d otus
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.
```
*Выключаем auto commit*
```
otus=# \echo :AUTOCOMMIT
on
otus=# \set AUTOCOMMIT OFF
```
*Создаем таблицу*
```
otus=# create table persons(id serial, first_name text, second_name text);
CREATE TABLE
otus=# insert into persons(first_name, second_name) values('ivan', 'ivanov');
INSERT 0 1
otus=# insert into persons(first_name, second_name) values('petr', 'petrov');
INSERT 0 1
otus=# commit;
WARNING:  there is no transaction in progress
COMMIT
otus=# SELECT * FROM persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
*Посмотреть текущий уровень изоляции: show transaction otuslation level*
```
otus=# show transaction otuslation level;
 transaction_otuslation
-----------------------
 read committed
(1 row)
```
*в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');*
```
otus=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
*сделать select from persons во второй сессии*
```
otus=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
*видите ли вы новую запись и если да то почему?  
Новой записи нет, так как auto commit отключен, а commit в первой сессии мы не сделали, поэтому запись во второй транзакции не видна, а текущий уровень транзакции у нас read committed.  
завершить первую транзакцию - commit;*
```
otus=*# commit;
COMMIT
```
*сделать select from persons во второй сессии*
```
otus=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
*видите ли вы новую запись и если да то почему?  
В первой сессии сделали commit и во второй появилась строка  
завершите транзакцию во второй сессии*
```
otus=# commit;
WARNING:  there is no transaction in progress
COMMIT
```
*начать новые но уже repeatable read транзации - set transaction otuslation level repeatable read;*
```
otus=# BEGIN TRANSACTION otusLATION LEVEL REPEATABLE READ;
BEGIN
```
*в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');*
```
otus=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```
*сделать select* from persons во второй сессии*
```
otus=*# select* from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | darya      | durandina
  6 | denis      | durandin
(5 rows)
```

видите ли вы новую запись и если да то почему?  
Новую запись не вижу во второй транзакции так как commit в первой еще не сделан, а исходя из нового уровня транзакции нужно будет сделать commit и в первом и во втором подключении  
завершить первую транзакцию - commit;*
```
otus=*# commit;
COMMIT
```
*сделать select from persons во второй сессии*
```
otus=*# select* from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | darya      | durandina
  6 | denis      | durandin
(5 rows)
```
*видите ли вы новую запись и если да то почему?  
Новой записи во втором подключении нет, так как уровень изоляции предполагает сделать commit и в первом и во втором подключении  
завершить вторую транзакцию*
```
otus=*# commit;
COMMIT
otus=# select* from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | darya      | durandina
  6 | denis      | durandin
  7 | sveta      | svetova
(6 rows)
```
*сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему? 
После выполненных действий вижу новую строку, так как commit выполнен и в первом и во втором подключении, уровень транзации REPEATABLE READ*
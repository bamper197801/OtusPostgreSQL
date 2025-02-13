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

> посмотреть текущий уровень изоляции: show transaction isolation level

```read committed```

> Новая сессия

```
alex@alex:~$ sudo su - postgres
[sudo] password for alex:
postgres@alex:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.

postgres=# \c otus
Вы подключены к базе данных "otus" как пользователь "postgres".
otus=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
otus=#
```

> Вторая сессия

```
alex@alex:~$ sudo su - postgres
[sudo] password for alex:
postgres@alex:~$ psql
psql (17.2 (Ubuntu 17.2-1.pgdg24.04+1))
Введите "help", чтобы получить справку.

postgres=# \c otus
Вы подключены к базе данных "otus" как пользователь "postgres".

otus=# SELECT id, first_name, second_name FROM public.persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)

otus=#
```

суть всех операций состоит в том что бы проверить как работает свойство auto commit
при включенном по умолчании auto commit во всех сессиях результат выборки данных будет одинаков
при выключенном auto commit резутат оновления данных бкдет только в сесси в котрой было произведеено действие, другие сесси изенений не увидят пока не применят commit; в первой сессии.
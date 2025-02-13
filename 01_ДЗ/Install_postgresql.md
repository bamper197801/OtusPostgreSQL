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

> в первой сессии создаем новую БД, таблицу и наполняем ее данными

```CREATE DATABASE otus;``` -- создание новой БД "otus"\
> postgres=# \c otus -- подключаемся к БД "otus"\

```create table persons(id serial, first_name text, second_name text); ```-- создаем таблицу\
```insert into persons(first_name, second_name) values('ivan', 'ivanov'); ``` -- добавляем в таблицу данные\
```insert into persons(first_name, second_name) values('petr', 'petrov'); commit;```-- добавляем в таблицу данные


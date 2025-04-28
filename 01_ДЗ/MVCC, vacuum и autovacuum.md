# MVCC, vacuum и autovacuum.
## Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
## Установить на него PostgreSQL 15 с дефолтными настройками
## Создать БД для тестов: выполнить pgbench -i postgres
## Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```
postgres@otus-vm:/home/otus$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 581.5 tps, lat 13.706 ms stddev 8.245, 0 failed
progress: 12.0 s, 475.7 tps, lat 16.804 ms stddev 14.859, 0 failed
progress: 18.0 s, 632.7 tps, lat 12.656 ms stddev 6.698, 0 failed
progress: 24.0 s, 626.8 tps, lat 12.756 ms stddev 7.055, 0 failed
progress: 30.0 s, 683.5 tps, lat 11.714 ms stddev 5.966, 0 failed
progress: 36.0 s, 631.2 tps, lat 12.667 ms stddev 6.795, 0 failed
progress: 42.0 s, 446.7 tps, lat 17.914 ms stddev 14.114, 0 failed
progress: 48.0 s, 593.5 tps, lat 13.481 ms stddev 7.299, 0 failed
progress: 54.0 s, 564.2 tps, lat 14.179 ms stddev 15.257, 0 failed
progress: 60.0 s, 651.0 tps, lat 12.281 ms stddev 6.306, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 35328
number of failed transactions: 0 (0.000%)
latency average = 13.586 ms
latency stddev = 9.702 ms
initial connection time = 14.226 ms
tps = 588.710683 (without initial connection time)
```
## Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
```
sudo nano /etc/postgresql/15/main/postgresql.conf
```
*Применяем параметры настройки*

> max_connections = 40 + (Определяет максимальное число одновременных подключений к серверу.)
>
> shared_buffers = 1GB + (25% от общего объема оперативной памяти компьютера) - увеличили
>
> effective_cache_size = 3GB + (Этот параметр влияет на планировщик запросов, а не ограничивает дисковый кэш. Чем выше, тем больше вероятность, что будет применяться сканирование по индексу. Чем ниже, тем более вероятно, что будет выбрано последовательное сканирование.) - уменьшили
>
> maintenance_work_mem = 512MB + (задаёт максимальный объём памяти для операций обслуживания) - увеличили
>
> checkpoint_completion_target = 0.9 + (доля времени между контрольными точками для завершения контрольной точки)
>
> wal_buffers = 16MB + (если у вас много одновременных подключений, то более высокое значение может повысить производительность)
>
> default_statistics_target = 500 + (число записей просматриваемых при сборе статистики по таблицам)
>
> random_page_cost = 4 + (Сам сервер Postgres не знает, какая у нас дисковая подсистема. И с помощью этого параметра мы ему об этом сообщаем.)
>
> effective_io_concurrency = 2 + (допустимое число параллельных операций ввода/вывода)
>
> work_mem = 6553kB + (Максимальный лимит памяти, который выделяется для обработки запросов)
>
> min_wal_size = 4GB + (Этот параметр ограничивает размер WAL снизу)
>
> max_wal_size = 16GB + (Максимальный размер, до которого может вырастать WAL)

## Протестировать заново
```
postgres@otus-vm:/home/otus$ pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 653.7 tps, lat 12.197 ms stddev 7.894, 0 failed
progress: 12.0 s, 664.0 tps, lat 12.044 ms stddev 7.007, 0 failed
progress: 18.0 s, 681.8 tps, lat 11.736 ms stddev 6.261, 0 failed
progress: 24.0 s, 499.0 tps, lat 16.012 ms stddev 13.443, 0 failed
progress: 30.0 s, 656.5 tps, lat 12.200 ms stddev 6.820, 0 failed
progress: 36.0 s, 594.7 tps, lat 13.451 ms stddev 7.585, 0 failed
progress: 42.0 s, 606.8 tps, lat 13.183 ms stddev 7.699, 0 failed
progress: 48.0 s, 593.7 tps, lat 13.471 ms stddev 7.444, 0 failed
progress: 54.0 s, 467.2 tps, lat 17.127 ms stddev 15.379, 0 failed
progress: 60.0 s, 667.5 tps, lat 11.977 ms stddev 6.262, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 36517
number of failed transactions: 0 (0.000%)
latency average = 13.142 ms
latency stddev = 8.848 ms
initial connection time = 14.232 ms
tps = 608.600884 (without initial connection time)
```
Что изменилось и почему?
*На старте: За 1 минуту работы с 8-ю одновременными соединениями кластер обработал 35328 транзакций с пропускной способностью примерно 589 транзакций в секунду*

*После применения настроек: За 1 минуту работы с 8-ю одновременными соединениями кластер обработал 36517 транзакций с пропускной способностью примерно 609 транзакций в секунду.*

*Мы смогли увеличить пропускную способность базы данных с 589 TPS до 609 TPS при 8 одновременных соединениях, это примерно на 3%. т.к. в настройках мы увеличили параметры wal_buffers, shared_buffers, maintenance_work_mem и таким образом повысили производительность.*

## Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
```
postgres=# create table test5(test text);
CREATE TABLE
postgres=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
 public | test5            | table | postgres
(5 rows)
postgres=# INSERT INTO test5(test) SELECT 'noname' FROM generate_series(1,1000000);
INSERT 0 1000000
```
## Посмотреть размер файла с таблицей
```
postgres=# SELECT pg_size_pretty(pg_TABLE_size('test5'));
 pg_size_pretty
----------------
 35 MB
(1 row)
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs  WHERE relname = 'test5';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test5   |    1000000 |          0 |      0 | 2024-03-08 01:37:09.983705+00
(1 row)
```
## 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
```
## Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs  WHERE relname = 'test5';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test5   |     996313 |    1001574 |    100 | 2024-03-08 01:48:22.367829+00
(1 row)

```
## Подождать некоторое время, проверяя, пришел ли автовакуум
```
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs  WHERE relname = 'test5';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test5   |    1642715 |          0 |      0 | 2024-03-08 01:49:10.540125+00
(1 row)
postgres=# SELECT pg_size_pretty(pg_TABLE_size('test5'));
 pg_size_pretty
----------------
 238 MB
(1 row)
```
## 5 раз обновить все строчки и добавить к каждой строчке любой символ
```
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
```
## Посмотреть размер файла с таблицей
```
postgres=# SELECT pg_size_pretty(pg_TABLE_size('test5'));
 pg_size_pretty
----------------
 238 MB
(1 row)
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs  WHERE relname = 'test5';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test5   |    1001794 |          0 |      0 | 2024-03-08 01:54:12.017963+00
(1 row)
```
## Отключить Автовакуум на конкретной таблице
```
postgres=# ALTER TABLE test5 SET (autovacuum_enabled = off);
ALTER TABLE
```
## 10 раз обновить все строчки и добавить к каждой строчке любой символ
```
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
postgres=# update test5 set test = CONCAT(test, 'd');
UPDATE 1000000
```
## Посмотреть размер файла с таблицей
```
postgres=# SELECT pg_size_pretty(pg_TABLE_size('test5'));
 pg_size_pretty
----------------
 570 MB
(1 row)

```
Объясните полученный результат
*Автовакуум отключен, при каждом update он создаёт столько же новых строк и таблица раздувается*

Не забудьте включить автовакуум)

create table tests (id bigserial, value text);

INSERT INTO tests (value)
SELECT substr(
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text),
    1,
    (floor(random() * (500 - 30 + 1)) + 30)::integer
)
FROM generate_series(1, 1000000);

CREATE INDEX tests_value_btree ON tests USING btree (value);
CREATE INDEX

explain analyze select * from tests order by value limit 10;
                                                                 QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.68..4.03 rows=10 width=277) (actual time=0.034..0.072 rows=10 loops=1)
   ->  Index Scan using tests_value_btree on tests  (cost=0.68..335884.61 rows=1000000 width=277) (actual time=0.032..0.069 rows=10 loops=1)
 Planning Time: 0.532 ms
 Execution Time: 0.085 ms
(4 строки)

\di+ tests_value_btree 
                                             Список отношений
 Схема  |        Имя        |  Тип   | Владелец | Таблица |  Хранение  | Метод доступа | Размер | Описание
--------+-------------------+--------+----------+---------+------------+---------------+--------+----------
 public | tests_value_btree | индекс | sterx    | tests   | постоянное | btree         | 330 MB |
(1 строка)


drop index tests_value_btree;

CREATE INDEX tests_value_hash ON tests USING hash (value);

\di+ tests_value_hash
                                             Список отношений
 Схема  |       Имя        |  Тип   | Владелец | Таблица |  Хранение  | Метод доступа | Размер | Описание
--------+------------------+--------+----------+---------+------------+---------------+--------+----------
 public | tests_value_hash | индекс | sterx    | tests   | постоянное | hash          | 32 MB  |
(1 строка)

explain analyze select * from tests order by value limit 10;

---

create table tests (id bigserial, value text);

INSERT INTO tests (value)
SELECT substr(
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text) ||
    md5(random()::text) || md5(random()::text) || md5(random()::text) || md5(random()::text),
    1,
    (floor(random() * (25 - 5 + 1)) + 5)::integer
)
FROM generate_series(1, 1000000);

CREATE INDEX tests_value_btree ON tests USING btree (value);

\di+ tests_value_btree 
                                             Список отношений
 Схема  |        Имя        |  Тип   | Владелец | Таблица |  Хранение  | Метод доступа | Размер | Описание
--------+-------------------+--------+----------+---------+------------+---------------+--------+----------
 public | tests_value_btree | индекс | sterx    | tests   | постоянное | btree         | 34 MB  |
(1 строка)

CREATE INDEX tests_value_hash ON tests USING hash (value);

\di+ tests_value_hash

CREATE INDEX
                                             Список отношений
 Схема  |       Имя        |  Тип   | Владелец | Таблица |  Хранение  | Метод доступа | Размер | Описание
--------+------------------+--------+----------+---------+------------+---------------+--------+----------
 public | tests_value_hash | индекс | sterx    | tests   | постоянное | hash          | 32 MB  |
(1 строка)




CREATE INDEX tests_value_hash ON tests USING hash (value, id);

Из [[Книга «PostgreSQL 16 изнутри», Рогов Е. В.]] причина некорректного плана запроса в `EXPLAIN`:

1. Неправильная статистика
2. Неправильная оценка селективности запроса

Из [[Видео «Производительность запросов в PostgreSQL. Илья Космодемьянский (PostgreSQL Consulting)»]].

cost — время необходимое для извлечения одного блока 8 килобайт (страница) при Seq Scan. cost 9.54 это значит, что в 9.54 раза эта операция медленнее, чем достать одну страницу 8 килобайтную.

Нижний узел Explain с большой стоимостью надо оптимизировать.

1. Длинный запрос от клиента (ORM генерирует плохой запрос)
2. Запрос возвращает много данных — 10Gb данных быстро не вернётся никуда
3. Много данных гонять по сети — тоже быстро не будет
4. Собрана ли статистика? По [[Книга «PostgreSQL 16 изнутри», Рогов Е. В.]] там указывалось в тч как статистику собирать по функции от колонки и иначе настраивать статистику
5. Ускорит ли индекс исполнение запроса? Попробовать можно. Селективность высокая должна быть (если выбирается много строк — селективность, то есть избирательность маленькая, и индекс не будет использован).
	1. Есть сессионные переменные (можно для теста отключить возможность использования индекса или Seq Scan или какого-то механизма соединения, see [docs](https://www.postgresql.org/docs/current/runtime-config-query.html)):
		1. `enable_indexscan` 
		2. `enable_seqscan`
		3. `enable_hashjoin`
		4. `enable_mergejoin`
6. иногда `work_mem` надо увеличить, чтобы hash join работал, чтобы хешированная таблица могла уместиться в памяти

Запросы, с которыми ничего нельзя сделать:

1. `count(*)`. Можно использовать приблизительный count — из `pg_catalog` можно достать количество строк в таблице на момент последнего выполнения `ANALYZE`, и это очень быстрый запрос. Можно написать хранимую процедуру и дать права на неё пользователю БД с `SECURITY DEFINER` [docs](https://postgrespro.ru/docs/postgresql/16/sql-createfunction), 33:45 в [[Видео «Производительность запросов в PostgreSQL. Илья Космодемьянский (PostgreSQL Consulting)»|видео]]
2. join 300 таблиц (300! вариантов сделать этот join)
3. клиенту возвращается млн строк (юзеру в большинстве случаев столько не надо. Если это выгрузка — ее можно сделать ночью или из дампа)


[[PostgreSQL]]

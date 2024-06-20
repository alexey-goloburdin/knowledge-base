Чтобы нормально `ORDER BY`  работал и прочее надо создавать БД с правильной кодировкой. Неважно при этом как создан кластер.

```sql
CREATE ROLE rroom WITH LOGIN PASSWORD 'mycoolp@ssword';

create database rroom_db
	with
	template=template0
	encoding='UTF8'
	lc_collate='ru_RU.UTF-8'
	lc_ctype='ru_RU.UTF-8'
	owner rroom;
```

[[PostgreSQL]]
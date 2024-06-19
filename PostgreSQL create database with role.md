```sql
create role rroom with login password 'mycoolp@ssword';

create database rroom_db with template=template0 encoding='UTF8' lc_collate='ru_RU.UTF-8' lc_ctype='ru_RU.UTF-8' owner rroom;
```

[[PostgreSQL]]
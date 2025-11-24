

```sql
DO
$$
DECLARE
    r RECORD;
BEGIN
    -- Перебираем все таблицы во всех схемах, кроме системных
    FOR r IN
        SELECT tablename, schemaname
        FROM pg_tables
        WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
    LOOP
        EXECUTE format('DROP TABLE IF EXISTS %I.%I CASCADE', r.schemaname, r.tablename);
    END LOOP;
END
$$;
```

при вставке длинного SQL-скрипта, чтобы он падал при первой ошибке:

```shell
psql -h localhost -U sterx -d birds -v ON_ERROR_STOP=1 -f sql.sql
```

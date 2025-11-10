

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
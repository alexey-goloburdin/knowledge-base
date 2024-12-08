### Увеличение размера сегмента WAL

На диске WAL-журнал хранится в виде файлов в каталоге `$PGDATA/pg_wal`. Каждый файл по умолчанию занимает 16 Мб. Размер можно увеличить, чтобы избежать большого числа файлов в одном каталоге.

Если в базу ведётся активная запись, имеет смысл увеличить размер сегментов WAL (дефолтное значение 16MB):

```bash
initdb -D /pgdata --wal-segsize=32
```

Из [[Книга «PostgreSQL 11. Мастерство разработки», Ганс-Юрген Шёниг]].

В Debian возможно надо `pg_createcluster` использовать вместо `initdb`.

If you want to change the WAL segment size of an existing cluster, you can use `pg_resetwal`. Warning: for that, run `pg_resetwal` only on a cluster that has been shut down cleanly. Running `pg_resetwal` on a crashed cluster will cause potential data loss.

```bash
/usr/lib/postgresql/13/bin/pg_resetwal -D /var/lib/postgresql/13/main --wal-segsize 64
```

You may need to increase `min_wal_size` if you increase the WAL segment size.

[[PostgreSQL]]
Чтобы нормально `ORDER BY`  работал и прочее надо создавать БД с правильной кодировкой. Неважно при этом как создан кластер.

```bash
sudo passwd postgres
su - postgres
psql
```

Создаём роль и БД:

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

Если есть проблемы с локалями:

```bash
sudo apt-get install -y locales
sudo locale-gen ru_RU.UTF-8
sudo dpkg-reconfigure locales
```

И, возможно, надо перезагрузиться (на WSL `sudo reboot` был нужен).

[[PostgreSQL]]
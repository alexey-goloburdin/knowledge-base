1. В настройках виртуальной машины Linux в UTM настраиваем сеть как Shared Network (если это ВМ)
2. Нам необходимо настроить PostgreSQL для доступа по сети. Для этого входим в `psql` на машине с PostgreSQL и смотрим расположение конфигурационных файлов:
	- `SHOW hba_file;`
	- `SHOW hba_file;`
	- `SHOW data_directory;`
3. Отлично, затем редактируем первый конфиг:
	- `sudo vim /etc/postgresql/16/main/postgresql.conf`
		- `listen_addresses = '*'`
4. Затем редактируем второй конфиг (разрешаем пользователю rroom коннект с любых IP — актуально только для dev-среды):
	- `sudo vim /etc/postgresql/16/main/pg_hba.conf`
		- `host    all             rroom          0.0.0.0/0            md5`
5. Рестартим сервер PostgreSQL:
	- `sudo service postgresql restart`
6. Смотрим айпишник сервера:
	- `ip a`

[[PostgreSQL]]
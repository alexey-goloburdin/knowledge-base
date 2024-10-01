Как хранит данные Postgres и как использовать это знание для увеличения производительности запросов?

Один к одному — фигачьте всё в 1 таблицу, как говорят некоторые специалисты, в том числе, например, Владимир Хориков, автор отличной книги о юнит-тестировании и автор блога https://enterprisecraftsmanship.com/. Он [пишет](https://enterprisecraftsmanship.com/posts/modeling-relationships-in-ddd-way/) в статье о моделировании отношений в DDD следующее: «Такие отношения совершенно не нужны. Вы не получите от них никакой пользы. Единственное, что делают отношения «один к одному», - это загромождают вашу базу данных. Если две таблицы соотносятся друг с другом как один к одному, объедините их в одну таблицу».

Так ли это? Попробуем разобраться в этом видео, прям вот потыкаем, проведём эксперименты, разберёмся, как работает база данных изнутри, как она хранит данные и достаёт их и может ли связь один к одному быть полезной. В видео есть таймкоды, можете прыгать на интересующую вас часть

## Настройка

Чистый Debian 12. Ставим актуальный вышедший недавно PostgreSQL 17.

Берём тачку на Selectel, параметры: 2 vCPU, 4 GB RAM, 30 GB HDD

```bash
# as root on Selectel
adduser www
adduser www sudo
su - www

# as www

sudo apt update
sudo apt install -y git gcc vim zip unzip zsh
# oh my zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# install PostgreSQL 16
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt -y install postgresql

# srart
sudo systemctl start postgresql

# set postgres user password
sudo passwd postgres
```

## Создание таблиц

```bash
su - postgres
psql
```

```sql
-- from postgres user
create role sterx with login password 'mycoolp@ssword';

create database wide_tables
	with
	template=template0
	encoding='UTF8'
	lc_collate='ru_RU.UTF-8'
	lc_ctype='ru_RU.UTF-8'
	owner sterx;

create database norm_tables
	with
	template=template0
	encoding='UTF8'
	lc_collate='ru_RU.UTF-8'
	lc_ctype='ru_RU.UTF-8'
	owner sterx;
```

Позволим входить пользователю `sterx`, входим в БД для широких таблиц:

```bash
exit
sudo sudo vim /etc/postgresql/17/main/pg_hba.conf
# add to file
host    all             rroom          0.0.0.0/0            md5

sudo systemctl restart postgresql
sudo apt install -y pgcli
pgcli -h localhost -U sterx -d wide_tables
```

```sql
create table employee (
	employee_id bigint generated always as identity primary key,
	first_name text,
	last_name text,
	phone text,
	email text,
	address text,
	office_id bigint,
	department_id text,
	employment_date date,
	grade smallint,
	referral_id bigint,
	salary bigint,
	photo_url text,
	notes text,
	corporate_money int,
	some_bullshit_01 text, some_bullshit_02 text, some_bullshit_03 text,
	some_bullshit_04 text, some_bullshit_05 text, some_bullshit_06 text,
	some_bullshit_07 text, some_bullshit_08 text, some_bullshit_09 text,
	some_bullshit_10 text, some_bullshit_11 text, some_bullshit_12 text,
	some_bullshit_13 text, some_bullshit_14 text, some_bullshit_15 text,
	some_bullshit_16 text, some_bullshit_17 text, some_bullshit_18 text,
	some_bullshit_19 text, some_bullshit_20 text, some_bullshit_21 text,
	some_bullshit_22 text, some_bullshit_23 text, some_bullshit_24 text,
	some_bullshit_25 text, some_bullshit_26 text, some_bullshit_27 text,
	some_bullshit_28 text, some_bullshit_29 text, some_bullshit_30 text,
	some_bullshit_31 text, some_bullshit_32 text, some_bullshit_33 text,
	some_bullshit_34 text, some_bullshit_35 text, some_bullshit_36 text,
	some_bullshit_37 text, some_bullshit_38 text, some_bullshit_39 text
);

INSERT INTO employee (
    first_name, last_name, phone, email, address, office_id, department_id,
    employment_date, grade, referral_id, salary, photo_url, notes, corporate_money,
    some_bullshit_01, some_bullshit_02, some_bullshit_03, some_bullshit_04,
    some_bullshit_05, some_bullshit_06, some_bullshit_07, some_bullshit_08,
    some_bullshit_09, some_bullshit_10, some_bullshit_11, some_bullshit_12,
    some_bullshit_13, some_bullshit_14, some_bullshit_15, some_bullshit_16,
    some_bullshit_17, some_bullshit_18, some_bullshit_19, some_bullshit_20,
    some_bullshit_21, some_bullshit_22, some_bullshit_23, some_bullshit_24,
    some_bullshit_25, some_bullshit_26, some_bullshit_27, some_bullshit_28,
    some_bullshit_29, some_bullshit_30, some_bullshit_31, some_bullshit_32,
    some_bullshit_33, some_bullshit_34, some_bullshit_35, some_bullshit_36,
    some_bullshit_37, some_bullshit_38, some_bullshit_39
)
SELECT 
    -- Случайное имя
    substr(md5(random()::text), 1, 8),
    -- Случайная фамилия
    substr(md5(random()::text), 1, 10),
    -- Случайный номер телефона
    format('(%s) %s-%s', (random() * 900 + 100)::int, (random() * 900 + 100)::int, (random() * 9000 + 1000)::int),
    -- Случайный email
    substr(md5(random()::text), 1, 6) || '@example.com',
    -- Случайный адрес
    format('%s %s St', (random() * 999)::int, substr(md5(random()::text), 1, 6)),
    -- Случайный идентификатор офиса
    (random() * 10 + 1)::int,
    -- Случайный отдел
    CASE WHEN (random() < 0.33) THEN 'HR'
         WHEN (random() < 0.66) THEN 'IT'
         ELSE 'Finance' END,
    -- Случайная дата найма
    '2015-01-01'::date + (random() * 2000)::int,
    -- Случайная оценка (grade)
    (random() * 10 + 1)::int,
    -- Случайный идентификатор рекомендателя
    (random() * 500 + 100)::int,
    -- Случайная зарплата
    (random() * 100000 + 40000)::int,
    -- Случайный URL фотографии
    'http://example.com/photo' || (random() * 10000)::int || '.jpg',
    -- Случайные заметки
    CASE WHEN random() < 0.5 THEN repeat('Hardworking', 50) ELSE repeat('Punctual', 50) END,
    -- Случайное значение корпоративного счёта
    (random() * 5000)::int,
    -- some_bullshit fields
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10)
FROM generate_series(1, 1000000) AS gs; -- миллион записей

VACUUM FULL ANALYZE employee;
```

Ребутаем сервер, чтобы очистить буфер:

```bash
sudo systemctl restart postgresql
pgcli -h localhost -U sterx -d wide_tables
```

```sql
explain (analyze, buffers)
select employee_id, first_name, last_name
from employee
where department_id='HR' order by employment_date limit 20;
-- Buffers: shared hit=306 read=142666
-- Execution Time: 234.087 ms
```

Здесь мы видим buffers shared hit — это прочитанные из буферного кеша страницы, а read это прочитаные из файловой системы страницы. Как видим, было прочитано почти 85 тысяч страниц с диска.

Теперь смотрим на нормализованные узкие таблицы:

```bash
pgcli -h localhost -U sterx -d norm_tables
```

```sql
create table employee (
    employee_id bigint generated always as identity primary key,
    first_name text,
    last_name text
);
create table employee_contact (
    employee_id bigint primary key references employee(employee_id),
    phone text,
    email text,
    address text
);
create table employee_hierarchy (
    employee_id bigint primary key references employee(employee_id),
    office_id bigint,
    department_id text,
    employment_date date,
    referral_id bigint,
    grade smallint
);
create table employee_accounting (
    employee_id bigint primary key references employee(employee_id),
    salary bigint,
    photo_url text,
    notes text,
    corporate_money int
);

create table employee_some_bullshit (
    employee_id bigint primary key references employee(employee_id),
	some_bullshit_01 text, some_bullshit_02 text, some_bullshit_03 text,
	some_bullshit_04 text, some_bullshit_05 text, some_bullshit_06 text,
	some_bullshit_07 text, some_bullshit_08 text, some_bullshit_09 text,
	some_bullshit_10 text, some_bullshit_11 text, some_bullshit_12 text,
	some_bullshit_13 text, some_bullshit_14 text, some_bullshit_15 text,
	some_bullshit_16 text, some_bullshit_17 text, some_bullshit_18 text,
	some_bullshit_19 text, some_bullshit_20 text, some_bullshit_21 text,
	some_bullshit_22 text, some_bullshit_23 text, some_bullshit_24 text,
	some_bullshit_25 text, some_bullshit_26 text, some_bullshit_27 text,
	some_bullshit_28 text, some_bullshit_29 text, some_bullshit_30 text,
	some_bullshit_31 text, some_bullshit_32 text, some_bullshit_33 text,
	some_bullshit_34 text, some_bullshit_35 text, some_bullshit_36 text,
	some_bullshit_37 text, some_bullshit_38 text, some_bullshit_39 text
);

INSERT INTO employee (first_name, last_name)
SELECT 
    -- Случайное имя
    substr(md5(random()::text), 1, 8),
    -- Случайная фамилия
    substr(md5(random()::text), 1, 10)
FROM generate_series(1, 1000000) AS gs;

-- Вставка данных в таблицу employee_contact
INSERT INTO employee_contact (employee_id, phone, email, address)
SELECT 
    gs,
    -- Случайный номер телефона
    format('(%s) %s-%s', (random() * 900 + 100)::int, (random() * 900 + 100)::int, (random() * 9000 + 1000)::int),
    -- Случайный email
    substr(md5(random()::text), 1, 6) || '@example.com',
    -- Случайный адрес
    format('%s %s St', (random() * 999)::int, substr(md5(random()::text), 1, 6))
FROM generate_series(1, 1000000) AS gs;

INSERT INTO employee_hierarchy (employee_id, office_id, department_id, employment_date, referral_id, grade)
SELECT 
    gs,
    -- Случайный идентификатор офиса
    (random() * 10 + 1)::int,
    -- Случайный отдел
    CASE WHEN (random() < 0.33) THEN 'HR'
         WHEN (random() < 0.66) THEN 'IT'
         ELSE 'Finance' END,
    -- Случайная дата найма
    '2015-01-01'::date + (random() * 2000)::int,
    -- Случайный идентификатор рекомендателя
    (random() * 500 + 100)::int,
    -- Случайная оценка (grade)
    (random() * 10 + 1)::int
FROM generate_series(1, 1000000) AS gs;

INSERT INTO employee_accounting (employee_id, salary, photo_url, notes, corporate_money)
SELECT 
    gs,
    -- Случайная зарплата
    (random() * 100000 + 40000)::int,
    -- Случайный URL фотографии
    'http://example.com/photo' || (random() * 10000)::int || '.jpg',
    -- Случайные заметки
    CASE WHEN random() < 0.5 THEN repeat('Hardworking', 50) ELSE repeat('Punctual', 50) END,
    -- Случайное корпоративное финансирование
    (random() * 5000)::int
FROM generate_series(1, 1000000) AS gs;

INSERT INTO employee_some_bullshit (employee_id,
	some_bullshit_01, some_bullshit_02, some_bullshit_03, some_bullshit_04,
    some_bullshit_05, some_bullshit_06, some_bullshit_07, some_bullshit_08,
    some_bullshit_09, some_bullshit_10, some_bullshit_11, some_bullshit_12,
    some_bullshit_13, some_bullshit_14, some_bullshit_15, some_bullshit_16,
    some_bullshit_17, some_bullshit_18, some_bullshit_19, some_bullshit_20,
    some_bullshit_21, some_bullshit_22, some_bullshit_23, some_bullshit_24,
    some_bullshit_25, some_bullshit_26, some_bullshit_27, some_bullshit_28,
    some_bullshit_29, some_bullshit_30, some_bullshit_31, some_bullshit_32,
    some_bullshit_33, some_bullshit_34, some_bullshit_35, some_bullshit_36,
    some_bullshit_37, some_bullshit_38, some_bullshit_39)
SELECT 
    gs,
    -- some_bullshit fields
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10), substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10),	substr(md5(random()::text), 1, 10),
	substr(md5(random()::text), 1, 10)
FROM generate_series(1, 1000000) AS gs;

VACUUM FULL ANALYZE employee;
VACUUM FULL ANALYZE employee_contact;
VACUUM FULL ANALYZE employee_hierarchy;
VACUUM FULL ANALYZE employee_accounting;
VACUUM FULL ANALYZE employee_some_bullshit;

EXPLAIN (analyze, buffers)
SELECT e.employee_id, e.first_name, e.last_name
FROM employee e
JOIN employee_hierarchy eh ON e.employee_id = eh.employee_id
WHERE department_id='HR' order by employment_date limit 20;
-- Buffers: shared hit=472 read=8304, temp read=1054 written=1057
-- Execution Time: 94.844 ms
```

Тут temp read и written говорит о том, что использовались файлы для кеширования — потому что серверу не хватило рабочей памяти. Увеличим work_mem, по умолчанию 4 мегабайта и это мало.

Ребутнем сервер, чтобы очистить буфер:

```bash
sudo systemctl restart postgresql
pgcli -h localhost -U sterx -d norm_tables
```

И смотрим новый результат:

```sql
show work_mem;
set work_mem='128MB';

EXPLAIN (analyze, buffers)
SELECT e.employee_id, e.first_name, e.last_name
FROM employee e
JOIN employee_hierarchy eh ON e.employee_id = eh.employee_id
WHERE department_id='HR' order by employment_date limit 20;
-- Buffers: shared hit=149 read=8627
-- Execution Time: 87.218 ms
```

Получаем 87мс против 155мс время выполнения запроса и считанных страниц 9 тысяч против 80 тысяч. Неплохой результат!

Всегда стоит помнить о этом, проектируя свои базы данных!


## Как работает Postgres

Где лежат все файлы баз данных? Они лежат в хранилище, которое называют кластером баз данных. Документация гласит, что 

> Кластер баз данных представляет собой набор баз, управляемых одним экземпляром работающего сервера.

То есть в постгресе кластер это не набор серверов с репликацией и так далее. Когда вы на один компьютер устанавливаете Postgres, то в процессе установки автоматически или вручную происходит инициализация кластера, то есть инициализация этой директории, в которой будут храниться данные всех баз данных.

Ну так вот, место на диске, где хранятся эти файлы, можно посмотреть в параметре 

```sql
SHOW data_directory;
-- /var/lib/postgresql/16/main
```

На выполнение этого запроса необходимы права, от обычного пользователя он не сработает, поэтому можно запустить от пользователя базы данных `postgres`.

В этой директории лежат данные кластера, то есть всех баз данных, входящих в кластер:

```bash
sudo ls -l /var/lib/postgresql/16/main
```

Как посмотреть, где лежат данные конкретной БД?

Данные БД могут лежать в разных табличных пространствах, то есть разных директориях, а как мы помним в linux мы можем примонтировать в директорию целый диск. То есть одни таблицы могут быть на медленных больших дисках, а другие таблицы могут быть на быстрых дисках. При необходимости.

Или разные БД могут быть на разных табличных пространствах.

Посмотрим, какое табличное пространство использует БД:

```sql
SELECT pg_database.datname, pg_tablespace_location(pg_database.dattablespace) AS tablespace_location
FROM pg_database
WHERE datname = 'birds';
```

Видим пустой `tablespace_location`, значит, используется стандартное табличное пространство. Его расположение это директория `base` в `data_directory`, то есть `/var/lib/postgresql/16/main/base`. Отлично!

А где там данные нашей БД? Для этого можно достать идентификатор этой БД:

```sql
SELECT oid FROM pg_database WHERE datname = 'birds';
-- 32774
```

Значит, данные БД лежат в `data_directory/base/32774`.

В этой директории мы видим тоже файлы с именами из чисел — это идентификаторы таблиц.

Как посмотреть, где лежат данные таблицы?

```sql
select pg_relation_filepath('wide_employee');
--  base/32774/74776
```

Посмотрим:

```bash
sudo ls -l /var/lib/postgresql/16/main/base/32774/74776
```

При этом файлов может быть больше одного:

```bash
sudo ls -l /var/lib/postgresql/16/main/base/32774 | grep 74776

-rw------- 1 postgres postgres 1073741824 Sep 30 03:33 74776
-rw------- 1 postgres postgres  958382080 Sep 30 03:33 74776.1
```

Здесь мы видим 2 файла, два сегмента — сначала данные таблицы писались в первый файл, потом когда его размер превысил заданный порог в гигабайт, то создался файл второго сегмента данные стали писаться туда. Размер в гигабайт можно изменить при сборке постгреса при необходимости через свой параметр параметр `./configure --with-segsize`. Надо уметь собирать софт из исходников, чтобы получать такие возможности. Из пакетного менеджера вашей операционной системы вы поставите только то, что там для вас уже собрали с дефолтными параметрами.

Там могут быть еще файлы fsm — карта свободного пространства, и файлы vm — карта видимости.

```sql
update wide_employee set email='tmp@example.com' where employee_id =1;
```

Посмотрим файлы:

```bash
sudo ls -l /var/lib/postgresql/16/main/base/32774 | grep 74776

-rw------- 1 postgres postgres 1073741824 Sep 30 18:21 74776
-rw------- 1 postgres postgres  958382080 Sep 30 18:21 74776.1
-rw------- 1 postgres postgres     516096 Sep 30 18:21 74776_fsm
-rw------- 1 postgres postgres      65536 Sep 30 18:21 74776_vm
```

Отлично! Об этих fsm-файлах и vm-файлах можете почитать в документации или замечательной книге «PostgreSQL 16 изнутри», нам тут это пока неважно.

Ну так вот реально данные таблицы хранятся в этих великолепных файлах. Давайте посмотрим бинарное содержание файла, который хранит данные таблицы:

```bash
sudo xxd /var/lib/postgresql/16/main/base/32774/74776 | head -50 | less
```

Здесь мы видим те данные, которые мы записали в эту таблицу. Кайф!

Как организованы эти файлы? Они организованы в блоки по 8 Кбайт, которые называют также страницами. Этот размер в 8 Кбайт можно изменить при сборке постгреса, но как правило он именно 8 Кбайт. И это минимальное количество данных, которое постгрес может прочесть с диска.

Если нам надо достать одно маленькое число, хранимое в таблице — Postgres прочитает как минимум 8 Кбайт. В этих 8 Кбайт будет несколько строк таблицы, и в одной из этих строк будет нужное нам число.

Когда страница считалась из файла, она помещается в оперативную память в буферный кеш может храниться какое-то время в этом кеше.

Соответственно когда постгрес ищет данные, он может прочесть их из буферного кеша, то есть из оперативной памяти, то есть это очень быстро, или он может прочесть эти данные с диска, что будет медленнее.

Что тут надо понять, что тут интересно?

Интересно тут, во-первых, то, что все файлы побиты на 8 Кбайт и система работает с такими блоками. Во-вторых, мы понимаем, что хорошо, если в каждую страницу поместится больше строк таблицы, чтобы система могла считать меньшее количество страниц с диска. Потому что работа с диском медленная. Плюс в буферном кеше в оперативной памяти кешируются тоже страницы и если на странице будет много строк, то получается, что мы бОльшую часть таблицы можем закешировать в оперативной памяти, что тоже хорошо.

Иными словами — как правило не надо делать слишком широкие таблицы, состоящие из большого количества колонок. А это как раз приводит нас к использованию связи один к одному.

Вот мы храним данные сотрудников. Какие данные часто будут получаться? Какие данные по каждому сотруднику — основные? Вероятно это его уникальный идентификатор и имя-фамилия. А остальные данные нужны в своих сценариях отдельных. Бухгалтерии нужны поля по зарплате, кадровикам по грейду и дате трудоустройства и тд.

Когда мы храним все эти данные вместе в одной широкой таблице, то мы вынуждены считывать с диска все эти данные, даже если нам нужны только имя и фамилия.

То есть когда мы пишем запрос:

```sql
select first_name, last_name from wide_employee;
```

система считывает с диска в том числе адреса сотрудников, их грейды, зарплаты и все прочие данные. Это неэффективно.

А когда в основной таблице по сотрудникам только необходимые поля, то считываются только они. Это эффективнее.

Да, когда у нас данные побиты по разным таблицам, когда они нужны вместе, нам надо использовать join и это небесплатно. Но join работает все равно достаточно быстро благодаря умным алгоритмам postgres.


## Дальше

Проверить когда достаём по PK, что получается.

Ребутаем сервер.

Широкая таблица:

```sql
explain (analyze, buffers)
select employee_id, first_name, last_name
from employee
WHERE employee_id=12381
-- Buffers: shared hit=3 read=4
-- Execution Time: 0.082 ms
```

Узкие таблицы:

```sql
EXPLAIN (analyze, buffers)
SELECT e.employee_id, e.first_name, e.last_name
FROM employee e
WHERE employee_id=12381
-- uffers: shared hit=3 read=4
-- Execution Time: 0.070 ms
```

Сравнить размер СУБД (с учетом индексов PK)
Сказать про сценарий является и владеет (или как-то так)

- **Оптимизация данных для редких обновлений**. В некоторых случаях данные обновляются неравномерно — часть данных обновляется часто, другая — редко. Чтобы избежать частых блокировок и конкуренции за ресурсы при обновлении всей записи, можно вынести редко обновляемые данные в отдельную таблицу.
- **Наследование или расширение сущностей**. Это полезно в случаях, когда у вас есть базовая сущность с общими атрибутами, а дополнительные специфические атрибуты могут быть разнесены по разным таблицам. Таким образом, вы можете хранить основную информацию в одной таблице, а дополнительные детали в других, каждая из которых имеет один к одному связь с основной таблицей.
- **Производительность**: Если часто используется только часть данных, запросы к базе данных становятся быстрее, так как они не обрабатывают ненужные колонки.
- 

посмотреть размер БД:

```sql
SELECT pg_size_pretty(pg_database_size('wide_tables')) AS db_size;
-- 1145 MB
SELECT pg_size_pretty(pg_database_size('norm_tables')) AS db_size;
-- 1336 MB
```

Да, узкие таблицы занимают больше места на диске. Как минимум поля первичных ключей и их индексы дублируются в каждой таблице. Является ли это проблемой — в каждом конкретном случае решать вам.

Юзеры — и расширяющие их таблицы видов юзеров, например, клиенты компании и сотрудники компании.

много null-значений

[[Сценарий]]
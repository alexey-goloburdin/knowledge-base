Здоров, котаны, поговорим в этом видео о недооценённой связи таблиц в реляционных базах данных один к одному, о том, как связь один к одному может ускорить вашу базу данных на примере PostgreSQL и упростить разработку.

Есть мнение, что связи один ко многим и многие ко многим необходимы, а связь один к одному, вообще говоря, лишняя. Потому что можно просто не разносить данные по разным таблицам, а собрать их в одной таблице и всё. Напомню, что связь один к одному это когда одной записи одной таблицы соответствует одна запись в другой таблице. Соответственно технически можно просто собрать это всё в одной таблице и не разносить по разным.

И некоторые специалисты так и говорят — фигачьте всё в 1 таблицу и не парьтесь. В том числе, например, Владимир Хориков, автор отличной книги о юнит-тестировании и автор блога https://enterprisecraftsmanship.com/. Он [пишет](https://enterprisecraftsmanship.com/posts/modeling-relationships-in-ddd-way/) в статье о моделировании отношений в DDD следующее: «Такие отношения совершенно не нужны. Вы не получите от них никакой пользы. Единственное, что делают отношения «один к одному», - это загромождают вашу базу данных. Если две таблицы соотносятся друг с другом как один к одному, объедините их в одну таблицу».

Так ли это? Попробуем разобраться в этом видео, прям вот потыкаем, проведём эксперименты, разберёмся, как работает база данных изнутри, как где в каких файлах она хранит данные, как достаёт их и может ли связь один к одному быть полезной и какие конкретные результаты она может принести. Погнали!

## Настройка

Для начала быстренько поставим PostgreSQL, чтобы показать, что все примеры я буду приводить на реальном постгресе на его дефолтных настройках.

>[!info] Интеграция
>И базы данных, конечно, надо где-то хостить. Здесь есть много вариантов — можно купить сервер и поставить его в стойку датацентра, можно проще — арендовать сервер или можно ещё проще — воспользоваться управляемой базой данных, managed PostgreSQL, то есть уже настроенным, оптимизированным кластером с резервным копированием и всем, чем нужно. А где же это всё можно сделать? А конечно же, это лучше всего сделать у наших друзей в СелектЭл!
>
>А если вы не знакомы с СелектЭл, то есть отличная возможность стать ближе! 10 октября в Москве они проведут свою флагманскую конференцию — Selectel Tech Day. Там можно послушать классные доклады, задать свои вопросы специалистам Selectel и просто повеселиться. Ещё где-то там вы встретите меня, потому что я тоже пойду, послушаю про прерываемые серверы в облаке и управляемый Kubernetes. Участие бесплатное по предварительной регистрации. А если вы не в Москве, то регистрируйтесь и смотрите онлайн.
>
>Я, кстати, был в дата-центре Selectel в Санкт-Петербурге и это, конечно, очень мощное зрелище. Там столько нюансов — начиная от выделенной линии электропитания прямо от электростанции до всяких хитрых систем вентиляции и шумоизоляции серверных, систем быстрого пожаротушения и прочего. Ну и сам вид огромных шкафов с серверами и мигающими светодиодами внушает... я бы даже сказал трепет. Смотришь и понимаешь, что вот он — век информации, а в этих дата-центрах его сердце. Да:)
>
>Регистрируйтесь по ссылке в описании и приходите на Selectel Tech Day 10 октября в центре событий в Москве. До встречи!

Итак, давай на Selectel новый сервер. Чистый Debian 12, 2 vCPU, 4 GB RAM, 30 GB HDD.


```bash
# as root on Selectel
# Добавим пользователя и войдём им.
adduser www
adduser www sudo
su - www

# as www
# поставим минимально необходимые мне пакеты, zsh и ph my zsh

sudo apt update
sudo apt install -y git gcc vim zip unzip zsh
# oh my zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Ставим актуальный вышедший недавно PostgreSQL 17 по инструкции
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt -y install postgresql

# стартуем  сервер
sudo systemctl start postgresql

# установим Linux-пользователю postgres пароль
sudo passwd postgres
```

## Тесты широкой таблицы

```bash
# войдём Linux-пользователем postgres и войдём в базу данных 
su - postgres
psql
```

Создадим нового пользователя БД и две БД — для широкой таблицы и для узких таблиц.

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

Позволим подключаться по паролю пользователю базы данных `sterx`:

```bash
exit
sudo sudo vim /etc/postgresql/17/main/pg_hba.conf
# add to file
host    all             rroom          0.0.0.0/0            md5

sudo systemctl restart postgresql

# установлю клиент с подсветкой синтаксиса pgcli
sudo apt install -y pgcli

# чтобы система не запрашивала пароль при каждом входе сохраним его в .pgpass
echo "localhost:5432:wide_tables:sterx:mycoolp@ssword" >> ~/.pgpass
echo "localhost:5432:norm_tables:sterx:mycoolp@ssword" >> ~/.pgpass
chmod 600 ~/.pgpass

# входим в БД для широких таблиц
pgcli -h localhost -U sterx -d wide_tables
```

Отлично, теперь создадим широкую таблицу и наполним её данными. Ссылку на скрипты я приложу в описании видео.

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
    -- Случайные значения полей some_bullshit
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

Здесь мы видим buffers shared hit — это прочитанные из буферного кеша страницы, а read это прочитанные из файловой системы страницы. Как видим, было прочитано 142 тысячи страниц с диска.

## Тесты узких таблиц

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

Тут `temp read` и `written` говорят о том, что использовались файлы для кеширования — потому что серверу не хватило рабочей памяти. Увеличим параметр `work_mem`, по умолчанию 4 мегабайта и это мало. Этот параметр отвечает за количество памяти, которое может использовать запрос для внутренних операций, например, сортировки. 

Ребутнем сервер, чтобы очистить буфер:

```bash
sudo systemctl restart postgresql
pgcli -h localhost -U sterx -d norm_tables
```

Увеличим значение `work_mem`:

```sql
show work_mem;
set work_mem='128MB';

# И смотрим новый результат:
EXPLAIN (analyze, buffers)
SELECT e.employee_id, e.first_name, e.last_name
FROM employee e
JOIN employee_hierarchy eh ON e.employee_id = eh.employee_id
WHERE department_id='HR' order by employment_date limit 20;
-- Buffers: shared hit=149 read=8627
-- Execution Time: 87.218 ms
```

Получаем 87 мс против 234 мс время выполнения запроса и считанных страниц 9 тысяч против 146 тысяч. Неплохой результат!

## Но ты не используешь индекс!

Да, у нас тут запрос по непроиндексированным полям происходит, что приводит к необходимости читать всю таблицу, выполняется Seq Scan, то есть вся таблица последовательно читается, все файлы, которые хранят данные таблицы.

И знатоки в кавычках скажут — а надо же индекс, индекс добавить, чтобы всё работало по уму! Это же правило номер один у них, если что-то работает медленно, накинь индекс!

Однако в данном случае и добавление индекса не изменит ситуацию, потому что селективность этого индекса будет низкой. Индексы используются при высокой селективности запроса, то есть когда возвращается мало данных из большой таблицы. А когда селективность слабая, то есть возвращается большой процент данных всей таблицы, то проще просканировать всю её и не трогать индекс.

Проверим?

А давайте проверим, шо нам кабанам!

```bash
pgcli -h localhost -U sterx -d wide_tables
```

```sql
create index idx_employee_department_id on employee(department_id);

vacuum full analyze employee;
```

Ребутнем сервер для чистоты эксперимента и проверим:

```bash
sudo systemctl restart postgresql
pgcli -h localhost -U sterx -d wide_tables
```

```sql
explain (analyze, buffers)
select employee_id, first_name, last_name
from employee
where department_id='HR' order by employment_date limit 20;
-- Buffers: shared hit=114 read=142858
-- Execution Time: 253.995 ms
```

Как видим, индекс не используется, по-прежнему обходится вся таблица `employee` «Parallel Seq Scan on employee».

## А шо так? А как работает Postgres?

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

## А шо там по размеру?

Однако когда у нас одна таблица, то наверное, она меньше места занимает на диске, чем когда таблиц много?

Давайте посмотрим:

```sql
select pg_size_pretty(pg_database_size('wide_tables')) as db_size;
-- 1145 MB
select pg_size_pretty(pg_database_size('norm_tables')) as db_size;
-- 1336 MB
```

Да, узкие таблицы занимают больше места на диске. Как минимум поля первичных ключей и их индексы дублируются в каждой таблице. Является ли это проблемой — в каждом конкретном случае решать вам.

При этом всегда ли это будет так? Не всегда. Если колонки могут хранить `null`, то схема с разными таблицами легко может быть более эффективной и по хранилищу. Например, если только у некоторых сотрудников есть данные адреса, то в широкой таблице надо хранить много значений null, а в схеме с 1 к одному в таблице с адресами просто будут только те сотрудники, у которых есть адреса. Это тоже приятно.

## Когда стоит подумать об использовании связи один к одному?

Когда у вас слишком широкая таблица из десятков и тем более сотен колонок. Когда люди игнорируют связь один к одному, это на больших проектах нередко приводит к таким таблицам. Мне рассказывали о таблицах на 800 колонок. С этим маловозможно уже работать.

На практике стоит основные данные выделить в основную таблицу, а дополнительные данные выносить в таблицу или таблицы со связью один к одному или один к нулю или одному для данных, которых может не быть. Если в таблице много значений null — опять же возможно стоит разбить таблицу на несколько.

Второй сценарий — когда один данные используются часто, а другие редко. Скажем, имя сотрудинка используется примерно во всех сценариях работы с ним, а его зарплата только в сценариях бухгалтерии, а дата трудоустройства только в сценариях для HR, например. Такие данные стоит разносить по разным таблицам, потому что запросы, которые реализуют частые сценарии, не будут тогда обрабатывать данные, которые им сейчас не нужны.

Например, часто данные для аналитики выносят в отдельные таблицы, то есть данные, которые нужны только в аналитических отчётах и нигде больше. Скажем, данные, откуда пришел клиент, из какого рекламного канала. Если это знание не используется нигде кроме как в аналитике, то незачем эти данные хранить в той же таблице, где хранится имя клиента, которое используется практически во всех сценариях работы с клиентом.

Третий сценарий — разделение часто изменяемых от редко изменяемых данных. Это тоже может повысить производительность, оптимизировать индексы, снизить фрагментацию данных и тд.

И четвертый сценарий, который связан с первым — это когда просто по мере развития проекта появляются новые грани наших сущностей. Можно добавлять эти новые поля в таблицу с основными данными сущности, а можно выносить в отдельные, чтобы не возникала каша, чтобы не разрасталась основная таблица, чтобы не было блокировок основной таблицы при добавлении новых колонок в неё и так далее.

Пятый сценарий, когда надо уточнить базовые данные. Например, мы пишем систему, в которой работают и клиенты, и сотрудники компании. Можно создать таблицу user, и сделать таблицы client и employee со связями один к нулю или одному. Базовая общая информация хранится в таблице пользователей, например, логин и хеш пароля, а специфичная информация по клиентам хранится в таблице клиентов, специфичная информация по сотрудникам — в таблице сотрудников соответственно.

## Выводы

Таким образом, подведём вывод — неправильно говорить, что связь один к одному не нужна, не несёт никакой ценности и можно просто класть данные в одну таблицу. Это не так. Используйте связь один к одному и пусть ваши базы данных работают быстро и эффективно, а процесс разработки будет лёгок, просто и приятен:)

Напоминаю, что у меня сейчас идет курс Хардкорная веб-разработка, на котором мы в частности подробно разбираем и SQL, и работу PostgreSQL. Приходите, ссылка на курс в описании под видео.

Спасибо, что досмотрели это видео и остаёмся на связи. Пока-пока!

[[Сценарий]]
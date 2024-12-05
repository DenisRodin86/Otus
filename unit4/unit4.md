Домашнее задание 4



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit4/4-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit4/4-2.jpg)




Заходим под ролью `postgres`


```bash
sudo su - postgres
```


Заходим в `psql`

```bash
psql
```


Задаем пароль роли `postgres`

```bash
alter user postgres encrypted password 'postgres';
```


Создаем новую базу данных

```bash
CREATE DATABASE testdb;
```


Заходим в созданную базу данных под пользователем `postgres`

```bash
\c testdb
```


Создаем новую схему `testnm`

```bash
CREATE SCHEMA testnm;
```


Создаем новую таблицу `t1` с одной колонкой `c1` типа `integer`

```bash
CREATE TABLE t1(c1 integer);
```


Вставляем строку со значением `c1=1`

```bash
INSERT INTO t1 values(1);
```


Создаем новую роль `readonly`

```bash
CREATE role readonly;
```


Дадим новой роли право на подключение к базе данных `testdb`

```bash
grant connect on DATABASE testdb TO readonly;
```


Даём новой роли право на использование схемы `testnm`

```bash
grant usage on SCHEMA testnm to readonly;
```


Даём новой роли право на `select` для всех таблиц схемы `testnm`

```bash
grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
```


Создаём пользователя `testread` с паролем `test123`

```bash
CREATE USER testread with password 'test123';
```


Даём роль `readonly` пользователю `testread`

```bash
grant readonly TO testread;
```


Зайдем под пользователем `testread` в базу данных `testdb`
Для этого сначала исправим в файле `pg_hba.conf` метод шифрования с `peer` на `md5` для локального входа

```bash
\c testdb testread
```

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit4/4-3.jpg)



Сделаем select * from t1;

**Результат: ОШИБКА:  нет доступа к таблице t1**


Таблица создана в схеме `Public`, а права мы выдаем в схеме `testnm`

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit4/4-4.jpg)



Возвращаемся в базу данных `testdb` под пользователем `postgres`

```bash
\c testdb postgres
```


Удаляем таблицу `t1`

```bash
DROP TABLE t1;
```


Создаем ее заново но уже с явным указанием имени схемы `testnm`

```bash
CREATE TABLE testnm.t1(c1 integer);
```


Вставляем строку со значением `c1=1`

```bash
INSERT INTO testnm.t1 values(1);
```


Зайдем под пользователем testread в базу данных testdb

```bash
\c testdb testread
```


Сделаем `select * from testnm.t1;`

```bash
select * from testnm.t1;
```

**ОШИБКА:  нет доступа к таблице t1**

`grant SELECT on all TABLEs in SCHEMA testnm TO readonly` дал доступ только для существующих на тот момент времени таблиц а `t1` пересоздавалась


Исправляем данную ошибку, чтобы в будующем её избежать

```bash
\c testdb postgres; 
ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
\c testdb testread;
```


Сделаем `select * from testnm.t1;`

```bash
select * from testnm.t1;
```

**ОШИБКА:  нет доступа к таблице t1**

`ALTER default` будет действовать для новых таблиц а `grant SELECT on all TABLEs in SCHEMA testnm TO readonly` отработал только для существующих на тот момент времени. надо сделать снова или `grant SELECT` или пересоздать таблицу

```bash
\c testdb postgres
grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
\c testdb testread;
```


Сделаем `select * from testnm.t1;`

```bash
select * from testnm.t1;
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit4/4-5.jpg)



Выполним команды

```bash
create table t2(c1 integer); 
insert into t2 values (2);
```

**ОШИБКА:  нет доступа к схеме public**


Убираем эти права

```bash
\c testdb postgres; 
REVOKE CREATE on SCHEMA public FROM public; 
REVOKE ALL on DATABASE testdb FROM public; 
\c testdb testread; 
```


Пользователю разрешаем создавать объекты в схеме, не принадлежащей ему. Первое слово «public» обозначает схему, а второе означает «каждый пользователь». В первом случае это идентификатор, а во втором — ключевое слово, поэтому они написаны в разном регистре;
Пользователю разрешаем всё в базе данных `testdb` в базе данных, не принадлежащей ему. 


Выполним команды

```bash
create table t3(c1 integer);  
insert into t2 values (2);
```

Так как использует `postgres 15` то выдается ошибка

**ОШИБКА:  нет доступа к схеме public**

в 15 версии права на `CREATE TABLE` по умолчанию отозваны у схемы `PUBLIC`, только `USAGE`
чтобы пользователь мог создавать объекты в этой схеме, ему нужно выдать привилегию `CREATE` на схему


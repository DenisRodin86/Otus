Домашнее задание 12



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Создаем БД


```bash
create database unit12_db;
```



Выдача прав на БД


```bash
grant all privileges on database unit12_db to postgres;
```


Создание схемы в БД `unit12_db`


```bash
 \c unit12_db
CREATE SCHEMA unit12_sh;
```


Создание таблицы и заполнение ее данными


```bash
 create table unit12_sh.unit12_tb as
select
generate_series(1,100) as id,
md5(random()::text)::char(10) as fio;
```


Создание директории для резервного копирования


```bash
mkdir /tmp/backup
```


Создание логического бэкап таблицы `unit12_sh.unit12_tb`, используя утилиту `COPY`


```bash
\copy unit12_sh.unit12_tb to '/tmp/backup/backup_copy_unit12_tb.sql';
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-2.jpg)



Проверяем созданный файл 


```bash
nano /tmp/backup/backup_copy_unit12_tb.sql
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-3.jpg)



Создание таблицы для востановления данных


```bash
create table unit12_sh.unit12_tb_backup (
id integer,
fio text);
```


Проверка созданной таблицы



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-4.jpg)



Востановление данных из резервной копии в новую таблицу


```bash
\copy unit12_sh.unit12_tb_backup from '/tmp/backup/backup_copy_unit12_tb.sql';
```


Все строки скопированы




![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-5.jpg)




Проверка заполнености таблицы `unit12_sh.unit12_tb_backup`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-6.jpg)



Используя утилиту `pg_dump`, создаем бэкап в кастомном сжатом формате двух таблиц


```bash
pg_dump -d unit12_db --compress=9 --table=unit12_sh.unit12_tb --table=unit12_sh.unit12_tb_backup -Fc > /tmp/backup/backup_unit12_tables.gz
```


Просмотр созданого файла бэкапа




![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-7.jpg)



Проверка содержимого бекапа


```bash
pg_restore --list /tmp/backup/backup_unit12_tables.gz
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-8.jpg)



Создание новой БД, для востановление в нее данных


```bash
create database unit12_db_restore;
```


Выдача прав на БД


```bash
grant all privileges on database unit12_db_restore to postgres;
```


Создание схемы в БД `unit12_db`


```bash
 \c unit12_db_restore
CREATE SCHEMA unit12_sh;
```


Востановление таблицы `unit12_sh.unit12_tb_backup` в БД `unit12_db_restore`


```bash
pg_restore -d unit12_db_restore --table=unit12_sh.unit12_tb_backup /tmp/backup/backup_unit12_tables.gz
```


Проверка востановления



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit12/12-9.jpg)



Для упрощения обращения к файлу резервных копий они были размещены в директории /tmp, при использовании в "боевой" среде такого следует избегать.

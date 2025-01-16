Домашнее задание 13



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-1.jpg)



Схема виртуальных серверов



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-2.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service


Настроем сервера для взаимодействия между собой


```bash
sudo nano /var/lib/pgsql/15/data/postgresql.conf
		Замените строку:# listen_addresses = 'localhost'
		На: listen_addresses = '*' 

		wal_level = logical

sudo nano /var/lib/pgsql/15/data/pg_hba.conf
		host all all 192.168.5.0/24 md5
```


Перезапуск сервиса


```bash
sudo systemctl restart postgresql-15.service
```


Создаем БД на всех серверах


```bash
create database replica;
```


Создаем две таблицы `test1` и `test2`


```bash
create table test1 (i int);
create table test2 (i int);
```


Cоздание публикации таблицы `test1` на сервере `Rodin-RedOS-Education-Postgres`


```bash
create publication pub_test for table test1;
```


Проверка


```bash
\dRp+
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-3.jpg)




Cоздание публикации таблицы `test2` на сервере `Rodin-RedOS-Education-Postgres-1`


```bash
create publication pub_test for table test2;
```


Проверка


```bash
\dRp+
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-4.jpg)



Создание подписки на сервере `Rodin-RedOS-Education-Postgre`s на публикацию `pub_test` сервера `Rodin-RedOS-Education-Postgres-1`


```bash
create subscription sub_test
connection 'host=192.168.5.223 port=5432 user=postgres password=yourpassword dbname=replica'
publication pub_test with (copy_data = false);
```


Проверка


```bash
\dRs+
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-5.jpg)



Создание подписки на сервере `Rodin-RedOS-Education-Postgres-1` на публикацию `pub_test` сервера `Rodin-RedOS-Education-Postgres`


```bash
create subscription sub_test
connection 'host=192.168.5.222 port=5432 user=postgres password=yourpassword dbname=replica'
publication pub_test with (copy_data = false);
```


Проверка


```bash
\dRs+
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-6.jpg)



На сервере `Rodin-RedOS-Education-Postgres-2` создаем подписку на публикацию `pub_test` `Rodin-RedOS-Education-Postgres` сервера


```bash
create subscription sub1_test
connection 'host=192.168.5.222 port=5432 user=postgres password=yourpassword dbname=replica'
publication pub_test with (copy_data = false);
```


На сервере `Rodin-RedOS-Education-Postgres-2` создаем подписку на публикацию `pub_test` `Rodin-RedOS-Education-Postgres-1` сервера


```bash
create subscription sub2_test
connection 'host=192.168.5.223 port=5432 user=postgres password=yourpassword dbname=replica'
publication pub_test with (copy_data = false);
```


Проверка


```bash
\dRs+
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-7.jpg)



Проверка работоспособности


На сервере `Rodin-RedOS-Education-Postgres` вставим данные в таблицу `test1`


```bash
insert into test1 (i) values (1), (2), (3), (4), (5);
```


На сервере `Rodin-RedOS-Education-Postgres-1` вставим данные в таблицу `test2`


```bash
insert into test2 (i) values (6), (7), (8), (9), (0);
```


На сервере `Rodin-RedOS-Education-Postgres`


```bash
select * from test2;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-8.jpg)



На сервере `Rodin-RedOS-Education-Postgres-1`


```bash
select * from test1;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-9.jpg)



На сервере `Rodin-RedOS-Education-Postgres-2`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit13/13-10.jpg)

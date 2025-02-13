Дипломный проект

Схема сети



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-1.jpg)



Схема сети состоит из Prod (Продуктивного контура) и Test (Тестового контура)


Виртуальная машина на `RedOS`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-2.jpg)



Производим обновление системы

```bash
dnf update
```


Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Дополнительно устанавливаются компоненты

```bash
sudo dnf install postgresql15-contrib
```


Подключение к PostgreSQL через pgadmin4, а так же для взаимодействия между реплицированными серверами вносим изменения в файл postgresql.conf и pg_hba.conf

```bash
sudo nano /var/lib/pgsql/15/data/postgresql.conf
		Замените строку:# listen_addresses = 'localhost'
		На: listen_addresses = '*' 
	sudo nano /var/lib/pgsql/15/data/pg_hba.conf
		host all all 0.0.0.0/0 md5
```


Задаем пароль пользователю postgres 

```bash
ALTER USER postgres WITH ENCRYPTED PASSWORD '111111';
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```


****Настройка репликации****


Сервер 1 *Master-сервер*


Создание пользователя `replication` для репликации между серверами

```bash
CREATE ROLE replication WITH REPLICATION PASSWORD '111111' LOGIN;
```

В файле `pg_hba.conf` вносим строку подключения к *slave*

```bash
host    replication     replication     192.168.5.228/32         md5	
```


В файле `postgresql.conf` вносим изменения

```bash
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1024
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```



****Настройка доп. сервера (slave)****


В файле `pg_hba.conf` вносим строку подключения к master

```bash
host    replication     replication     192.168.5.223/32         md5	
```

В файле `postgresql.conf` вносим изменения

```bash
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1024
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```

Удаляем директорию установки `postgres`

```bash
rm -Rf /var/lib/pgsql/15/data/*
```

В директории `/var/lib/pgsql/15/`

```bash
rm -rf main; mkdir main; chmod go-rwx main
pg_basebackup -h IP -D /var/lib/pgsql/15/data -P -U replication --wal-method=stream
```

В файле `postgresql.conf` вносим изменения

```bash
hot_standby = on
primary_conninfo = 'user=replication password=111111 host=192.168.5.223 port=5432 sslmode=prefer sslcompression=1 krbsrvname=postgresql-15.service'
```

Выполняем процесс синхронизации

```bash
touch standby.signal
chown postgres:postgres standby.signal
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```

Проверка работы репликации на `master` создаем `БД db1`

```bash
create database db1;
```

Создадим таблицу `table1` в этой БД

```bash
create table table1 (
id integer,
fio text
);
```

Заполнение таблицы `table1` данными

```bash
insert into table1 (id, fio) values (1, 'Иванов И'), (2, 'Родин Д'), (3, 'Семенов С');
```


Вывод таблицы 

```bash
select * from table1 ;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-3.jpg)



на *slave* выполним запрос к `table1` БД `db1`

```bash
select * from table1;
```


Получаем аналогичный вывод.

***Репликация работает.***




****Настройка сервера тестового контура.****

Сервер тестового контура будет забирать БД с slave сервера, чтобы не нагружать master сервер.

Для начала на сервере slave в файл pg_hba.conf внесем адрес тестового сервера

host    all     all     192.168.5.222/32         md5	


На самом сервере тестового контура так же внесем изменения

В файле pg_hba.conf вносим строку подключения к slave

host    all     all     192.168.5.228/32         md5


В файле  postgresql.conf
Замените строку:# listen_addresses = 'localhost'
На: listen_addresses = '*' 

Скрипт обновления БД распологается в директории пользователя, в данном случае /home/rodin/

При выполнении процедуры резервного копирования дамп БД сохраняется в директорию /backups/<имя продуктивного сервера>/. В эту же директорию записывается лог выполнения процедуры. 

Для выполнения скриптов прав администратора не требуется. Хранение логинов и паролей для подключения к БД PostgreSQL осуществляется в файле .pgpass, который находится в домашней директории авторизованного пользователя.

При повторном выполнении скрипта предыдущие дампы БД заменяются текущими.

При выполнении процедуры резервного копирования/восстановления БД PostgreSQL схема и данные копируются по отдельности.



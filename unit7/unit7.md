Домашнее задание 7



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```

***Вопрос 1***

Для того, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд.
необходимо изменить конфигурационный файл `postgresql.conf` 


```bash
nano /var/lib/pgsql/15/data/postgresql.conf
```


Неоходимо изменить параметры:

`deadlock_timeout`

`log_lock_waits (Определяет, нужно ли фиксировать в журнале события, когда сеанс ожидает получения блокировки дольше, чем указано в deadlock_timeout.)`


По умолчанию эти параметры закаментированы:


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-2.jpg)



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-3.jpg)



Установим параметры на 200 ms:



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-4.jpg)



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-5.jpg)



Перезагрузим кластер, для применения настроек


```bash
sudo systemctl restart postgresql-15.service
```


***Вопрос 2***


Создаем БД `test_locks`:


```bash
create database test_locks;
```


в БД `test_locks` создал таблицу `users`


```bash
create table users (id int, "name" text);
```


Вносим данные:

```bash
insert into users (id, "name") values (1, 'Ivanov'), (2, 'Petrov'); 
```


Запуск ещё двух `ssh` сеанса к серверу


Во всех сеансах выполняем команду update

```bash
begin;
update users set name = 'Sidorov' where id = 1; 
```


Выполним в первом сеансе `ssh`


```bash
select locktype, pid, relation::regclass, virtualxid as virtxid, 
       transactionid as xid, mode, granted 
from pg_locks 
where pid <> pg_backend_pid() 
order by pid, locktype;
```


Результат блокировок


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-6.jpg)



Результаты анализа `pg_locks`:
 
транзакциям присвоены виртуальные идентификаторы virtualxid => 4/105, 5/11 и эти номера успешно (granted=t) заблокированы самими транзакциями в режиме исключительной блокировки (ExclusiveLock)

транзакции дополнительно получили физические номера transactionid (xid) => 745, 744, 746 при попытке изменить данные командой update, и самостоятельно их удерживают в режиме ExclusiveLock

также из-за команды update появились блокировки с типом relation - блокировки отношений (таблицы users), в режиме RowExclusiveLock, выданы (granted=t), и самостоятельно удерживаются транзакциями

вторая транзакция ожидает завершения первой транзакции - попыталась (granted=f) получить блокировку номера transactionid = 744 (первой транзакции) в режиме ShareLock, и наложила (granted=t) блокировку tuple (блокировка версии строки) на обновляемую строку в режиме ExclusiveLock 

третья транзакция тоже попыталась (granted=f) получить блокировку tuple на обновляемую строку в режиме ExclusiveMode, но неудачно, так как вторая транзакция уже наложила такую блокировку

После завершения первой транзакции, ситуация с третьей будет как во второй.


```bash
drop table users (id int, "name" text);
```

***Вопрос 3***


Создание таблицы users в БД `test_locks`:


```bash
create table users (id int, name text, salary int);
```


Заполнение данными


```bash
insert into users (id, name, salary) 
values 
  (1, 'Ivanov', 35000),
  (2, 'Petrov', 28000),
  (3, 'Sidorov', 41000);
```


Выполним команды `update` в трех терминалах

**Терминал 1**	

```bash
begin;
update users
set salary = salary + 3000
where id = 1;
```


**Терминал 2**


```bash
begin;
update users
set salary = salary + 1000
where id = 2;
```


**Терминал 3**


```bash
begin;
update users
set salary = salary + 6000
where id = 3;
```


**Терминал 1**


```bash
update users
set salary = salary + 5000
where id = 2;
```


**Терминал 2**


```bash
update users
set salary = salary + 9000
where id = 3;
```


**Терминал 3**


```bash
update users
set salary = salary + 7000
where id = 1;
```


При выполнении появляется ощибка



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-7.jpg)



```bash
commit;
```



**Терминал 2**


```bash
commit;
```



**Терминал 1**


```bash
commit;
```


Просмотр логов в файле   `/var/lib/pgsql/15/data/log/postgresql-Thu.log`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit7/7-8.jpg)



В файл логирования попадает информация о блокировках





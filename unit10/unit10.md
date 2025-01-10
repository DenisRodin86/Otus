Домашнее задание 10



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit10/10-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```

Скачиваем демо базу

```bash
wget https://edu.postgrespro.ru/demo_small.zip && sudo dnf install unzip && unzip demo_small.zip && sudo -u postgres psql -d postgres -f /home/yc-user/demo_small.sql -c 'alter database demo set search_path to bookings' 
```


Распаковывем БД `demo`


```bash
unzip demo_small.zip
```


Устанавливаем демо базу


```bash
 psql -f /var/lib/pgsql/15/data/demo_small.sql
```


Проверка установки



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit10/10-2.jpg)



Подключаемся к `demo`


```bash
\c demo
```


Просмотр таблиц


```bash
select * from bookings.flights;
```


Для секционирования будем использована таблица `ticket_flights`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit10/10-3.jpg)



`DDL` данные


```bash
pg_dump -t 'bookings.ticket_flights' --schema-only demo
```


Создаем таблицу


```bash
CREATE TABLE bookings.ticket_flightsNEW ( 
ticket_no character(13) NOT NULL,
flight_id integer NOT NULL,
fare_conditions character varying(10) NOT NULL,
amount numeric(10,2) NOT NULL,
CONSTRAINT ticket_flights_amount_check CHECK ((amount >= (0)::numeric)),
CONSTRAINT ticket_flights_fare_conditions_check CHECK (((fare_conditions)::text = ANY (ARRAY[('Economy'::character varying)::text, ('Comfort'::character varying)::text, ('Business'::character varying)::text])))
)
partition by hash(ticket_no); 
```


Создание секций


```bash
create table bookings.ticket_flights_1 partition of bookings.ticket_flightsnew for values with (modulus 5, remainder 0);
create table bookings.ticket_flights_2 partition of bookings.ticket_flightsnew for values with (modulus 5, remainder 1);
create table bookings.ticket_flights_3 partition of bookings.ticket_flightsnew for values with (modulus 5, remainder 2);
create table bookings.ticket_flights_4 partition of bookings.ticket_flightsnew for values with (modulus 5, remainder 3);
create table bookings.ticket_flights_5 partition of bookings.ticket_flightsnew for values with (modulus 5, remainder 4);
```


Перенос данных в новую таблицу


```bash
INSERT INTO bookings.ticket_flightsnew SELECT * FROM bookings.ticket_flights;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit10/10-4.jpg)



Проверка



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit10/10-5.jpg)



Сумма записей в секциях равно количество записей в исходной таблице


```bash
explain select * from bookings.ticket_flightsnew where flight_id='24836';
```


Из вывода команды `explain` видно, что секционирование отрабатывает корректно.

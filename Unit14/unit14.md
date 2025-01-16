Домашнее задание 14



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service


Создание таблицы и заполнение ее данными


```bash
create table staff (
id integer,
fio text
);

create table status (
id integer,
position text);
```


Заполнение таблицы


```bash
insert into staff (id, fio) values (1, 'Иванов И'), (2, 'Родин Д'), (3, 'Семенов С'), (8, 'Смирнов П'), (5, 'Петров М');
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-2.jpg)




```bash
insert into status (id, position) values (1, 'Директор'), (2, 'Конструктор'), (3, 'Бухгалтер'), (4, 'Сторож'), (5, 'Юрист');;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-3.jpg)



**Прямое соединение двух таблиц (Определяем должность сотрудника)**


```bash
select a.id id_a,
a.fio fio_a,
b.id id_b,
b.position position_b
from
staff a
inner join status b on a.id = b.id;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-4.jpg)



**Левостороннее соединение двух таблиц (Определяем сотрудников без должности)**


```bash
select a.id id_a,
a.fio fio_a,
b.id id_b,
b.position position_b
from
staff a
left join status b on a.id = b.id;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-5.jpg)



**Кросс соединение двух таблиц**


```bash
select a.id id_a,
a.fio fio_a,
b.id id_b,
b.position position_b
from
staff a
cross join status b;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-6.jpg)



**Полное соединение двух таблиц (Определяем сотрудников без должности и свободные вакансии)**


```bash
select a.id id_a,
a.fio fio_a,
b.id id_b,
b.position position_b
from
staff a
full join status b on a.id = b.id;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-7.jpg)



**Запрос с разными типами соединений (`inner` и `cross`)**


```bash
select a.id id_a,
a.fio fio_a,
b.id id_b,
b.position position_b
from
staff a
inner join status b on a.id = b.id
cross join status;
```

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Unit14/14-8.jpg)

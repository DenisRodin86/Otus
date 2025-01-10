Домашнее задание 9



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Создание таблицы 

```bash
create table staff
(
id integer,
FirstName text,
SecondName text,
Salary integer
 );
```


Заполняем ее случайными значениями, где поля `Salary` (премия) имеет значение 1 или 0 (есть или нет)


```bash
INSERT INTO staff (ID, FirstName, SecondName, Salary)
SELECT s, md5(random()::text), md5(random()::text), (random()::integer)
FROM generate_series(1, 1000000) s;
```


Создаем индекс для данной таблицы


```bash
create index idx_staff_id on staff(id);
```


Сделаем запрос и проанализируем его выполнение с помощью `explain analyze`


```bash
explain analyze
select ID, FirstName, SecondName, Salary  from staff where salary = 1 and ID  >=1000 and ID <= 10000;
```


Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-2.jpg)



Видно, что теперь используется `Index Scan`


Удалим и вновь создадим таблицу `staff`, заполнив ее случайными значениями


```bash
drop table staff;
```


Полнотекстовый индекс


```bash
create index hire_staff_idx on staff using gin (to_tsvector('english', SecondName));
```


Запрос


```bash
explain analyze
select ID, FirstName, SecondName, Salary  from staff where to_tsvector('english', SecondName) @@ to_tsquery('d15ae95dc7d27fc0c067b8591f94518b');
```


Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-3.jpg)




Видно, что теперь используется оптимизатор `Bitmap Index Scan`


**Индекс на часть таблицы или индекс на поле с функцией**


Удаление индекса


```bash
drop index hire_staff_idx;
```


Добавлен сгенерированный столбец типа `tsvector` использующий функцию `to_tsvector`


```bash
alter table staff
  add column textsearchable_SecondName tsvector
    generated always as (to_tsvector('english', coalesce(SecondName, ''))) stored;
```


Создание индекса для полнотекстового поиска, использовав новый столбец


```bash
create index hire_staff_idx on staff using gin (textsearchable_SecondName);
```


Запрос


```bash
explain analyze
select ID, FirstName, SecondName, Salary  from staff where textsearchable_SecondName @@ to_tsquery('d15ae95dc7d27fc0c067b8591f94518b');
```


Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-4.jpg)



При использовании индекса для полнотекстового поиска на столбец скорость выполнения запроса незначительно повысилась (уменьшилось время).


**Индекс на несколько полей**


Создание индекса


```bash
create index hire_SecondName_Salary_idx on staff (SecondName, Salary);
```


```bash
explain analyze
select ID, FirstName, SecondName, Salary  from staff where SecondName = 'd15ae95dc7d27fc0c067b8591f94518b' and Salary ='0';
```


Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-5.jpg)



Для сравнения такой же запрос без использование индексов



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit9/9-6.jpg)



Использование индексов существенно увеличивает производительность базы данных.


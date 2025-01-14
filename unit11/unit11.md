Домашнее задание 11



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Создание БД


```bash
DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;
SET search_path = pract_functions, publ
```


Создание таблицы `goods` (товары)


```bash
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозайственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);
```


Создание таблицы `sales` (продажи)


```bash
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);
```


Заполнение таблицы


```bash
INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
```


Отчет `goods`


```bash
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-2.jpg)



Отчет `sales`


```bash
select * from sales;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-3.jpg)



Денормализация БД



```bash
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);
``


Проверка


```bash
\dt
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-4.jpg)



Создание тригерной функции по таблице `sales`



**Добавление новой записи**


```bash
CREATE or replace function ft_insert_sales()
RETURNS trigger
AS
$TRIG_FUNC$
DECLARE
g_name varchar(63);
g_price numeric(12,2);
BEGIN
SELECT G.good_name, G.good_price*NEW.sales_qty INTO g_name, g_price FROM goods G where G.goods_id = NEW.good_id;
IF EXISTS(select from good_sum_mart T where T.good_name = g_name)
THEN UPDATE good_sum_mart T SET sum_sale = sum_sale + g_price where T.good_name = g_name;
ELSE INSERT INTO good_sum_mart (good_name, sum_sale) values(g_name, g_price);
END IF;
RETURN NEW;
END;
$TRIG_FUNC$
LANGUAGE plpgsql
VOLATILE
SET search_path = pract_functions, public;
```


**Удаление записи**


```bash
CREATE or replace function ft_delete_sales()
RETURNS trigger
AS
$TRIG_FUNC$
DECLARE
g_name varchar(63);
g_price numeric(12,2);
BEGIN
SELECT G.good_name, G.good_price*OLD.sales_qty INTO g_name, g_price FROM goods G where G.goods_id = OLD.good_id;
IF EXISTS(select from good_sum_mart T where T.good_name = g_name)
THEN 
UPDATE good_sum_mart T SET sum_sale = sum_sale - g_price where T.good_name = g_name;
DELETE FROM good_sum_mart T where T.good_name = g_name and (sum_sale < 0 or sum_sale = 0);
END IF;
RETURN NEW;
END;
$TRIG_FUNC$
LANGUAGE plpgsql
VOLATILE
SET search_path = pract_functions, public;
```


**Обновление записи**


```bash
CREATE or replace function ft_update_sales()
RETURNS trigger
AS
$TRIG_FUNC$
DECLARE
g_name_old varchar(63);
g_price_old numeric(12,2);
g_name_new varchar(63);
g_price_new numeric(12,2);
BEGIN
SELECT G.good_name, G.good_price*OLD.sales_qty INTO g_name_old, g_price_old FROM goods G where G.goods_id = OLD.good_id;
SELECT G.good_name, G.good_price*NEW.sales_qty INTO g_name_new, g_price_new FROM goods G where G.goods_id = NEW.good_id;
IF EXISTS(select from good_sum_mart T where T.good_name = g_name_new)
THEN UPDATE good_sum_mart T SET sum_sale = sum_sale + g_price_new where T.good_name = g_name_new;
ELSE INSERT INTO good_sum_mart (good_name, sum_sale) values(g_name_new, g_price_new);
END IF;
IF EXISTS(select from good_sum_mart T where T.good_name = g_name_old)
THEN 
UPDATE good_sum_mart T SET sum_sale = sum_sale - g_price_old where T.good_name = g_name_old;
DELETE FROM good_sum_mart T where T.good_name = g_name_old and (sum_sale < 0 or sum_sale = 0);
END IF;
RETURN NEW;
END;
$TRIG_FUNC$
LANGUAGE plpgsql
VOLATILE
SET search_path = pract_functions, public;
```



Добавление тригеров


```bash
CREATE TRIGGER tr_insert_sales
AFTER INSERT
ON sales
FOR EACH ROW
EXECUTE PROCEDURE ft_insert_sales();

CREATE TRIGGER tr_delete_sales
AFTER DELETE
ON sales
FOR EACH ROW
EXECUTE PROCEDURE ft_delete_sales();

CREATE TRIGGER tr_update_sales
AFTER UPDATE
ON sales
FOR EACH ROW
EXECUTE PROCEDURE ft_update_sales();
```




Проверка работы тригеров


**Добавление**



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-5.jpg)




**Удаления**



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-6.jpg)



**Обновления**



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit11/11-7.jpg)

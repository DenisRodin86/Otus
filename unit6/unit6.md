Домашнее задание 6



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-2.jpg)



Установка утилиты для просмотра аппартаной конфигурации сервера

```bash
sudo dnf install lshw
```


Просмотр конфигурации сервера

```bash
sudo lshw -short
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-3.jpg)



Устанавливаем дополнительные компоненты для `pgbench`

```bash
sudo dnf install postgresql-contrib
```


Создание БД для тестов

```bash
pgbench -i postgres
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-4.jpg)



Запускаем тест 


-c: количество одновременных клиентов или сеансов БД

-P: отображает прогресс и метрики каждые 60 секунд

-T: запустит тест на 60 секунд (1 минут).



```bash
pgbench -c8 -P 6 -T 60 -U postgres postgres
```


Результаты теста



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-5.jpg)



Применяем параметры настройки `PostgreSQL` из прикрепленного к материалам занятия файла


Редактируем файл `/var/lib/pgsql/15/data/postgresql.conf`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-6.jpg)



Перезагрузим кластер, для применения настроек

```bash
sudo systemctl restart postgresql-15.service
```


Запустим тест заново



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-7.jpg)



Немного упало значение `TPS` (`TPS` – это показатель пропускной способности базы данных, который показывает количество транзакций, обработанных базой данных за одну секунду), вследствии того, что мы установили принудительно 1 `CPU`, 


Создадим таблицу с текстовым полем и заполним случайными или сгенерированными данным в размере 1млн строк

```bash
CREATE TABLE student(
    fio char(100)
);
```

```bash
INSERT INTO student(fio) SELECT 'noname' FROM generate_series(1,1000000);
```


Посмотрим размер файла с таблицей

```bash
SELECT pg_size_pretty(pg_total_relation_size('student'));
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-8.jpg)




5 раз обновляем все строчки и добавляем к каждой строчке любой символ

```bash
update student set fio = 'noname1';
```


Посмотрим количество мертвых строчек в таблице и когда последний раз приходил автовакуум

```bash
SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'student';
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-9.jpg)



Подождём некоторое время, проверяя, пришел ли автовакуум



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-10.jpg)



5 раз обновляем все строчки и добавляем к каждой строчке любой символ

```bash
update student set fio = 'noname2';
```


Посмотрим размер файла с таблицей

```bash
SELECT pg_size_pretty(pg_total_relation_size('student'));
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-11.jpg)



Отключаем `autovacuum` на конкретной таблице

```bash
ALTER TABLE student SET (autovacuum_enabled = off);
```


10 раз обновляем все строчки и добавляем к каждой строчке любой символ

```bash
update student set fio = 'noname3';
```


Посмотрим размер файла с таблицей

```bash
SELECT pg_size_pretty(pg_total_relation_size('student'));
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit6/6-12.jpg)



При отключении autovacuum "мертвые" строчки не зачищаются в БД и она начинает "пухнуть". При всего лишь 1 млн. строчек она разрослась до 1.5 Гб
Автоматическая очистка перестаёт работать, таблицы и индексы перестают чиститься. В результате в них появляются мусорные строки, таблицы и индексы начинают расти в размерах.

Включаем `autovacuum` на конкретной таблице

```bash
ALTER TABLE student SET (autovacuum_enabled = on);
```

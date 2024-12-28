Домашнее задание 8



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Просмотр `default` настроек контрольных точек


```bash
show checkpoint_timeout;
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-2.jpg)


```bash
show log_checkpoints;
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-3.jpg)



Изменение параметров, для создания контрольных точек раз в 30 сек.


```bash
alter system set checkpoint_timeout = 30;
```



Перезапуск `postgresql`


```bash
sudo systemctl restart postgresql-15.service
```


Проверка внесенных изменений


```bash
show checkpoint_timeout;
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-4.jpg)



Устанавливаем дополнительные компоненты для `pgbench`


```bash
sudo dnf install postgresql-contrib
```


Создание БД для тестов


```bash
pgbench -i postgres
```


Просмотр текущего `LSN (Log Sequence Number)`


```bash
select pg_current_wal_insert_lsn();
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-5.jpg)



Просмотр количество до текущего момент контрольных точек


```bash
select checkpoints_timed from pg_stat_bgwriter;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-6.jpg)



Запускаем тест 

-c: количество одновременных клиентов или сеансов БД

-P: отображает прогресс и метрики каждые 60 секунд.

-T: запустит тест на 60 секунд (1 минут).


```bash
pgbench -c8 -P 60 -T 600 -U postgres postgres
```


После выполнения `pgbench`


Контрольных точек:



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-7.jpg)



`LSN`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-8.jpg)



Число байт в журнале предзаписи (WAL) до и после выполнения pgbench


```bash
select '0/29E5DE40'::pg_lsn - '0/220B690'::pg_lsn as byte_size;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-9.jpg)




Объем приходящийся в среднем на одну контрольную точку


```bash
667232176 / 29  =  23008006 байт ~ 22468 кбайт  ~ 29 Мбайт
```


Проверка записи о выполнении контрольных точек в журнале `/var/lib/pgsql/15/data/log/postgresql- Fri.log`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-10.jpg)



Разница между началом контрольной точки и её завершением 27 секунд

Этот параметр устанавливается в настройке `checkpoint_completion_target`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-11.jpg)



Задаёт целевое время для завершения процедуры контрольной точки, как коэффициент для общего времени между контрольными точками. 

0.9 * 30 секунд (checkpoint_timeout) = 27 секунд



***Сравнение tps в синхронном/асинхронном***



**Синхронный**


Запускаем тест 


```bash
pgbench -i postgres;
```


-c: количество одновременных клиентов или сеансов БД

-P: отображает прогресс и метрики каждые 60 секунд.

-T: запустит тест на 60 секунд (1 минут).


```bash
pgbench -c8 -P 60 -T 600 -U postgres postgres
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-12.jpg)



**Асинхронный**


Необходимо изменить показатель `synchronous_commit` в значение `off` в файле конфигурации `/var/lib/pgsql/15/data/postgresql.conf`


```bash
sudo nano /var/lib/pgsql/15/data/postgresql.conf
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-13.jpg)



Перезапуск `postgresql`


```bash
sudo systemctl restart postgresql-15.service
```


Запускаем тест 


```bash
pgbench -i postgres;
```


-c: количество одновременных клиентов или сеансов БД

-P: отображает прогресс и метрики каждые 60 секунд.

-T: запустит тест на 60 секунд (1 минут).


```bash
pgbench -c8 -P 60 -T 600 -U postgres postgres
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit8/8-14.jpg)



В результате tps в асинхронном режиме более чем в два раза превышают синхронный. Это следствие того, что серверу в асинхронном режиме не нужно ждать пока WAL записи сохранятся на диск.

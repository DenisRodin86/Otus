Домашнее задание 5



Виртуальная машина на `RedOS`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-1.jpg)



Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-2.jpg)



Установка утилиты для просмотра аппартаной конфигурации сервера

```bash
sudo dnf install lshw
```


Просмотр конфигурации сервера

```bash
sudo lshw -short
```


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-3.jpg)



С помощью `https://pgtune.leopard.in.ua` расчитываем корректные настройки `postgres`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-4.jpg)



Редактируем файл `postgres.conf` согласно этим рекомендациям



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-5.jpg)



Устанавливаем дополнительные компоненты для `pgbench`

```bash
sudo dnf install postgresql-contrib
```


Создаем БД

```bash
CREATE DATABASE benchmark;
```


Заполняем БД

```bash
pgbench  -U postgres -i -s 150 benchmark
```


Начинаем тетсирование

```bash
pgbench -U postgres -c 50 -j 2 -P 60 -T 600 benchmark
```

`-c: количество одновременных клиентов или сеансов БД
-j: количество рабочих потоков, которые pgbench будет использовать во время теста.
-P: отображает прогресс и метрики каждые 60 секунд.
-T: запустит тест на 600 секунд (10 минут).`


Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-6.jpg)



Сознательно превысим производительность сервера в файле `postgres.conf` и посмотрим как изменяться данные



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-7.jpg)




Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-8.jpg)



Теперь установим значение `postgres.conf` в изначальные параметры при установке

Результат



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-9.jpg)



`TPS` – это показатель пропускной способности базы данных, который показывает количество транзакций, обработанных базой данных за одну секунду.
Из результатов тестирования видно, что при повышенных настройках лучшие показатели `TPS`, это может быть связано с тем, что сервер расположен на `Hyper-V`, и в случае нехватки ресурсов (как в нашем случае из-за установки заранее завышенных показателей) система добавляет ресурсы виртуальной машине (конечно если есть свободные), чтобы она не "легла".


Попробуем установить "заоблачные" характеристики


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-10.jpg)



Результат


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit5/5-11.jpg)



Из результатов видно, что `TPS` "упал" (даже ниже чем default), т.к. неправильно установлены характеристики и `Hyper-V` не может выделить столько ресурсов.




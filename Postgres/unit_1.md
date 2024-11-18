Unit - 1



Виртуальная машина на RedOS 



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-1.jpg)




Добавление ssh ключа в metadata ВМ
Выполняется под пользователем
Включение входа по ключу:
mkdir ~/.ssh
chmod 0700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 0644 ~/.ssh/authorized_keys
nano /etc/ssh/sshd_config
PasswordAuthentication no  - отключение входа по логин/пароль




Удаленный сеанс ssh (первая сессия)




![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-2.jpg)





Установка postgres
sudo dnf install postgresql15-server
sudo postgresql-15-setup initdb
sudo systemctl enable postgresql-15.service --now
sudo systemctl status postgresql-15.service

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-3.jpg)
![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-4.jpg)






Вторая ssh сессия

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-5.jpg)





Запускаем psql из под пользователя postgres



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-6.jpg)



Выключаем auto commit
set autocommit=off




Работа с таблицами


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-7.jpg)


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/Postgres/1-8.jpg)


При добавлении записи в первой сессии я ее не вижу во второй, т.к. уровень изоляции  read committed
и она не видит "грязные данные" пока не будет commit;
По сути запрос SELECT видит снимок базы данных в момент начала выполнения запроса.


Новая транзакция repeatable read 
При добавлении записи в первой сессии я ее не вижу во второй, даже после commit;
Таким образом, последовательные команды SELECT в одной транзакции видят одни и те же данные; они не видят изменений, внесённых и зафиксированных другими транзакциями после начала их текущей транзакции.






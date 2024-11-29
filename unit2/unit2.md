Домашнее задание 2



Виртуальная машина на RedOS 



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-1.jpg)



Установка `Docker Engine`

```bash
dnf install docker-ce docker-ce-cli
systemctl enable docker --now
systemctl status docker
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-2.jpg)



Вывод команды `docker info`


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-3.jpg)



Добавление пользователя в группу для работы с `docker`
```bash
usermod -aG docker rodin
```


Поиск доступных `postgres docker`

```bash
sudo docker search postgres
```


Развертывание `docker postgres`

```bash
sudo docker pull postgres
```


Создание каталога для `postgres`

```bash
sudo mkdir /var/lib/postgresql/data
```


Запуск `docker`, проброс портов и создание учетной записи

```bash
sudo docker run -itd -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/data:/var/lib/postgresql/data --name postgresql -d postgres
```


Запуск `psql` в контейнере с `postgres`

```bash
sudo docker exec -it postgresql psql -U postgres
```

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-4.jpg)



Развертывание `docker` клиент `psql`

```bash
sudo docker pull nansencenter/postgres-client
```


Создаем тестовую таблицу

```bash
CREATE DATABASE test;
```


Подключение к контейнеру `postgres docker` со стороней машины 


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-5.jpg)


![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-6.jpg)




Удаление контейнера с сервером

```bash
sudo docker stop postgresql
sudo docker rm postgresql
```


Создаем `docker` заново

```bash
sudo docker run -itd -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/data:/var/lib/postgresql/data --name postgresql -d postgres
```


Проверяем, что данные остались на месте

![Postgers](https://github.com/DenisRodin86/Otus/blob/main/unit2/2-7.jpg)


Особых проблем с выполнением дз не было. Единственное, что ОС выполнения отличается от ОС лекции, что внесло коррективы в выполнения ДЗ.

















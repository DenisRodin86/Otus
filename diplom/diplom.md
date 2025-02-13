Дипломный проект

Схема сети



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-1.jpg)



Схема сети состоит из Prod (Продуктивного контура) и Test (Тестового контура)


Виртуальная машина на `RedOS`



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-2.jpg)



Производим обновление системы

```bash
dnf update
```


Устанавливаем `Postgresql 15`

```bash
  sudo dnf install postgresql15-server
	sudo postgresql-15-setup initdb
	sudo systemctl enable postgresql-15.service --now
	sudo systemctl status postgresql-15.service
```


Дополнительно устанавливаются компоненты

```bash
sudo dnf install postgresql15-contrib
```


Подключение к PostgreSQL через pgadmin4, а так же для взаимодействия между реплицированными серверами вносим изменения в файл postgresql.conf и pg_hba.conf

```bash
sudo nano /var/lib/pgsql/15/data/postgresql.conf
		Замените строку:# listen_addresses = 'localhost'
		На: listen_addresses = '*' 
	sudo nano /var/lib/pgsql/15/data/pg_hba.conf
		host all all 0.0.0.0/0 md5
```


Задаем пароль пользователю postgres 

```bash
ALTER USER postgres WITH ENCRYPTED PASSWORD '111111';
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```


****Настройка репликации****


Сервер 1 *Master-сервер*


Создание пользователя `replication` для репликации между серверами

```bash
CREATE ROLE replication WITH REPLICATION PASSWORD '111111' LOGIN;
```

В файле `pg_hba.conf` вносим строку подключения к *slave*

```bash
host    replication     replication     192.168.5.228/32         md5	
```


В файле `postgresql.conf` вносим изменения

```bash
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1024
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```



****Настройка доп. сервера (slave)****


В файле `pg_hba.conf` вносим строку подключения к master

```bash
host    replication     replication     192.168.5.223/32         md5	
```

В файле `postgresql.conf` вносим изменения

```bash
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1024
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```

Удаляем директорию установки `postgres`

```bash
rm -Rf /var/lib/pgsql/15/data/*
```

В директории `/var/lib/pgsql/15/`

```bash
rm -rf main; mkdir main; chmod go-rwx main
pg_basebackup -h IP -D /var/lib/pgsql/15/data -P -U replication --wal-method=stream
```

В файле `postgresql.conf` вносим изменения

```bash
hot_standby = on
primary_conninfo = 'user=replication password=111111 host=192.168.5.223 port=5432 sslmode=prefer sslcompression=1 krbsrvname=postgresql-15.service'
```

Выполняем процесс синхронизации

```bash
touch standby.signal
chown postgres:postgres standby.signal
```

Перезапускаем процесс `postgres`

```bash
sudo systemctl restart postgresql-15.service
```

Проверка работы репликации на `master` создаем `БД db1`

```bash
create database db1;
```

Создадим таблицу `table1` в этой БД

```bash
create table table1 (
id integer,
fio text
);
```

Заполнение таблицы `table1` данными

```bash
insert into table1 (id, fio) values (1, 'Иванов И'), (2, 'Родин Д'), (3, 'Семенов С');
```


Вывод таблицы 

```bash
select * from table1 ;
```



![Postgers](https://github.com/DenisRodin86/Otus/blob/main/diplom/d-3.jpg)



на *slave* выполним запрос к `table1` БД `db1`

```bash
select * from table1;
```


**Получаем аналогичный вывод.**

***Репликация работает.***




****Настройка сервера тестового контура.****

Сервер тестового контура будет забирать БД с *slave* сервера, чтобы не нагружать *master* сервер.

Для начала на сервере *slave* в файл `pg_hba.conf` внесем адрес тестового сервера

```bash
host    all     all     192.168.5.222/32         md5	
```

На самом сервере тестового контура так же внесем изменения

В файле `pg_hba.conf` вносим строку подключения к *slave*

```bash
host    all     all     192.168.5.228/32         md5
```

В файле  `postgresql.conf`

Замените строку `listen_addresses = 'localhost'`
На: `listen_addresses = '*' `

Скрипт обновления БД распологается в директории пользователя, в данном случае `/home/rodin/`

При выполнении процедуры резервного копирования дамп БД сохраняется в директорию `/backups/<имя продуктивного сервера>/`. В эту же директорию записывается лог выполнения процедуры. 

**Для выполнения скриптов прав администратора не требуется**. Хранение логинов и паролей для подключения к БД `PostgreSQL` осуществляется в файле `.pgpass`, который находится в домашней директории авторизованного пользователя.

При повторном выполнении скрипта предыдущие дампы БД заменяются текущими.

При выполнении процедуры резервного копирования/восстановления БД `PostgreSQL` схема и данные копируются по отдельности.

**Скрипт обновления БД**

```bash
#!/bin/bash

# === Конфигурация ===
SOURCE_HOST="<FQDN_source_server_name>"	# FQDN имя продуктивного сервера
SOURCE_PORT="5432"						          # Порт
SOURCE_USER="backup_user"				        # Учетная запись для бэкапирования
SOURCE_DB="db_name"						          # Имя бэкапируемой базы данных
SOURCE_SERVER="server_name"				      # host-name продуктивного сервера

TARGET_HOST="FQDN_target_server_name"	  # FQDN имя тестового сервера
TARGET_PORT="5432"						          # Порт
TARGET_USER="backup_user"				        # Учетная запись для восстановления
TARGET_DB="db_name"						          # Имя восстанавливаемой базы данных

BACKUP_DIR="/backups/${SOURCE_SERVER}"								# Расположение файлов
DATA_DUMP="${BACKUP_DIR}/${SOURCE_DB}_dump.sql"						# Файл бэкапа данных
ROLES_DUMP="${BACKUP_DIR}/${SOURCE_DB}_roles.sql"					# Файл бэкапа ролей
LOG_FILE="${BACKUP_DIR}/${SOURCE_DB}_$(date +%Y%m%d_%H%M%S).log"	# Лог файл

PGPASS_FILE="$HOME/.pgpass"				# Расположение файла с данными авторизации
export PGPASSFILE="$PGPASS_FILE"

# Логирование
exec > >(tee -a "$LOG_FILE") 
# exec 2>&1
echo "====== Начало выполнения: $(date) ======"

# Проверка наличия файла .pgpass
if [ ! -f "$PGPASS_FILE" ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка: файл $PGPASS_FILE не найден! Убедитесь, что он существует и правильно настроен."
  exit 1
fi

# Бэкап ролей
echo "$(date +%Y-%m-%d\ %T) Создание бэкапа ролей с сервера $SOURCE_HOST..."
pg_dumpall -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER --roles-only > $ROLES_DUMP 2>>$LOG_FILE
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка при бэкапе ролей!"
  exit 1
fi
echo "$(date +%Y-%m-%d\ %T) Бэкап ролей успешно создан: $ROLES_DUMP"

# Создание дампа базы данных с удалённого сервера
echo "$(date +%Y-%m-%d\ %T) Создание дампа базы данных $SOURCE_DB с сервера $SOURCE_HOST..."
pg_dump -h "$SOURCE_HOST" -p "$SOURCE_PORT" -U "$SOURCE_USER" -d "$SOURCE_DB" -Fc -f "$DATA_DUMP"
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка: не удалось создать дамп базы данных $SOURCE_DB с сервера $SOURCE_HOST!"
  exit 1
fi
echo "$(date +%Y-%m-%d\ %T) Дамп базы данных $SOURCE_DB успешно создан: $DATA_DUMP"

# Проверка подключения к целевому серверу
echo "$(date +%Y-%m-%d\ %T) Проверка подключения к серверу $TARGET_HOST..."
psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -c "\l" > /dev/null
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка: не удалось подключиться к серверу $TARGET_HOST!"
  exit 1
fi
echo "$(date +%Y-%m-%d\ %T) Подключение к серверу $TARGET_HOST успешно установлено."

# Проверка и удаление базы данных на сервере назначения
echo "$(date +%Y-%m-%d\ %T) Проверка существования базы данных $TARGET_DB на сервере $TARGET_HOST..."
DB_EXISTS=$(psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -tAc "SELECT 1 FROM pg_database WHERE datname = '$TARGET_DB';")
if [ "$DB_EXISTS" == "1" ]; then
  echo "$(date +%Y-%m-%d\ %T) База данных $TARGET_DB существует. Удаление..."
  psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '$TARGET_DB';" > /dev/null 2>&1
  psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -c "DROP DATABASE $TARGET_DB;" > /dev/null 2>&1
  if [ $? -ne 0 ]; then
    echo "$(date +%Y-%m-%d\ %T) Ошибка: не удалось удалить базу данных $TARGET_DB на сервере $TARGET_HOST!"
    exit 1
  fi
  echo "$(date +%Y-%m-%d\ %T) База данных $TARGET_DB успешно удалена."
else
  echo "$(date +%Y-%m-%d\ %T) База данных $TARGET_DB не существует."
fi

# Создание базы данных на сервере назначения
echo "$(date +%Y-%m-%d\ %T) Создание базы данных $TARGET_DB на сервере $TARGET_HOST..."
psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -c "CREATE DATABASE $TARGET_DB;" > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка: не удалось создать базу данных $TARGET_DB на сервере $TARGET_HOST!"
  exit 1
fi
echo "$(date +%Y-%m-%d\ %T) База данных $TARGET_DB успешно создана."

# Восстановление ролей 
echo "$(date +%Y-%m-%d\ %T) Восстановление ролей на сервере $TARGET_HOST..."
# Сохранение хэша текущего пароля postgres
CURRENT_PASSWORD=$(psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -tAc "SELECT rolpassword FROM pg_authid WHERE rolname = 'postgres';")
psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d postgres -f $ROLES_DUMP 2>>$LOG_FILE > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка при восстановлении ролей!"
  exit 1
fi

# Восстановление пароля postgres
if [ -n "$CURRENT_PASSWORD" ]; then
  psql -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d postgres -c "ALTER ROLE postgres WITH PASSWORD '$CURRENT_PASSWORD';" > /dev/null 2>&1
fi
echo "$(date +%Y-%m-%d\ %T) Роли успешно восстановлены."

# Восстановление базы данных на сервере назначения
echo "$(date +%Y-%m-%d\ %T) Восстановление базы данных $TARGET_DB на сервере $TARGET_HOST..."
pg_restore -h "$TARGET_HOST" -p "$TARGET_PORT" -U "$TARGET_USER" -d "$TARGET_DB" "$DATA_DUMP"
if [ $? -ne 0 ]; then
  echo "$(date +%Y-%m-%d\ %T) Ошибка: восстановление базы данных $TARGET_DB на сервере $TARGET_HOST завершилось неудачно!"
  exit 1
fi
echo "$(date +%Y-%m-%d\ %T) Восстановление базы данных $TARGET_DB успешно завершено."

echo "====== Завершение выполнения: $(date) ======"
```





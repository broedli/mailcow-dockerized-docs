MySQL is used for SOGo's and most of mailcow's settings.

## Connect

```
source mailcow.conf
docker-compose exec mysql-mailcow mysql -u${DBUSER} -p${DBPASS} ${DBNAME}
```

## Backup

```
cd /path/to/mailcow-dockerized
source mailcow.conf
DATE=$(date +"%Y%m%d_%H%M%S")
docker-compose exec mysql-mailcow mysqldump --default-character-set=utf8mb4 -u${DBUSER} -p${DBPASS} ${DBNAME} > backup_${DBNAME}_${DATE}.sql
```

## Restore

You should redirect the sql dump without Docker-Compose to prevent parsing errors.

```
cd /path/to/mailcow-dockerized
source mailcow.conf
docker exec -i $(docker-compose ps -q mysql-mailcow) mysql -u${DBUSER} -p${DBPASS} ${DBNAME} < backup_file.sql
```

## Reset MySQL passwords

Stop the stack by running `docker-compose stop`.

When the containers came to a stop, run this command:

```
docker-compose run --rm --entrypoint '/bin/sh -c "gosu mysql mysqld --skip-grant-tables & sleep 10 && mysql -hlocalhost -uroot && exit 0"' mysql-mailcow
```

### 1\. Find database name

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mailcow_database   | <=====
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```

### 2\. Reset one or more users

Both "password" and "authentication_string" exist. Currently "password" is used, but better set both.

```
MariaDB [(none)]> SELECT user FROM mysql.user;
+--------------+
| user         |
+--------------+
| mailcow_user | <=====
| root         |
+--------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> UPDATE mysql.user SET authentication_string = PASSWORD('gotr00t'), password = PASSWORD('gotr00t') WHERE User = 'root' AND Host = '%';
MariaDB [(none)]> UPDATE mysql.user SET authentication_string = PASSWORD('mookuh'), password = PASSWORD('mookuh') WHERE User = 'mailcow' AND Host = '%';
MariaDB [(none)]> FLUSH PRIVILEGES;
```
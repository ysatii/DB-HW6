# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - Мельник Юрий Александрович

## Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.


## Решение 1 
 
```
![рис 1](https://github.com/ysatii/DB-HW4/blob/main/img/image1.jpg)

Сумарно по всем таблицам 
```
### Ответить в свободной форме. 

## Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.
### Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.

## Решение 2
1. запуск контерйнеров
- master 
   docker run -d --name replication-master -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.3

- slave
   docker run -d --name replication-slave -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.3

Контейнеры в работе 
![рис 2](https://github.com/ysatii/DB-HW6/blob/main/img/image2.jpg)

2.  Создадим учетную запись master для сервера репликации:
  подлючимся к shell контейнера
```
docker exec -it replication-master mysql
```
создадим пользователя И дадим ему привелегии  
```
CREATE USER 'replication'@'%';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%'
```
Проверим права пользователя
```
SHOW GRANTS FOR replication@'%';
```
![рис 2_1](https://github.com/ysatii/DB-HW6/blob/main/img/image2_1.jpg)

Вносим измения в файл /etc/my.cnf
 секция [mysqld] добавляем следующие параметры:
```
server_id = 1  
log_bin = mysql-bin
```



3. перезагрузка replication-master  
```
sudo docker restart replication-master
``` 
![рис 2_3](https://github.com/ysatii/DB-HW6/blob/main/img/image2_3.jpg)

4. проверка стуса мастера
``` 
docker exec -it replication-master mysql
``` 

проверка статуса  
``` 
SHOW MASTER STATUS;
``` 

![рис 2_4](https://github.com/ysatii/DB-HW6/blob/main/img/image2_4.jpg)


Файл бинарного журнала mysql-bin.000001, позиция 158  
5. на slave Открываем конфигурационный файл на Slave /etc/my.cnf.  
``` 
log_bin = mysql-bin
server_id = 2
relay-log = /var/lib/mysql/mysql-relay-bin
relay-log-index = /var/lib/mysql/mysql-relay-bin.index
 read_only = 1
``` 
![рис 2_5](https://github.com/ysatii/DB-HW6/blob/main/img/image2_5.jpg)

6. Перезагружаем slave  
``` 
sudo docker restart replication-slave
``` 

7. подлючаемся
```
docker exec -it replication-slave mysql
```

добавляем настройки
```
CHANGE MASTER TO
MASTER_HOST='replication-master',
MASTER_USER='replication',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=158;
```

Запускаем репликацию  
```
mysql> START SLAVE; (или START REPLICA;)
mysql> SHOW SLAVE STATUS\G
```

![рис 2_6](https://github.com/ysatii/DB-HW6/blob/main/img/image2_6.jpg)

8. проверим репликацию  
на мастере
```
CREATE database world;
SHOW databases;
USE world;
CREATE TABLE `city` (
  `Name` varchar(50) NOT NULL,
  `CountryCode` varchar(50) NOT NULL,
  `District` varchar(50) NOT NULL,
  `Population` int NOT NULL  
) ENGINE=InnoDB AUTO_INCREMENT=601 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO city (Name, CountryCode, District, Population) VALUES
('Test-Replication', 'ALB', 'Test', 42);
```

на слайф
```
mysql> SHOW databases;
mysql> USE world;
mysql> SHOW tables;
SELECT * FROM city;
```

![рис 2_7](https://github.com/ysatii/DB-HW6/blob/main/img/image2_7.jpg)
![рис 2_8](https://github.com/ysatii/DB-HW6/blob/main/img/image2_8.jpg)

Базу и таблицы в ней создаем на мастер, выборку данных делаем на слайф, реапликация работает
 

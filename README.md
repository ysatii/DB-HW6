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

server_id = 1
log_bin = mysql-bin

![рис 2_2](https://github.com/ysatii/DB-HW6/blob/main/img/image2_2.jpg)

3. 
![рис 2_3](https://github.com/ysatii/DB-HW6/blob/main/img/image2_3.jpg)
![рис 2_4](https://github.com/ysatii/DB-HW6/blob/main/img/image2_4.jpg)
![рис 2_5](https://github.com/ysatii/DB-HW6/blob/main/img/image2_5.jpg)
![рис 2_6](https://github.com/ysatii/DB-HW6/blob/main/img/image2_6.jpg)
![рис 2_7](https://github.com/ysatii/DB-HW6/blob/main/img/image2_7.jpg)
![рис 2_8](https://github.com/ysatii/DB-HW6/blob/main/img/image2_8.jpg)
 

# Домашнее задание к занятию «Индексы» - Мельник Юрий Александрович

## База данных Sakila содержит 16 основных таблиц, описывающих различные аспекты компании по прокату DVD-дисков. Ниже приведен список этих таблиц:

   - actor – информация об актерах, которые участвовали в фильмах.
   - address – информация об адресах, в которых зарегистрированы клиенты компании.
   - category – список категорий, к которым могут относиться фильмы.
   - city – информация о городах, в которых расположены адреса клиентов.
   - country – информация о странах, в которых расположены города.
   - customer – информация о клиентах, которые берут в аренду фильмы.
   - film – информация о фильмах, доступных для проката.
   - film_actor – связь между фильмами и актерами, которые участвуют в этих фильмах.
   - film_category – связь между фильмами и категориями, к которым они относятся.
   - film_text – полный текст сюжетов фильмов, доступных для проката.
   - inventory – информация о DVD-дисках, доступных для проката.
   - language – список языков, на которых доступны фильмы.
   - payment – информация о платежах, сделанных клиентами за аренду фильмов.
   - rental – информация о факте аренды фильма клиентом.
   - staff – информация о сотрудниках компании, работающих в магазине.
   - store – информация о магазинах компании по прокату DVD-дисков.
## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.


## Решение 1 
(размера всех индексов /  размеру всех таблиц) * 100 
выберем только таблицы, без представлений, тип TABLE_TYPE = 'BASE TABLE'
 
размер всех индексов именно таблиц 
```
 select index_length from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE'
```
![рис 1](https://github.com/ysatii/DB-HW4/blob/main/img/image1.jpg)

Сумарно по всем таблицам 
```
 select sum(index_length) from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE'
```
![рис 1_1](https://github.com/ysatii/DB-HW4/blob/main/img/image1_1.jpg)

Место занимаемое таблицами
```
 SELECT data_length from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE'
```
![рис 1_2](https://github.com/ysatii/DB-HW5/blob/main/img/image1_2.jpg)

СУммарно по всем таблицам
```
 SELECT sum(data_length) from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE'
```
![рис 1_3](https://github.com/ysatii/DB-HW5/blob/main/img/image1_3.jpg)

Итоговый запрос
```
SELECT  ((select sum(index_length) from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE')
/(SELECT sum(data_length) from information_schema.tables WHERE table_schema = 'sakila' and TABLE_TYPE = 'BASE TABLE'))*100 as index_size_in_data_size  
 ```
![рис 1_4](https://github.com/ysatii/DB-HW5/blob/main/img/image1_4.jpg)

Процентное отношение всех индексов ко всем данным

```
SELECT  ((select sum(index_length) from information_schema.tables WHERE table_schema = 'sakila')
/(SELECT sum(data_length) from information_schema.tables WHERE table_schema = 'sakila'))*100 as index_size_in_data_size  
```
![рис 1_5](https://github.com/ysatii/DB-HW5/blob/main/img/image1_5.jpg)


## Задание 2
- Выполните explain analyze следующего запроса:
```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

## Решение 2
Возможные узкие места:
- отсутвие индексовтаблиц на поля используемые в запросы
- мы выбираем только сумму заказов, наличие поля f.title в запросе .. sum(p.amount) over (partition by c.customer_id, f.title) может замедлить запрос, также названия фильмов поле f.title мы не используем
- много условий в команде where

что можем сделать для ускорения запроса
- добавить индексы
- использовать операции join для ускорения

выполним запрос оценим его производительность
```
explain analyze
select distinct concat(c.last_name, ' ', c.first_name), t
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
![рис ](https://github.com/ysatii/DB-HW5/blob/main/img/image2.jpg)

используем join, уберем из запроса таблицы  inventory, film, данныые из них для запроса платежей не нужны



```
explain analyze
select distinct concat(c.last_name, ' ', c.first_name),   sum(p.amount) over (partition by c.customer_id  )
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
-- join inventory i on i.inventory_id = r.inventory_id
where  date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
```

![рис 2_1](https://github.com/ysatii/DB-HW5/blob/main/img/image2_1.jpg)

сравним оба рисунка, видим явное уменьшение времени выполнения  
например 

было Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=9936..9936 rows=391 loops=1)  
стало Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=27.5..27.5 rows=391 loops=1)  

было Temporary table with deduplication  (cost=0..0 rows=0) (actual time=9936..9936 rows=391 loops=1)  
стало Temporary table with deduplication  (cost=0..0 rows=0) (actual time=27.5..27.5 rows=391 loops=1)  

Выполним оба запроса, исначальный и исправленный
![рис 2_2](https://github.com/ysatii/DB-HW5/blob/main/img/image2_2.jpg)
![рис 2_3](https://github.com/ysatii/DB-HW5/blob/main/img/image2_3.jpg)

Выполним оба запроса, исначальный и исправленный
предложенный в задании запрос в среднем выполняеться не менее 8 секунд
исправленный 0,035 с. скорость обработки запроса увеличилась в боле чем 200 раз

попробуем увеличить скорость за счет индексов 
```
CREATE INDEX customer_rental_id_amount_payment_date ON payment(customer_id, rental_id, amount, payment_date);

CREATE INDEX idx_amount ON payment(amount);

CREATE INDEX customer_id_amount ON payment(customer_id, amount);
```
![рис 2_4](https://github.com/ysatii/DB-HW5/blob/main/img/image2_4.jpg)

Выполним 
```
explain analyze
select distinct concat(c.last_name, ' ', c.first_name),   sum(p.amount) over (partition by c.customer_id  )
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
-- join inventory i on i.inventory_id = r.inventory_id
where  date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
```
![рис 2_5](https://github.com/ysatii/DB-HW5/blob/main/img/image2_5.jpg)
![рис 2_5_1](https://github.com/ysatii/DB-HW5/blob/main/img/image2_5_1.jpg)

Среднее время не выполнения запроса сильно не изменилось, но при увеличении объема базы, использование индексов должно помощь
в быстром выполнении запросов


select  concat(c.last_name, ' ', c.first_name), sum(p.amount), c.customer_id
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
join inventory i on i.inventory_id = r.inventory_id
where  date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
group by c.customer_id;

среднее врменя выполнения дажет немного больше

![рис 2_6](https://github.com/ysatii/DB-HW5/blob/main/img/image2_6.jpg)

С использованием подзапросов  
```
explain analyze
select distinct concat(c.last_name, ' ', c.first_name),   sum(p2.amount) over (partition by c.customer_id  )
from  (select p.payment_id, p.customer_id, p.amount, p.payment_date
       from  payment p
        where  date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)) as p2
join rental r on p2.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
```
![рис 2_7](https://github.com/ysatii/DB-HW5/blob/main/img/image2_7.jpg)

Вероятно на малых объемах не возможно определеить кто из них выполняеться быстрее

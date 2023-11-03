# Домашнее задание к занятию "`Индексы`" - `Мурчин Артем`

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение 1

![alt text](https://github.com/artmur1/12-05-hw/blob/main/12-05-zad1.png)

SELECT SUM(data_length), SUM(index_length), (SUM(index_length)*100/SUM(data_length)) AS 'index_length in %'

FROM INFORMATION_SCHEMA.TABLES;

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение 2

![alt text](https://github.com/artmur1/12-05-hw/blob/main/12-05-zad2-1.png)

На скриншоте выделил узкие места в запросе. Видно, что стоимости запросов запредельно высоки, время выполнения длительное.

![alt text](https://github.com/artmur1/12-05-hw/blob/main/12-05-zad2-2.png)

С помошью оператора LEFT JOIN удалось снизить стоимость запроса и время на выполнение.

##### текст запроса

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)

from rental r

LEFT JOIN customer c ON c.customer_id  = r.customer_id  

LEFT JOIN payment p ON p.rental_id  = r.rental_id 

LEFT JOIN inventory i ON i.inventory_id = r.inventory_id 

LEFT JOIN film f ON f.film_id = i.film_id 

where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id;

#### Доработка задания 2

![alt text](https://github.com/artmur1/12-05-hw/blob/main/12-05-zad2-dor1.png)

Создал индекс pay_date

CREATE INDEX pay_date ON payment(payment_date, amount);

![alt text](https://github.com/artmur1/12-05-hw/blob/main/12-05-zad2-dor2.png)

Видно, что стоимость запросов уменьшилась в 10 раз. Также видно, что запрос обращается к созданному индексу pay_date.

explain analyze

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)

from rental r

INNER JOIN customer c ON c.customer_id  = r.customer_id  

INNER JOIN payment p ON p.rental_id  = r.rental_id 

INNER JOIN inventory i ON i.inventory_id = r.inventory_id 

INNER JOIN film f ON f.film_id = i.film_id 

where payment_date >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY);

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

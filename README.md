# Домашнее задание к занятию "`Индексы`" - `Бызгаев Александр`

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение:

```sql
select sum(data_length) as SUM_Data_Length, sum(index_length) as SUM_Index_Length, sum(index_length)*100.0/sum(data_length) as Persentage_ratio
from information_schema.tables
where table_schema='sakila' and data_length is not null;
```

![image](https://github.com/Byzgaev-I/Index/blob/main/Index%20-1.png)

---

### Задание 2

Выполните explain analyze следующего запроса:

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

### Решение:

Конечно пергуглил кучу всего и немного в голове каша от количества инфы и нового, но получилось так:

Вывод запроса:

![image](https://github.com/Byzgaev-I/Index/blob/main/Index%20-2.png)

Я могу использовать EXPLAIN ANALYZE для нашего запроса:

```sql
EXPLAIN ANALYZE
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name),
       SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE DATE(p.payment_date) = '2005-07-30';

```
После выполнения этого запроса в нашей среде можно увидеть подробный план выполнения и статистику, которая поможет идентифицировать узкие места.  

Потенциальные узкие места могут включать:  

 - Использование функции **DATE()** на **p.payment_date**, что может препятствовать использованию индексов.  
 - Возможное отсутствие индексов на ключевых столбцах для соединений и фильтрации.
 - Отсутствие явных JOIN: Использование списка таблиц разделенного запятыми и условий в WHERE является устаревшей практикой. Вместо этого следует использовать явные JOIN-ы.  
 - Отсутствие индексов: Если индексы на соединяемые столбцы отсутствуют, это может значительно замедлить выполнение запроса.  
  
Оптимизация запроса:

```sql
WHERE p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00'
```

Убедиться, что на следующих столбцах есть индексы:  

- payment.payment_date
- rental.rental_date
- customer.customer_id
- inventory.inventory_id
- film.film_id

Использовать явные JOIN-ы для лучшей читаемости и потенциальной оптимизации со стороны оптимизатора запросов.

После внесения этих изменений, запрос выглядит так:

```sql
EXPLAIN ANALYZE
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name),
       SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00';

```

---































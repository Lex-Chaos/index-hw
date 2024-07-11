# Домашняя работа к занятию «Индексы» - Боровик А. А.

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Ответ:

```SQL
SELECT  SUM(INDEX_LENGTH)/SUM(DATA_LENGTH)*100 "INDEX / TABLE %"
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = "sakila" AND TABLE_TYPE = "BASE TABLE"
;
```

![Задание 1. Ответ](https://github.com/Lex-Chaos/index-hw/blob/master/img/index_Task_1.png)

---

### Задание 2

Выполните explain analyze следующего запроса:

```sql
select
 distinct concat(c.last_name, ' ', c.first_name),
 sum(p.amount) over (partition by c.customer_id, f.title)
from
 payment p,
 rental r,
 customer c,
 inventory i,
 film f
where date(p.payment_date) = '2005-07-30'
 and p.payment_date = r.rental_date
 and r.customer_id = c.customer_id
 and i.inventory_id = r.inventory_id
```

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Ответ:

Результат выполнения запроса:

![Задание 2. Результат запроса](https://github.com/Lex-Chaos/SQL-2-hw/blob/master/img/SQL2_Task_2.1.png)

Результат выполнения EXPLAIN ANALYZE:

![Задание 2. Результат EXPLAIN ANALYZE](https://github.com/Lex-Chaos/SQL-2-hw/blob/master/img/SQL2_Task_2.1.png)

Узкие места — большие вложенные циклы, в которых проверяются все возможные совпадения во всех перечисленных таблицах.

Вместо проверки всех возможных совпадений предлагаю объединение таблиц по совпадающим значениям столбцов и выбирать нужные данные уже из этой, составной таблицы.

```sql
SELECT
 DISTINCT CONCAT(c.last_name, ' ', c.first_name) "Customer's name",
 SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) "Amount summ"
FROM
 payment p
 INNER JOIN rental r on r.rental_date = p.payment_date
 LEFT JOIN inventory i on i.inventory_id = r.rental_id
 LEFT JOIN film f on f.film_id = i.inventory_id
 LEFT JOIN customer c on c.customer_id = r.customer_id
WHERE
 DATE(p.payment_date) = '2005-07-30'
;
```

Результат выполнения запроса:

![Задание 2. Результат оптимизированного запроса](https://github.com/Lex-Chaos/SQL-2-hw/blob/master/img/SQL2_Task_2.3.png)

Результат выполнения EXPLAIN ANALYZE:

![Задание 2. Результат EXPLAIN ANALYZE](https://github.com/Lex-Chaos/SQL-2-hw/blob/master/img/SQL2_Task_2.4.png)

---

### Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

_Приведите ответ в свободной форме._

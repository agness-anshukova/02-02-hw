# Домашнее задание к занятию "`Домашнее задание к занятию «Индексы»`" - `Аншукова Ания`


### Задание 1
## Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц. 
Запрос.
`SELECT (sum(t.INDEX_LENGTH)/sum(t.DATA_LENGTH))*100`
`FROM information_schema.tables as t`
`WHERE table_schema = 'sakila';`

![Запрос индексов](/img/indexes.jpg)


### Задание 2
## Выполните explain analyze следующего запроса:
Запрос на получение данных.
`select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)`
`from payment p, rental r, customer c, inventory i, film f`
`where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id`

![Explain analize](/img/explain_analize.jpg)

## перечислите узкие места;
Самый дорогая по стоимости операция `Sort: c.customer_id, f.title  (actual time=4866..5043 rows=642000 loops=1)`.
Полагаю, что стоимость операции обусловлена соединением таблицы `film` с соединением из таблиц `payment p, rental r, customer c, inventory i`, поскольку не указано, по какому столбцу таблицы `film` следует ее соединять с другими таблицами, получили декартово произведение.

## оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
Запрос был преобразован следующим образом:
`select distinct concat(c.last_name, ' ', c.first_name), `
     ` sum(p.amount) over (partition by c.customer_id, f.title)`
 ` from payment p,` 
     `rental r,`   
       `customer c,` 
       `inventory i, `
       `film f `
  `where date(p.payment_date) = '2005-07-30'` 
    `and p.payment_date = r.rental_date `
    `and r.customer_id = c.customer_id `
    `and i.inventory_id = r.inventory_id`
   ` and i.film_id = f.film_id;`

Стоимость запроса без дополнительных индексов:
![Explain analize 2](/img/costs_without_ind.jpg)

Стоимость запроса уменьшилась при добавлении доп.индексов:
`CREATE INDEX rci ON rental (rental_date, customer_id, inventory_id );`
`CREATE INDEX lfc ON customer (last_name, first_name, customer_id ); `
`CREATE INDEX fti ON film (film_id, title ); `
`CREATE INDEX inv ON inventory (inventory_id, film_id ); `
![Explain analize 3](/img/explain_analize1.jpg)
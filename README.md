# Домашнее задание к занятию "`Индексы`" - `Дунаев Дмитрий`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

---

### Решение 1

Запрос и вывод ниже:

```SQL
MySQL [(none)]> SELECT TABLE_SCHEMA AS 'Table schema',
    -> SUM(`DATA_LENGTH`) AS 'Data volume',
    -> SUM(`INDEX_LENGTH`) AS 'Indexes volume',
    -> SUM(`INDEX_LENGTH`)/(SUM(`DATA_LENGTH`)+SUM(`INDEX_LENGTH`))*100 AS 'Percentage ratio'
    -> FROM information_schema.TABLES
    -> WHERE `TABLE_SCHEMA` = "sakila";
+--------------+-------------+----------------+------------------+
| Table schema | Data volume | Indexes volume | Percentage ratio |
+--------------+-------------+----------------+------------------+
| sakila       |     4374528 |        2392064 |          35.3511 |
+--------------+-------------+----------------+------------------+
1 row in set (0,005 sec)
```

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

---

### Решение 2

1. Сделаем EXPLAIN ANALYZE:

```SQL
MySQL [sakila]> EXPLAIN ANALYZE
    -> select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
    -> from payment p, rental r, customer c, inventory i, film f
    -> where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id;
...
| -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=2794..2794 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=2794..2794 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1137..2702 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=1137..1165 rows=642000 loops=1)
                -> Stream results  (cost=22.1e+6 rows=16.7e+6) (actual time=0.311..822 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=22.1e+6 rows=16.7e+6) (actual time=0.306..707 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=20.5e+6 rows=16.7e+6) (actual time=0.303..617 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=18.8e+6 rows=16.7e+6) (actual time=0.298..526 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.285..20.2 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.72 rows=16500) (actual time=0.026..2.68 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.72 rows=16500) (actual time=0.0183..2.01 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=0.0481..0.197 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.938 rows=1.01) (actual time=513e-6..726e-6 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=58.3e-6..71.1e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=55.5e-6..68.5e-6 rows=1 loops=642000)
 |
1 row in set (2,800 sec)
```

Видим, что выполнение занимает внушительные 2,8 секунд.

2. Выполним сам запрос, и посмотрим результат, которыый он выдает, чтобы оценить его:

```SQL
...
391 rows in set (2,549 sec)
```

3. В ходе анализа запроса и результата, предполагаю, что некоторые таблицы не используется, можно удалить их из запроса. Конструкцию в конце, сочетающую множество условий AND, тоже удаляем, это существенно сократит время выполнения и объем самого запроса. Также делаю JOIN для таблицы 'customer'.
Запуск после оптимизации дал результат:

```SQL
...
MySQL [sakila]> EXPLAIN ANALYZE
    -> select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
    -> from payment p
    -> LEFT JOIN customer c ON c.customer_id = p.customer_id
    -> where date(p.payment_date) >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY) 
    -> GROUP BY c.customer_id;
...
| -> Sort with duplicate removal: `concat(c.last_name, ' ', c.first_name)`, `sum(p.amount)`  (actual time=5.9..5.91 rows=391 loops=1)
    -> Table scan on <temporary>  (actual time=5.78..5.8 rows=391 loops=1)
        -> Aggregate using temporary table  (actual time=5.78..5.78 rows=391 loops=1)
            -> Nested loop left join  (cost=3599 rows=5499) (actual time=0.197..5.27 rows=634 loops=1)
                -> Filter: ((cast(p.payment_date as date) >= '2005-07-30') and (p.payment_date < <cache>(('2005-07-30' + interval 1 day))))  (cost=1674 rows=5499) (actual time=0.162..4.69 rows=634 loops=1)
                    -> Table scan on p  (cost=1674 rows=16500) (actual time=0.119..3.43 rows=16044 loops=1)
                -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=755e-6..777e-6 rows=1 loops=634)
 |

1 row in set (0,009 sec)
```

4. Для ускорения поиска добавляю индекс для столбца 'payment_date', в результате получаем:

```SQL
MySQL [sakila]> CREATE INDEX pdate ON payment(payment_date);
Query OK, 0 rows affected (0,087 sec)
Records: 0  Duplicates: 0  Warnings: 0

MySQL [sakila]> EXPLAIN ANALYZE
    -> select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
    -> from payment p
    -> LEFT JOIN customer c ON c.customer_id = p.customer_id
    -> where date(p.payment_date) >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY) 
    -> GROUP BY c.customer_id;                                                           |
...
| -> Sort with duplicate removal: `concat(c.last_name, ' ', c.first_name)`, `sum(p.amount)`  (actual time=3.54..3.55 rows=391 loops=1)
    -> Table scan on <temporary>  (actual time=3.42..3.45 rows=391 loops=1)
        -> Aggregate using temporary table  (actual time=3.42..3.42 rows=391 loops=1)
            -> Nested loop left join  (cost=4998 rows=9497) (actual time=0.0718..3.21 rows=634 loops=1)
                -> Filter: ((cast(p.payment_date as date) >= '2005-07-30') and (p.payment_date < <cache>(('2005-07-30' + interval 1 day))))  (cost=1674 rows=9497) (actual time=0.0594..2.93 rows=634 loops=1)
                    -> Table scan on p  (cost=1674 rows=16500) (actual time=0.0448..2.11 rows=16044 loops=1)
                -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=359e-6..372e-6 rows=1 loops=634)
 |
1 row in set (0,006 sec)
```

Если сделать "EXPLAIN FORMAT = TRAITIONAL", видно упоминание индекса:

```SQL
MySQL [sakila]> EXPLAIN FORMAT = TRADITIONAL
    -> select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
    -> from payment p
    -> LEFT JOIN customer c ON c.customer_id = p.customer_id
    -> where date(p.payment_date) >= '2005-07-30' and payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY) 
    -> GROUP BY c.customer_id;
+----+-------------+-------+------------+--------+---------------------------------------------------------+---------+---------+----------------------+-------+----------+------------------------------+
| id | select_type | table | partitions | type   | possible_keys                                           | key     | key_len | ref                  | rows  | filtered | Extra                        |
+----+-------------+-------+------------+--------+---------------------------------------------------------+---------+---------+----------------------+-------+----------+------------------------------+
|  1 | SIMPLE      | p     | NULL       | ALL    | pdate                                                   | NULL    | NULL    | NULL                 | 16500 |    57.56 | Using where; Using temporary |
|  1 | SIMPLE      | c     | NULL       | eq_ref | PRIMARY,idx_fk_store_id,idx_fk_address_id,idx_last_name | PRIMARY | 2       | sakila.p.customer_id |     1 |   100.00 | NULL                         |
+----+-------------+-------+------------+--------+---------------------------------------------------------+---------+---------+----------------------+-------+----------+------------------------------+
2 rows in set, 1 warning (0,003 sec)
```


---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

---

### Решение 3*

В отличие от MySQL в PostgreSQL есть следующие типы индексов:

* Bitmap index,
* Inverted index,
* Partial index,
* Function based index.

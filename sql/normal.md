## [Тестовое Альфа-банк] Покупки телефонов в Туле по месяцам

*Необходимо вывести количество людей из Тулы, которые покупали телефоны с разбивкой по месяцам*

```sql
	select  
	  trim( to_char(created_at, 'month') ) as month, 
	  count(distinct id_customer) as people
	from purchases p
	join customer c on p.user_id = c.id_customer
	where sku_id in (select id from skus where category = 2)
	 and town = 'Tula'
	group by 1
	order by 2 desc 
```
	
## Среднемесячное количество продаж больше 2

*Вывести ID анализов, у которых среднемесячное количество заказов больше 2 в 2020 году*

```sql 
	select 
	  ord_an as an_id,
	  round(1.0 * count(1) / 12, 3) as cnt
	from orders o
	where to_char(ord_datetime, 'yyyy')::int = 2020
	group by 1
	having round(1.0 * count(1) / 12, 3) > 2
	order by 2 desc, 1 
```

## Продажи за 2019 и 2020 год

*Для каждого анализа вывести: ID анализа, кол-во продаж за 2019 год и кол-во продаж за 2020 год*

```sql
	select 
	  ord_an as an,
	  count(case when to_char(ord_datetime, 'yyyy')::int = 2019 then 1 else null end) as year2019,
	  count(case when to_char(ord_datetime, 'yyyy')::int = 2020 then 1 else null end) as year2020
	from orders
	group by 1
	order by 1
```

## Помесячный прирост продаж с разбивкой по группе

*Нарастающим итогом рассчитать, как увеличивалось количество проданных тестов каждый месяц каждого года с разбивкой по группе*

```sql
	with t as (
		select
		  to_char(ord_datetime, 'yyyy') as year,
		  to_char(ord_datetime, 'mm') as month,
		  gr_id as g, 
		  count(o.*) as cnt
		from orders o 
		join analysis a on o.ord_an = a.an_id
		join groups g on a.an_group = g.gr_id
		group by 1, 2, 3
		)
	select 
	  year, 
	  month, 
	  g as group,
	  sum(cnt) over (partition by g, year order by year, month rows between unbounded preceding and current row)::int as sum
	from t
	order by 3, 1, 2 
```

## Прирост продаж для анализов с условием

*Помесячно вывести прирост количества продаж в процентах относительно предыдущего месяца для всех анализов в 2020 году, где в названии в любом месте располагается слово "кров" или "тестостерон" в любом регистре*

```sql
	with t as (
		select
		  an_name as name,
		  to_char(ord_datetime, 'mm') as month,
		  count(o.*) as current_cnt,
		  lag(count(o.*)) over (partition by an_name order by to_char(ord_datetime, 'mm')) as prev_cnt
		from analysis a
		join orders o on o.ord_an = a.an_id
		where to_char(ord_datetime, 'yyyy')::int = 2020 
		and (an_name ~~* '%кров%' or an_name ~~* '%тестостерон%')
		group by 1, 2
		order by 1, 2
		)
	select 
	  name,
	  month,
	  current_cnt,
	  prev_cnt as prev_cnt,
	  coalesce(round(100.0 * (current_cnt - prev_cnt) / prev_cnt, 3), 0) as change
	from t
	order by name, month 
```
	
## Проранжировать продажи по группам в разрезе лет

*Для каждого года в отдельности проранжировать продажи по группам анализов. Если для каких-то групп в рамках одного года продажи сопадают, то им присваивать одинаковое место. Ранги должны идти без разрывов*

```sql
	select
	  an_group as id,
	  to_char(ord_datetime, 'yyyy') as year,
	  count(o.*) as cnt,
	  dense_rank() over (partition by to_char(ord_datetime, 'yyyy') order by count(o.*) desc) as rnk
	from orders o join analysis a on o.ord_an = a.an_id
	group by 1, 2
	order by 2, 4, 1
```

## Проверить падение продаж 3 года подряд

*Помесячно проверить: если суммарное количество продаж в течение трех месяцев подряд падают, то вывести в столбец Result 1, иначе 0*

```sql
	with t as (
		select
		  to_char(ord_datetime, 'yyyy') as year,
		  to_char(ord_datetime, 'mm') as month,
		  lag(count(1), 2) over(order by to_char(ord_datetime, 'yyyy'), to_char(ord_datetime, 'mm')) as prev2_cnt,
		  lag(count(1)) over(order by to_char(ord_datetime, 'yyyy'), to_char(ord_datetime, 'mm')) as prev_cnt,
		  count(1) as cur_cnt
		from orders
		group by 1, 2
		order by 1, 2
		)
	select 
	  *,
	  case when prev_cnt < prev2_cnt and cur_cnt < prev_cnt then 1 else 0 end as result
	from t
```
	
## Изучить самую дорогую и самую дешевую позицию

*Для каждого дня найти самую дорогую и самую дешевую позицию в нем и посчитать среднюю стоимость самых дорогих и среднюю стоимость самых дешевых позиций*

```sql	
	with t as (
		select 
		  ord_datetime::date,
		  max(an_price) as ma, 
		  min(an_price) as mi
		from orders o 
		join analysis a on o.ord_an = a.an_id
		group by 1
		order by 1
		)
	select 
	  avg(mi) as avg_cheap, 
	  avg(ma) as avg_exp
	from t
```

## Найти ID с самым большим количеством заказов по годам

*Выберите ID товара с самым большим количеством заказов с разбивкой по годам*

```sql
	select 
	  year, 
	  an_id, 
	  cnt 
	from 
	   (select 
		 to_char(ord_datetime, 'yyyy') as year,
		 ord_an as an_id,
		 count(1) as cnt,
		 dense_rank() over (partition by to_char(ord_datetime, 'yyyy') order by count(1) desc) as rnk
		from orders o
		group by 1, 2
		order by 1, 4
		) foo
	where rnk = 1
	order by 1, 2
```

## Третий анализ по количеству продаж

*Вывести третий анализ по количеству продаж за весь период*

```sql
	with t as (
		select
		  an_id,
		  an_name,
		  count(ord_id) as cnt,
		  dense_rank() over (order by count(ord_id) desc) rn
		from orders o
		join analysis a on o.ord_an = a.an_id
		group by 1, 2
		)
	select * from t 
	where rn = 3
```
	
## Расположение студентов после пересадки

*Было принято решение поменять студентов, которые сидят рядом, местами. Напишите SELECT-запрос, который из исходной таблицы сформирует расположение студентов после пересадки*

```sql
	with t as (
		select 
		  seat,
		  name,
		  ntile(8) over(order by seat) as n
		from students s
		),
	t2 as (
		select 
		  seat, 
		  name, 
		  n,
		  rank() over (partition by n order by seat desc) as rnk
		from t
		order by n, rnk
		)
	select 
	  row_number() over() as seat, 
	  name
	from t2 
```
	
## Не менее трех значений подряд

*Найти все Num, которые встречаются не менее трех раз подряд. Результат выведите в столбце ConsecutiveNums*
*Числа выведите без повторов - чтобы одно число в столбце ConsecutiveNums встречалось только 1 раз*

```sql
	with t as (
		select
		  id,
		  num,
		  count(num) over(partition by num) as cnt
		from logs l
		) 
	select distinct num as ConsecutiveNums from t
	where cnt >= 3
	order by 1 desc
```
	
## Вторая по размеру зарплата

*Вывести 2 по размеру зарплату*

```sql
	select 
	  salary 
	from 
	  (select 
	    salary,
		rank() over (order by salary desc) as rnk
	   from employee
	  ) foo
	where rnk = 2
```
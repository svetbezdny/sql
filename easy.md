## Покупки товара после 10 октября 2021

*Необходимо вывести количество людей, которые покупали товар с id = 5 после 10 октября 2021 (включительно)*

```sql
	select 
	  count(distinct id_customer)
	from customer c
	join purchases p on c.id_customer = p.user_id
	where sku_id = 5
	and created_at >= '2021-10-10' 
```
	
## TOP ↑ | Продажи анализов в течение недели

*Для всех анализов, которые продавались 5 февраля 2020 и всю следующую неделю, вывести:*
- *an_name*
- *an_cost*
- *ord_datetime*

```sql 
	select 
	  an_name,
	  an_cost,
	  ord_datetime
	from analysis a
	join orders o on a.an_id = o.ord_an
	where ord_datetime between '2020-02-05' and '2020-02-05'::date + interval '7 days'
	order by 3, 2 
```

## Анализы, которые не продавались за период

*Вывести название всех анализов, которые не продавались с 1 по 3 мая 2020 года*

```sql
	select 
	  distinct an_name
	from analysis
	where an_id not in 
		(
		select 
		  ord_an 
		from orders 
		where ord_datetime between '2020-05-01' and '2020-05-03'
		) 
```

## Анализы в диапазоне цен

*Вывести все анализы, которые в рознице стоят от 250 до 500 рублей*

```sql
	select 
	  an_id,
	  an_name,
	  an_price
	from analysis a
	where an_price between 250 and 500 
```

## Продажи в апреле 2020

*Вывести все номера заказов, которые были совершены в апреле 2020 года*

```sql
	select 
	  ord_id,
	  ord_datetime
	from orders
	where to_char(ord_datetime, 'mm-yyyy') = '04-2020'
	order by 2 
```
	
## Режим хранения анализов

*Вывести все заказы, которые содержат анализы, предполагающие режим хранения 22*

```sql
	select 
	  ord_id,
	  ord_an,
	  an_name,
	  gr_temp
	from orders o
	join analysis a on o.ord_an = a.an_id
	join groups g on a.an_group = g.gr_id
	where gr_temp = 22 
```

## Анализ крови

*Найти все уникальные названия анализов, которые фигурировали в заказах и имеют в названии любую форму слова кровь*

```sql
	select 
	  distinct an_name
	from analysis a
	join orders o on a.an_id = ord_an
	and an_name ~~* '%кровь%'
```
	
## Задача на расчет наценки

*Рассчитать и вывести наценку для каждой строки таблицы Заказов. Рассчитанную наценку округлите до 3 знака после запятой и выразите в % (то есть не 0.01, а 1)*

```sql	
	select 
	  ord_id,
	  ord_datetime,
	  ord_an,
	round(100.0 * (an_price - an_cost) / an_cost, 3) as markup
	from analysis a
	join orders o on a.an_id = o.ord_an 
```

## Анализы в апреле 2018

*Отобрать все анализы, которые были заказаны в апреле 2018 года*

```sql
	select 
	  distinct an_name
	from analysis a
	where an_id = any 
		(
		select 
		  ord_an 
		from orders 
		where to_char(ord_datetime, 'mm-yyyy') = '04-2018'
		) 
```

## Средняя наценка по годам

*Вывести из таблицы Заказов информацию по средней наценке с разбивкой по годам*

```sql
	select
	  to_char(ord_datetime, 'yyyy')::int as year,
	  avg(1.0 * (an_price - an_cost) / an_cost) as Mean_markup
	from orders o
	join analysis a on ord_an = a.an_id
	group by 1
	order by 1 
```
	
## Округлить и отфильтровать наценку

*Округлите наценку на каждую позицию из таблицы Заказов до ближайшего десятка. Выберите только те строки, где наценка больше 50%*

```sql
	select
	  ord_id,
	  ord_datetime,
	  ord_an,
	  round(100.0 * (an_price - an_cost) / an_cost, -1) as markup
	from orders o
	join analysis a on o.ord_an = a.an_id
	where round(100.0 * (an_price - an_cost) / an_cost, -1) > 50 
```
	
## Найти анализы из заданных категорий

*Найти анализы с наценкой выше 35% или из 7 категории. Если какие-то анализы относятся сразу к двум категориям, то их необходимо оставить*

```sql
	select 
	  *, 
	  round(100.0 * (an_price - an_cost) / an_cost, 3) as charge
	from analysis
	where 100.0 * (an_price - an_cost) / an_cost > 35
	union all
	select 
	  *, 
	  round(100.0 * (an_price - an_cost) / an_cost, 3) as charge
	from analysis
	where an_group = 7 
```
	
## Самый продаваемый анализ

*Найти анализ, который продавался больше всего раз за весь период времени*

```sql
	select
	  ord_an as an_id,
	  count(ord_id) as cnt
	from orders
	group by 1
	order by 2 desc
	limit 1 
```
	
## Отфильтрованные продажи в период дат

*Найти количество продаж с разбивкой по датам в период дат с '2019-10-25' до '2019-11-02'. Оставить только те строки, где количество заказов больше 5*

```sql
	select * from 
		(
		select 
		  ord_datetime,
		  count(ord_id) as cnt
		from orders o
		where cast( ord_datetime as date ) between '2019-10-25' and '2019-11-02'
		group by 1
		order by 2 desc
		) foo
	where cnt > 5
	order by ord_datetime
```
	
## Нарастающий итог продаж

*Нарастающим итогом вывести увеличение общей суммы продаж с каждым новым заказом в порядке увеличения ID*

```sql
	select 
	  ord_datetime::date as dt,
	  ord_id,
	  sum(an_price) over(order by ord_id rows between unbounded preceding and current row) as cumsum
	from analysis a
	join orders o on a.an_id = o.ord_an
	order by 2 
```
	
## Нумерация строк по дате

*В рамках каждого дня пронумеруйте строки, опираясь на дату и время заказа (по убыванию). Для каждого нового дня нумерация должна начинаться заново*

```sql
	select 
	  ord_datetime::date as dt,
	  ord_datetime,
	  ord_id,
	  ord_an,
	  row_number() over(partition by ord_datetime::date order by ord_datetime) as rn
	from orders o
	order by 1, 2 
```
	
## Анализы с нечетным ID и заданной себестоимостью

*Выведите анализы с нечетным ID, у которых себестоимость больше 100*

```sql
	select 
	  an_id,
	  an_name,
	  an_cost
	from analysis a
	where an_id % 2 > 0
	and an_cost > 100 
```
	
## Группы и температурный режим анализов

*Напишите запрос, в котором для каждого анализа будет выведены название группы и нужный температурный режим*

```sql
	select 
	  an_id,
	  an_name,
	  gr_name,
	  gr_temp
	from analysis a 
	left join groups g on a.an_group = g.gr_id 
```
	
## Сотрудники с большой зарплатой

*Необходимо вывести имена всех сотрудников, у которых зарплата больше, чем у их менеджеров*

```sql
	select 
	  emp as employee 
	from 
		(select 
		  e.name as emp, 
		  e.salary as emp_sal,
		  m.name as man, 
		  m.salary as man_sal
		  from employee e
		join employee m on m.id = e.managerid
		) foo
  where emp_sal > man_sal 
```
  
## Анализы, которые никогда не заказывались

*Выведите ID и названия всех анализов, которые никогда не заказывались*

```sql
	select
	  an_id,
	  an_name
	from analysis a
	where an_id not in ( select ord_an from orders o ) 
```
	
## Рост продаж по сравнению с предыдущим днем

*Необходимо вывести информацию о продажах за те даты, когда продажи были больше, чем за предыдущий день из таблицы sales*

```sql
	with t as 
		( 
		select
		  date,
		  sum(value) as value,
		  coalesce(lag(sum(value)) over (order by date), 0) as lg
		from sales s
		group by 1 
		)
	select * from t
	where value > lg
	order by date 
```

## Проранжировать баллы

*Нужно проранжировать строки по убыванию поля Score. Если значения совпадают, то нужно присвоить одинаковый ранг. Разрывов в нумерации быть не должно*

```sql
	select
	  score,
	  dense_rank() over (order by score desc) as rnk
	from scores 
```

## Найти округа, в которых не было покупок

*Необходимо найти названия всех округов, жители которых никогда не совершали покупки в этом магазине*

```sql
	select
	  distinct co.name
	from county co
	left join customer cs using(county_code)
	left join c_orders c_o using(id_customer)
	where c_o.id_customer is null 
```
	
## Снизить цену для отмеченных товаров

*Написать SELECT-запрос для уменьшения цены на 20% для тех продуктов в таблице STORE, которые содержат пометку note = DISCOUNT*

```sql
	select
	  s.*,
	  case when note ~~* 'discount' then unit_price * (1 - 0.2) else unit_price end as price_with_discount
	from store s 
```
	
## Найти все пары различных клиентов

*Найти все пары различных клиентов (поле FIRST_NAME) из города Moscow*

```sql
	select
	  c1.first_name as customer1,
	  c2.first_name as customer2
	from customer c1 join customer c2 on c1.first_name != c2.first_name
	where c1.town = 'Moscow' and c2.town = 'Moscow' 
```
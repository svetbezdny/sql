## Средняя стоимость 5 покупки

*Необходимо вывести среднюю стоимость 5-ой покупки с разбивкой по городам*

```sql
	with t as (
		select 
		  user_id, 
		  price,
		  row_number() over(partition by user_id order by created_at) r
		from purchases p
		join skus s on p.sku_id = s.id
		)
	select 
	  town,
	  avg(price) as avg_price_5th_purchase
	from customer c join t on c.id_customer = t.user_id
	where t.r = 5
	group by 1
	order by 2 desc 
```

|  town  | avg_price_5th_purchase|   
|--------|-----------------------|
| London | 		976.0            |         
| Moscow | 		976.0            |        
| Kaluga | 		605.5            |         
| Tula   | 		351.0            |
	
## Сделать из «широкой» таблицы «длинную»

*Как из таблицы WideTable получить LongTable?*

- **LongTable:**

|  Name   |   key  |        value          |
|---------|--------|-----------------------|
| Ivanov  |   FIO  |  Иванов Иван Иванович |
| Ivanov  | Phone  |  +(7) 111-1111111     |
| Ivanov  | Email  | ivanov@ivanov.com     |
| Petrov  |   FIO  | Петров Петр Петрович  |
| Petrov  | Phone  |  +(7) 222-2222222     |
| Petrov  | Email  | petrov@petrov.com     |


- **WideTable:**

|   Name  |         FIO          |      Phone       |       Email       |
|---------|----------------------|------------------|-------------------|
| Ivanov  | Иванов Иван Иванович | +(7) 111-1111111 | ivanov@ivanov.com |
| Petrov  | Петров Петр Петрович | +(7) 222-2222222 | petrov@petrov.com |


```sql 
	select 
	  name,
	  unnest(array['FIO', 'Phone', 'Email']) as key,
	  unnest(array[fio, phone, email]) as value
	from WideTable
	order by 1, 2, 3
```

## Сделать из «длинной» таблицы «широкую»

*Как из таблицы LongTable получить WideTable? Предполагается чтение таблицы один раз и отсутствие соединений*

- **LongTable:**


|  Name   |   key  |        value          |
|---------|--------|-----------------------|
| Ivanov  |   FIO  |  Иванов Иван Иванович |
| Ivanov  | Phone  |  +(7) 111-1111111     |
| Ivanov  | Email  | ivanov@ivanov.com     |
| Petrov  |   FIO  | Петров Петр Петрович  |
| Petrov  | Phone  |  +(7) 222-2222222     |
| Petrov  | Email  | petrov@petrov.com     |


- **WideTable:**


|   Name  |         FIO          |      Phone       |       Email       |
|---------|----------------------|------------------|-------------------|
| Ivanov  | Иванов Иван Иванович | +(7) 111-1111111 | ivanov@ivanov.com |
| Petrov  | Петров Петр Петрович | +(7) 222-2222222 | petrov@petrov.com |


```sql
	select 
	  name,
	  max(case when key = 'FIO' then value else null end) as FIO,
	  max(case when key = 'Phone' then value else null end) as Phone,
	  max(case when key = 'Email' then value else null end) as Email
	from LongTable
	group by 1
	order by 1
```

## Перемножить последовательность чисел

*Задача состоит в том, чтобы перемножить все значения между собой*


|  a  |
|-----|
|  1  |
|  2  |
|  3  |
|  4  |
|  6  |
|  7  |
|  10 |
|  11 |
|  12 |
|  13 |
|  15 |
|  17 |


```sql
	select 
	  round( exp( sum( ln( a ) ) ) ) as multiplier
	from numbers
```

|  multiplier |     
|-------------|
|4410806400.0 |

## Интервалы подряд идущих чисел

*Задача состоит в том, чтобы вернуть интервалы подряд идущих чисел*


|  a  |
|-----|
|  1  |
|  2  |
|  3  |
|  4  |
|  6  |
|  7  |
|  10 |
|  11 |
|  12 |
|  13 |
|  15 |
|  17 |


```sql
	with t as (
		select
		  a,
		  dense_rank() over(order by a) as rnk,
		  a - dense_rank() over(order by a) as diff
		from numbers
		)
	select 
	  min(a) as start,
	  max(a) as end
	from t
	group by diff
	order by 1, 2
```


| start | end |
|-------|-----|
|   1   |  4  |
|   6   |  7  |
|  10   | 13  |
|  15   | 15  |
|  17   | 17  |

	
## Фиксация входа-выхода сотрудников

*Дана таблица фиксации прохода через турникет gate*


|  employee  |   check_time  |  is_entered  |


*Известно, что за каждым входом следует выход. В начале дня всегда вход, последний - всегда выход*
*Необходимо вывести сотрудников и дни, когда они находились на рабочем месте менее 8 часов*

```sql
	with t_in as (
		select
		  employee,
		  check_time
		from gate 
		where is_entered = true
		),
	t_out as (
		select
		  employee,
		  check_time
		from gate 
		where is_entered = false
		),
	final as (
		select 
		  t_in.employee as employee,
		  t_in.check_time::date as check_date,
		  extract(epoch from t_out.check_time) - extract(epoch from t_in.check_time) as time_diff,
		  t_out.check_time - t_in.check_time as sum
		from t_in
		join t_out on t_in.employee = t_out.employee 
		 and t_in.check_time::date = t_out.check_time::date
		)
	select 
	  employee,
	  check_date,
	  sum
	from final
	where time_diff < 8 * 60 * 60
	order by sum
```

|employee |   check_date |   sum  |      
|---------|--------------|--------|
|Бердяев  |  2020-11-25  | 4:00:00|   
|Воронин  |  2020-11-25  | 7:59:00|
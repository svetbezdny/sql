*Write an SQL query to find employee_id of all employees that directly or indirectly report their work to the head of the company.*
*The indirect relation between managers will not exceed three managers as the company is small.*
*Return the result table in any order.*

1. **Create an employee table**

```sql
	with t as (
	 select *
	 from (
		values
		(1, 'Boss', 1),
		(3, 'Alice', 3),
		(2, 'Bob', 1),
		(4, 'Daniel', 2),
		(7, 'Luis', 4),
		(8, 'Jhon', 3),
		(9, 'Angela', 8),
		(77, 'Robert', 1)
		) employees (employee_id, employee_name, manager_id)
	)
	 select * 
	 into employees 
	 from t;
```

|employee_id | employee_name | manager_id |
|------------|---------------|------------|
|	  1	     |    Boss		 |	   1	  |
|	  3		 |	  Alice		 |	   3	  |
|	  2		 |	  Bob		 |	   1	  |
|	  4		 |	  Daniel	 |	   2	  |
|	  7		 |	  Luis		 |	   4	  |
|	  8		 |	  Jhon		 |	   3	  |
|	  9		 |	  Angela	 |	   8	  |
|	  77	 |	  Robert	 |	   1	  |

2. **Solution using RECURSIVE CTE**

```sql
	with recursive t(emp_name, emp_id, "level") as 
	(
	 select employee_name, employee_id, 0 as "level"
	 from employees
	 where employee_id in (1, 3)
	 	union all
	 select emp_name || ' => ' || e.employee_name, e.employee_id, "level" + 1
	 from employees e, t
	 where t.emp_id = e.manager_id and e.employee_id not in (1, 3)
	)
	select 
	  emp_name as work_heirarchy,
	  "level"
	from t;
```

|         work_heirarchy         | level |   
|------------------------------- |-------|
|Boss	                         |   0   |
|Alice	                         |   0   |
|Boss  => Bob	                 |   1   |
|Alice => Jhon	                 |   1   |
|Boss  => Robert	             |   1   |
|Boss  => Bob  => Daniel	     |   2   |
|Alice => Jhon => Angela	     |   2   |
|Boss  => Bob  => Daniel => Luis |   3   |

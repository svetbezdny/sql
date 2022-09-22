[stratascratch.com](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=1)  

*Calculate each user's average session time.*  
*A session is defined as the time difference between a page_load and page_exit.*  
*For simplicity, assume a user has only 1 session per day and if there are multiple of the same events on that day, consider only the latest page_load and earliest page_exit.*  
*Output the user_id and their average session time.*  
  
**Solution**  

```sql
with t as (
	select 
	  user_id,
	  to_char(timestamp, 'yyyy-mm-dd') as date,
	  max(case when action = 'page_load' then timestamp else null end) as page_load,
	  max(case when action = 'page_exit' then timestamp else null end) as page_exit
	from facebook_web_log
	group by 1, 2
	)
	select 
	  user_id,
	  avg(page_exit - page_load) as mean
	from t
	where page_exit is not null
	group by 1;
```

|  user_id  |   mean  |   
|-----------|---------|
|	  0	  	|  1883.5 |
|     1	  	|   35    |
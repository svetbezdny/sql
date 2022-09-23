[stratascratch.com](https://platform.stratascratch.com/coding/10284-popularity-percentage?code_type=1)  

*Find the popularity percentage for each user on Meta/Facebook.*  
*The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform, then converted into a percentage by multiplying by 100.*  
*Output each user along with their popularity percentage. Order records in ascending order by user id.*  
*The 'user1' and 'user2' column are pairs of friends.*   

1. **Create the facebook_friends table**

```sql
with foo as (
	select * from (
		values 
		  (2, 1), (1, 3), (4, 1), 
		  (1, 5), (1, 6), (2, 6),
		  (7, 2),(8, 3), (3, 9)
	)t (user1, user2)
)
select * into facebook_friends from foo;
```

2. **Solution**

```sql
with t as (
	select user1 as users, count(1) as cnt from facebook_friends group by 1
		union all
	select user2 as users, count(1) as cnt from facebook_friends group by 1
)
	select 
	  users,
	  100.0 * sum(cnt) / (select count(1) from facebook_friends) as percentage
	from t
	group by 1
	order by 1;
```
  
|  users  | percentage |   
|---------|------------|
|    1	  |  55.556    |
|    2	  |  33.333    |
|    3	  |  33.333	   |
|    4	  |  11.111	   |
|    5	  |  11.111	   |
|    6	  |  22.222	   |
|    7	  |  11.111	   |
|    8	  |  11.111	   |
|    9	  |  11.111	   |
 
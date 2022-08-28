[leetcode.com](https://leetcode.com/problems/tree-node/)  

*Each node in the tree can be one of three types:*  
**- "Leaf"**: *if the node is a leaf node.*  
**- "Root"**: *if the node is the root of the tree.*  
**- "Inner"**: *If the node is neither a leaf node nor a root node.*    

*Write an SQL query to report the type of each node in the tree.*  

![Tree node](https://assets.leetcode.com/uploads/2021/10/22/tree1.jpg?raw=true "Tree Node")     

1. **Create a "tree" table**

```sql
	with t as (
	 select *
	 from (
		values
		(1, null),
		(2, 1),
		(3, 1),
		(4, 2),
		(5, 2)
		) tree (id, p_id)
	)
	 select * 
	 into tree 
	 from t;
```

|     id     |    p_id   |
|------------|-----------|
|	  1	     |   null	 |
|	  2		 |	  1		 |
|	  3		 |	  1		 |
|	  4		 |	  2	     |
|	  5		 |	  2		 |


2. **Solution**

```sql
	select 
	 id,
	 case
	   when p_id is null then 'Root'
	   when id in (select p_id from tree) then 'Inner'
	   else 'Leaf'
	    end as type
	from tree;
```

|     id     |    p_id   |
|------------|-----------|
|	  1	     |   Root	 |
|	  2		 |	 Inner	 |
|	  3		 |	 Leaf	 |
|	  4		 |	 Leaf	 |
|	  5		 |	 Leaf	 |

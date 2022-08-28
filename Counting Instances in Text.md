[stratascratch.com](https://platform.stratascratch.com/coding/9814-counting-instances-in-text?code_type=1)  

*Find the number of times the words 'bull' and 'bear' occur in the contents.*  
*We're counting the number of times the words occur so words like 'bullish' should not be included in our count.*  
*Output the word 'bull' and 'bear' along with the corresponding number of occurrences.*  

1. **Create a google_file_store table**

```sql
with t as (
	select * from (
		values
		 ('draft1.txt', 'The stock exchange predicts a bull market which would make many investors happy'),
		 ('draft2.txt', 'The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear mark'),
		 ('final.txt', 'The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market. As always predicting the future market is an uncertain game and all investors should follow their instincts and best practices')
		) google_file_store (filename, contents)
	)
	select * 
	 into google_file_store 
	from t;
```

2. **Solution with *regexp_matches***

```sql
with t as (
	select 
	  regexp_matches(contents, 'bull', 'g') as bull,
	  regexp_matches(contents, 'bear', 'g') as bear
	from google_file_store
	)
	select 
	  unnest(array['bull', 'bear']) as word,
	  unnest(array[count(bull), count(bear)]) as cnt
	from t;
```

|  word   | cnt |   
|---------|-----|
|bull	  |  3  |
|bear	  |  2  |
# My SQL Jounrey 

## Problem # 1
### Problem Statement
Find the Intials of all names in a table separated by '.'

#### Sample Table

| Name  | Initials  |
|:----- |:---------:| 
| ATHU     | A | 
| ATHULYA G KANDY      |A.G.K      |           $12 |
| JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN |J.V.P.V  |
| JAYAKRISHNAN VIJAYARAGHAVAN |J.V  |


### Solution
My proposed solution is as follows:
	
1. Create a NAMES table which contains the given names
2. Count the number of parts in each name
3. Repeat each name the part number of times
4. Extract the first letter of each part based on part number
5. Group the partial inital in different rows into a single row and join by delimiter

	
#### STEP 1: Creating the NAMES table
Using a simple `UNION ALL`, we can create a table with four records. Code below: 

	WITH NAMES AS (
		SELECT 'ATHULYA G KANDY' AS NAME
		UNION ALL
		SELECT 'JAYAKRISHNAN VIJAYARAGHAVAN' AS NAME
		UNION ALL
		SELECT 'JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN' AS NAME
		UNION ALL
		SELECT 'ATHU' AS NAME
	)
	SELECT * FROM NAMES;
The above code creates the first column of the `Sample Table`. 
#### STEP 2: Count the NAME parts
In this step, we'd like to create the number of parts in each names. Part is a portion of the name, when separated by the delimiter. In our case space(`' '`) is the delimiter. 

	
	...
	, PARTS AS (
	SELECT
		NAME,
		LENGTH(NAME) - 
		LENGTH(REPLACE(NAME,' ','')) + 1 AS NUM_PARTS
	FROM
		NAMES
	)
	SELECT * FROM PARTS
	
Here the idea is that, when we subtract the length of the actual name from the length of the named with spaces removed, and then add 1, we get the `NUM_PARTS`. After executing this this is what we get. 


|NAME|	NUM_PARTS|
|:--|-----|
|ATHULYA G KANDY|	3|
|JAYAKRISHNAN VIJAY|	2|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN|	4|
|ATHU|	1|

#### STEP 3: Repeating Each Name, `Part` Number of times
##### STEP 3.a: Creating Sequencing Table
To repeat anything `n` number of times in MySQL, we need a sequence table; a table which has numbers 1 to n. Now some databases have this built-in, but if not, no need to worry; we can build our own. 

The naïve aproach of building this would be to use the `UNION ALL` for each number we want. Something like: 

	SELECT 1 val
	UNION ALL
	SELECT 2 val
	UNION ALL
	...
	SELECT <n> val

But a smart approach would be use the power of exponents and `CROSS JOIN`

	WITH SEQ_GEN_0_15 AS (
		SELECT pow0.val as Two_Power_0,
		pow1.val as Two_Power_1, 
		pow2.val as Two_Power_2, 
		pow3.val as Two_Power_3,
		pow0.val + pow1.val + pow2.val + pow3.val AS Sum
		FROM 
			(SELECT 0 val UNION ALL SELECT 1 val) pow0 CROSS JOIN 
			(SELECT 0 val UNION ALL SELECT 2 val) pow1 CROSS JOIN 
			(SELECT 0 val UNION ALL SELECT 4 val) pow2 CROSS JOIN 
			(SELECT 0 val UNION ALL SELECT 8 val) pow3
	)
	SELECT * FROM SEQ_GEN_0_15 WHERE Sum > 0;
	
The above code generates a table like this: 

| Two-power-0 | Two-power-1 | Two-power-2 | Two-power-3 | Sum |
|:--:|:--:|:--:|:--:|:--:|
|1|0|0|0|1|
|0|2|0|0|2|
|1|2|0|0|3|
|0|0|4|0|4|
|1|0|4|0|5|
|0|2|4|0|6|
|1|2|4|0|7|
|0|0|0|8|8|
|1|0|0|8|9|
|0|2|0|8|10|
|1|2|0|8|11|
|0|0|4|8|12|
|1|0|4|8|13|
|0|2|4|8|14|
|1|2|4|8|15|
The idea behind this is that any sequence from number `0` to `n - 1` can be created by using the sum of the `exponents of 2` from `0` to `n/2`; each term multipled by either 0 or 1

e.g 
```15 = 1(2^4) + 1(2^3) + 1(2^2) + 1(2^1) + 1(2^0)```

- Thus, using the numbers 0, 1, 2, 4, 8 and its combinations, I can generate sequences upto 15. 
- Using 0, 2, 4, 8, 16 and its combinations I can generate sequences upto 31  
- Using 2^0, 2^1, 2^2, 2^3, 2^4, .. , 2^n and its combinations, I can generate sequences upto *2n - 1*

#### STEP 3.b. Repeating Names
Repeating each name `NUM_PARTS` number of times just involves performing a cross join with the sequence table, where the sequence is between _1_ and *NUM_PARTS*
	
	...,
	REP_PARTS AS (
		SELECT
			a.NAME,
			b.ID
		FROM
			PARTS a,
			SEQ_GEN_0_15 b
		WHERE
			b.ID BETWEEN 1
			AND a.NUM_PARTS
	)
	SELECT * FROM REP_PARTS

The result of the above SQL looks like this: 

|NAME	|ID|
|:--:	|:--:|
|ATHULYA G KANDY	|1|
|ATHULYA G KANDY	|2|
|ATHULYA G KANDY	|3|
|JAYAKRISHNAN VIJAYARAGHAVAN	|1|
|JAYAKRISHNAN VIJAYARAGHAVAN	|2|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|1|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|2|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|3|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|4|
|ATHU	|1|

#### STEP 4: Extract the First Letter of Each Name

To extract the first letter of each part we will be using two functions:

1. SUBSTRING_INDEX
2. SUBSTR

- `SUBSTRING_INDEX(str, delim, count)` >> returns the substring from the given string before a specified number of occurrences of a delimiter.
- `SUBSTR(string, start, length)` >> extracts a substring from a string (starting at any position).

The following line of code will extract the initial from each part of the name based in the part_number coded as _ID_. 

	SUBSTR(substring_index(substring_index(NAME, ' ', ID), ' ', - 1), 1, 1)

When fully implemented the code looks like this:
	
	..., REP_PARTS AS (
		SELECT
			a.NAME,
			b.ID, 
			SUBSTR(substring_index(substring_index(NAME, ' ', ID), ' ', - 1), 1, 1) part_initial 
		FROM
			PARTS a,
			SEQ_GEN_0_15 b
		WHERE
			b.ID BETWEEN 1
			AND a.NUM_PARTS
	)
	select * from REP_PARTS;
	
The output of the table looks like this:

|NAME	|ID|	part_initial|
|:--:	|:--:|	:--:|
|ATHULYA G KANDY	|1|	A|
|ATHULYA G KANDY	|2|	G|
|ATHULYA G KANDY	|3|	K|
|JAYAKRISHNAN VIJAYARAGHAVAN	|1|	J|
|JAYAKRISHNAN VIJAYARAGHAVAN	|2|	V|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|1|	J|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|2|	V|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|3|	P|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|4|	V|
|ATHU	|1|	A|

#### STEP 5: Group Concatenate

In this stage, we'd like to group the partial inital in different rows into a single row and join by delimiter; the delimiter being `.` 

THe following lines of codes accomplishes that: 

	...
	SELECT
		NAME, 
		GROUP_CONCAT(
			part_initial
			ORDER BY ID 
			SEPARATOR '.'
			) AS INITIAL
	FROM
		REP_PARTS
	GROUP BY 1
	ORDER BY NAME

And the final result looks like this: 

|NAME	|initial|
|:--:	|:--:|
|ATHU	|A|
|ATHULYA G KANDY	|A.G.K|
|JAYAKRISHNAN VATTAPARAMBBIL PATTATH VIJAYARAGHAVAN	|J.V.P.V|
|JAYAKRISHNAN VIJAYARAGHAVAN	|J.V|

Happy MySQL :) 

****




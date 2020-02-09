# SQL Problems
## Nth Highest Salary
	CREATE FUNCTION getNthHighestSalary (N INT) RETURNS INT
	BEGIN
		DECLARE M INT;
		SET M = N - 1;
		RETURN ( SELECT DISTINCT
				SAL
			FROM
				EMP
			ORDER BY
				1 DESC
			LIMIT 1 OFFSET M);
	END
	
	SELECT getNthHighestSalary (5);
	
	
## Hierarchy of Managers
	SET GLOBAL log_bin_trust_function_creators = 1;

	CREATE FUNCTION getHierarchOfBosses (EMPID INT) RETURNS VARCHAR (100)
	BEGIN
	
	
		RETURN(
			with RECURSIVE EmployeeCTE as 
				(
					-- ANCHOR
					
					SELECT EMPNO, ENAME, MGR
					FROM EMP 
					WHERE EMPNO = EMPID
					
					UNION ALL 
					
					-- RECURSIVE PART 
					
					SELECT EMP.EMPNO, EMP.ENAME, EMP.MGR
					FROM EMP
					JOIN EmployeeCTE
					ON EMP.EMPNO = EmployeeCTE.MGR				
				)
			SELECT GROUP_CONCAT(ENAME SEPARATOR ' --> ') hierarchy_manager
			from EmployeeCTE
		);
	END
		
	select getHierarchOfBosses(7934) as 'Boss of Bosses';

### Results: 

|Boss of Bosses|
|:-------:|
|MILLER --> CLARK --> KING|

## Human Traffic of Stadium
[Leetcode URL
](https://leetcode.com/problems/human-traffic-of-stadium/)

## Problem
X city built a new stadium, each day many people visit it and the stats are saved as these columns: id, visit_date, people

Please write a query to display the records which have 3 or more consecutive rows and the amount of people more than 100(inclusive).

For example, the table stadium:


| id   | visit_date | people    |
|:------|------------+-----------|
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
For the sample data above, the output is:

| id   | visit_date | people    |
|:------|------------|-----------|
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
### Note:
Each day only have one row record, and the dates are increasing with id increasing.

## Solution

	with date_rolling_sum as 
	(
	    SELECT 
	        id,
	        visit_date, 
	        people as today,
	        LEAD(people, 1, 0) OVER (ORDER BY id) tom,
	        LEAD(people, 2, 0) OVER (ORDER BY id) day_after, 
	        LAG(people, 1, 0) OVER (ORDER BY id) yest,
	        LAG(people, 2, 0) OVER (ORDER BY id) day_before
	    from stadium
	)
	select 
	    id,
	    visit_date, 
	    today as people
	from date_rolling_sum
	where 
	(today >= 100 and tom >= 100 and day_after >= 100)  or
	(today >= 100 and yest >= 100 and day_before >= 100)  or
	(today >= 100 and yest >= 100 and tom >= 100)

#Pivot Occupations
## Problem
Pivot the Occupation column in OCCUPATIONS so that each Name is sorted alphabetically and displayed underneath its corresponding Occupation. The output column headers should be Doctor, Professor, Singer, and Actor, respectively.

Note: Print NULL when there are no more names corresponding to an occupation.

### Input Format

The OCCUPATIONS table is described as follows:

![](https://s3.amazonaws.com/hr-challenge-images/12889/1443816414-2a465532e7-1.png) 

Occupation will only contain one of the following values: Doctor, Professor, Singer or Actor.

**Sample Input**

![](https://s3.amazonaws.com/hr-challenge-images/12890/1443817648-1b2b8add45-2.png) 

**Sample Output**

| Doctor  | professors | Actors    |Actors|
|:------|------|--------|-----------:|
|Jenny  |  Ashley  |   Meera | Jane|
|Samantha| Christeen | Priya | Julia|
|NULL  |   Ketty | NULL |Maria |

**Explanation**

- The first column is an alphabetically ordered list of Doctor names.
- The second column is an alphabetically ordered list of Professor names.
-  The third column is an alphabetically ordered list of Singer names.
- The fourth column is an alphabetically ordered list of Actor names.
- The empty cell data for columns with less than the maximum number of names per occupation (in this case, the Professor and Actor columns) are filled with NULL values.

## Solution

		WITH master
		     AS (SELECT Rank ()
		                  OVER(
		                    partition BY occupation
		                    ORDER BY NAME) AS ROW_NUM,
		                NAME,
		                occupation
		         FROM   occupations),
		     doctors
		     AS (SELECT row_num,
		                NAME
		         FROM   master
		         WHERE  occupation = 'Doctor'),
		     actors
		     AS (SELECT row_num,
		                NAME
		         FROM   master
		         WHERE  occupation = 'Actor'),
		     professor
		     AS (SELECT row_num,
		                NAME
		         FROM   master
		         WHERE  occupation = 'Professor'),
		     singer
		     AS (SELECT row_num,
		                NAME
		         FROM   master
		         WHERE  occupation = 'Singer'),
		     max_rn
		     AS (SELECT Max(row_num) AS max_rn
		         FROM   master),
		     sequence
		     AS (SELECT e.row_num
		         FROM   (
		         select pow1.val + pow2.val + pow3.val+ pow4.val + pow5.val  as ROW_NUM, d.max_rn  
	            from 
		            ( SELECT 0 as val UNION ALL SELECT 1 as val) as pow1, 
		            ( SELECT 0 as val UNION ALL SELECT 2 as val ) as pow2 , 
		            ( SELECT 0 as val UNION ALL SELECT 4 as val ) as pow3,
		            ( SELECT 0 as val UNION ALL SELECT 8 as val ) as pow4,
		            ( SELECT 0 as val UNION ALL SELECT 16 as val ) as pow5, 
		            max_rn d
	            ) e
		         WHERE  e.row_num BETWEEN 1 AND max_rn)
		SELECT d.NAME AS doctor,
		       p.NAME AS prof,
		       s.NAME AS singer,
		       a.NAME AS actor
		FROM   sequence t1
		       LEFT JOIN professor p
		              ON t1.row_num = p.row_num
		       LEFT JOIN doctors d
		              ON t1.row_num = d.row_num
		       LEFT JOIN actors a
		              ON t1.row_num = a.row_num
		       LEFT JOIN singer s
		              ON t1.row_num = s.row_num
		ORDER  BY t1.row_num;  
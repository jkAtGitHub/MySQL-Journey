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

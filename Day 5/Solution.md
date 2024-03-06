## Here is data
```SQL
drop table if exists salary;
create table salary
(
	emp_id		int,
	emp_name	varchar(30),
	base_salary	int
);
insert into salary values(1, 'Rohan', 5000);
insert into salary values(2, 'Alex', 6000);
insert into salary values(3, 'Maryam', 7000);


drop table if exists income;
create table income
(
	id			int,
	income		varchar(20),
	percentage	int
);
insert into income values(1,'Basic', 100);
insert into income values(2,'Allowance', 4);
insert into income values(3,'Others', 6);


drop table if exists deduction;
create table deduction
(
	id			int,
	deduction	varchar(20),
	percentage	int
);
insert into deduction values(1,'Insurance', 5);
insert into deduction values(2,'Health', 6);
insert into deduction values(3,'House', 4);
```
-----
## Solution to create emp_transaction table
```SQL
drop table if exists emp_transaction;
create table emp_transaction
(
	emp_id		int,
	emp_name	varchar(50),
	trns_type	varchar(20),
	amount		numeric
);
INSERT INTO emp_transaction (emp_id, emp_name, trns_type, amount)
SELECT
    s.emp_id,
    s.emp_name,
    t.trns_type,
    CASE
        WHEN t.trns_type IN (t.trns_type) THEN ROUND(s.base_salary * (t.percentage / 100.0), 2)
    END AS amount
FROM
    salary s
CROSS JOIN (
    SELECT income AS trns_type, percentage FROM income
    UNION ALL
    SELECT deduction AS trns_type, percentage FROM deduction
) AS t;


select * from salary;
select * from income;
select * from deduction;
select * from emp_transaction;
```
---
## Solution to Create Salary report 

### First Way
```SQL 

SELECT 
    emp_name,
    SUM(CASE WHEN trns_type = 'Basic' THEN amount ELSE 0 END) AS Basic,
    SUM(CASE WHEN trns_type = 'Allowance' THEN amount ELSE 0 END) AS Allowance,
    SUM(CASE WHEN trns_type = 'Others' THEN amount ELSE 0 END) AS Others,
    SUM(CASE WHEN trns_type IN ('Basic', 'Allowance', 'Others') THEN amount ELSE 0 END) AS gross,
    SUM(CASE WHEN trns_type = 'insurance' THEN amount ELSE 0 END) AS insurance,
    SUM(CASE WHEN trns_type = 'Health' THEN amount ELSE 0 END) AS Health,
    SUM(CASE WHEN trns_type = 'House' THEN amount ELSE 0 END) AS House,
    SUM(CASE WHEN trns_type IN ('insurance', 'Health', 'House') THEN amount ELSE 0 END) AS total_deduction
FROM 
    emp_transaction
GROUP BY 
    emp_name;

```
### Second way (Microsoft SQL Server )

```SQL
with sl_rprt as
              (SELECT * FROM   
              (
                  SELECT 
                      emp_name, 
                      trns_type,
                      amount
                  FROM 
                      emp_transaction p
              ) t 
              PIVOT(
                  SUM(amount) 
                  FOR trns_type IN (
                      [Basic],
                      [Allowance], 
                      [Others], 
                      [insurance], 
                      [Health], 
                      [House])
              ) AS pivot_table
)

SELECT 
emp_name,
Basic,
Allowance,
Others,
(Basic + Allowance + Others) as gross,
insurance,
Health,
House,
(insurance+ Health + House) as total_deduction
from sl_rprt
```

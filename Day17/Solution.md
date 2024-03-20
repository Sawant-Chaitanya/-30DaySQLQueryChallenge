```SQL

-- Solution in MSSQL Server
with cte as 
	(select * 
	, dateadd(day, -dense_rank() over(partition by user_id order by user_id,login_date), login_date) as date_group
    from user_login)
select user_id
, min(login_date) as start_date
, max(login_date) as end_date
--, count(1) as consecutive_days
, (datediff(day, min(login_date), max(login_date)) + 1) as consecutive_days
from cte
group by user_id, date_group
having count(1) >= 5 
	and datediff(day, min(login_date), max(login_date)) >= 4
order by user_id, date_group;
```

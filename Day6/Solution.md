## Data
```SQL
drop table if exists  student_tests;
create table student_tests
(
	test_id		int,
	marks		int
);
insert into student_tests values(100, 55);
insert into student_tests values(101, 55);
insert into student_tests values(102, 60);
insert into student_tests values(103, 58);
insert into student_tests values(104, 40);
insert into student_tests values(105, 50);

select * from student_tests;


```
---

## Solution
```SQL
---Output 1
with p AS (
    SELECT *,
        LAG(marks, 1, 0) OVER (ORDER BY test_id) AS prv_mrk
    FROM student_tests
)
SELECT  
test_id,
marks
FROM  p
WHERE prv_mrk < marks


--- Output 2
with p AS (
    SELECT *,
        LAG(marks, 1) OVER (ORDER BY test_id) AS prv_mrk
    FROM student_tests
)
SELECT  
test_id,
marks
FROM  p
WHERE prv_mrk < marks
```

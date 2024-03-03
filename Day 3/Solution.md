## Question:

Write a SQL query to return the footer values from the input table, meaning all the last non-null values from each field as shown in the expected output.

## Data Table Create Query:

```sql
CREATE TABLE FOOTER 
(
    id INT PRIMARY KEY,
    car VARCHAR(20), 
    length INT, 
    width INT, 
    height INT
);

INSERT INTO FOOTER VALUES (1, 'Hyundai Tucson', 15, 6, NULL);
INSERT INTO FOOTER VALUES (2, NULL, NULL, NULL, 20);
INSERT INTO FOOTER VALUES (3, NULL, 12, 8, 15);
INSERT INTO FOOTER VALUES (4, 'Toyota Rav4', NULL, 15, NULL);
INSERT INTO FOOTER VALUES (5, 'Kia Sportage', NULL, NULL, 18); 
```

## Input Table:

```
| id | car           | length | width | height |
|----|---------------|--------|-------|--------|
| 1  | Hyundai Tucson| 15     | 6     | NULL   |
| 2  | NULL          | NULL   | NULL  | 20     |
| 3  | NULL          | 12     | 8     | 15     |
| 4  | Toyota Rav4   | NULL   | 15    | NULL   |
| 5  | Kia Sportage  | NULL   | NULL  | 18     |
```

## Expected Output Table:

```
| car          | last_length | last_width | last_height |
|--------------|-------------|------------|-------------|
| Kia Sportage | 12          | 15         | 18          |
```


## Solutions:

### Solution 1:

```sql
-- SQL Server does not support the LIMIT clause. We'll need to use TOP instead.
with cte as
    (select *
    , sum(case when car is null then 0 else 1 end) over(order by id) as car_segment
    , sum(case when length is null then 0 else 1 end) over(order by id) as length_segment
    , sum(case when width is null then 0 else 1 end) over(order by id) as width_segment
    , sum(case when height is null then 0 else 1 end) over(order by id) as height_segment
    from FOOTER)
select 
  top 1
  first_value(car) over(partition by car_segment order by id) as car
, first_value(length) over(partition by length_segment order by id) as last_length 
, first_value(width) over(partition by width_segment order by id) as last_width
, first_value(height) over(partition by height_segment order by id) as last_height
from cte 
order by id desc;
```

#### Explanation:

##### Step 1: Common Table Expression (CTE) Calculation

```sql
with cte as
    (select *
    , sum(case when car is null then 0 else 1 end) over(order by id) as car_segment
    , sum(case when length is null then 0 else 1 end) over(order by id) as length_segment
    , sum(case when width is null then 0 else 1 end) over(order by id) as width_segment
    , sum(case when height is null then 0 else 1 end) over(order by id) as height_segment
    from FOOTER)
```

In this step, a CTE named `cte` is created. This CTE calculates segments based on non-null values for each column using the `SUM` and `CASE` functions with `OVER` clause. Each non-null value increments the corresponding segment, and the segments are ordered by the `id`.

##### Step 2: Selecting Last Non-Null Values

```sql
select 
  top 1
  first_value(car) over(partition by car_segment order by id) as car
, first_value(length) over(partition by length_segment order by id) as last_length 
, first_value(width) over(partition by width_segment order by id) as last_width
, first_value(height) over(partition by height_segment order by id) as last_height
from cte 
order by id desc;
```

In this step, the last non-null values for each column are selected using the `FIRST_VALUE` function along with window functions `OVER` 
and `PARTITION BY`. The `TOP 1` with `ORDER BY id DESC` ensures that only the last record is selected.

---
### Solution 2:

```sql
SELECT *
FROM (SELECT car FROM FOOTER WHERE car IS NOT NULL ORDER BY id DESC OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY) car
CROSS JOIN (SELECT length FROM FOOTER WHERE length IS NOT NULL ORDER BY id DESC OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY) last_length
CROSS JOIN (SELECT width FROM FOOTER WHERE width IS NOT NULL ORDER BY id DESC OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY) last_width
CROSS JOIN (SELECT height FROM FOOTER WHERE height IS NOT NULL ORDER BY id DESC OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY) last_height;
```

### Explanation:

Selecting Last Non-Null Values for Each Column
- In this solution, subqueries are used to select the last non-null value for each column using the `OFFSET` and `FETCH NEXT` clauses. The `ORDER BY id DESC` ensures that the last record is selected.



---

### Solution 3:

```sql
SELECT TOP 1
    car,
    (SELECT TOP 1 length FROM FOOTER WHERE length IS NOT NULL ORDER BY id DESC) AS last_length,
    (SELECT TOP 1 width FROM FOOTER WHERE width IS NOT NULL ORDER BY id DESC) AS last_width,
    (SELECT TOP 1 height FROM FOOTER WHERE height IS NOT NULL ORDER BY id DESC) AS last_height
FROM 
    FOOTER
WHERE 
    (length IS NOT NULL OR width IS NOT NULL OR height IS NOT NULL)
ORDER BY 
    id DESC;
```

#### Explanation:
 Selecting Last Non-Null Values for Each Column

- In this solution, subqueries are used to select the last non-null value for each column. 
The `TOP 1` with `ORDER BY id DESC` ensures that the last record is selected. The `COALESCE` function is used to handle scenarios where there are no non-null values.

---
### Solution 4:

```sql
SELECT TOP 1
    car,
    COALESCE((SELECT TOP 1 length FROM FOOTER WHERE length IS NOT NULL ORDER BY id DESC), (SELECT TOP 1 length FROM FOOTER ORDER BY id DESC)) AS last_length,
    COALESCE((SELECT TOP 1 width FROM FOOTER WHERE width IS NOT NULL ORDER BY id DESC), (SELECT TOP 1 width FROM FOOTER ORDER BY id DESC)) AS last_width,
    COALESCE((SELECT TOP 1 height FROM FOOTER WHERE height IS NOT NULL ORDER BY id DESC), (SELECT TOP 1 height FROM FOOTER ORDER BY id DESC)) AS last_height
FROM 
    FOOTER
WHERE 
    (length IS NOT NULL OR width IS NOT NULL OR height IS NOT NULL)
ORDER BY 
    id DESC;
```



#### Explanation:


In this solution, subqueries are used to select the last non-null value for each column. Let's break down how `COALESCE` is working:

- For `last_length`:
  - The inner query `(SELECT TOP 1 length FROM FOOTER WHERE length IS NOT NULL ORDER BY id DESC)` selects the last non-null value for the `length` column from rows where `length` is not null, ordered by `id` in descending order.
  - If there are no non-null values for `length`, the inner query `(SELECT TOP 1 length FROM FOOTER ORDER BY id DESC)` selects the last value for `length` from the entire table, regardless of whether it's null or not.
  - `COALESCE` returns the first non-null value from the list of arguments. So, if the first inner query returns a non-null value for `length`, it will be selected. Otherwise, the value from the second inner query will be selected.

##### Example:
Let's consider a sample row from the `FOOTER` table:

```
| id | car           | length | width | height |
|----|---------------|--------|-------|--------|
| 1  | Hyundai Tucson| 15     | 6     | NULL   |
```

For the `last_length` column:
- The first inner query `(SELECT TOP 1 length FROM FOOTER WHERE length IS NOT NULL ORDER BY id DESC)` will return `15`, as it's the last non-null value for `length` in the table.
- The second inner query `(SELECT TOP 1 length FROM FOOTER ORDER BY id DESC)` will also return `15`, as it's the last value for `length` in the table.
- Therefore, `COALESCE(15, 15)` will return `15` as the final value for `last_length`.

The same logic applies to `last_width` and `last_height` columns.



- Solution 4 effectively handles scenarios where there are no non-null values for a column by selecting the last value from the entire table using `COALESCE`. This ensures that the query always returns the last non-null values for each column, providing the desired output. Therefore, Solution 4 is a robust approach to solving the problem.


---
## Conclusion:


Solution 1 uses Common Table Expressions (CTEs) to calculate segments based on non-null values for each column and then selects the last non-null value for each column. Solution 2 directly selects the last non-null values for each column using subqueries. Solution 3 and Solution 4 also utilize subqueries to select the last non-null values, with Solution 4 using the `COALESCE` function to handle null values.

Among the provided solutions, Solution 1 stands out as the best choice for several reasons:

- It uses window functions efficiently, resulting in better performance on larger datasets.
- The logic is clear once understood, providing a balance between readability and complexity.
- It's adaptable to changes in requirements or dataset structure due to the use of window functions and CTEs.
Therefore, Solution 1 is considered the better option.

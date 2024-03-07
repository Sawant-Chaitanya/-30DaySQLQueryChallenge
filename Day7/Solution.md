
## PROBLEM STATEMENT:

In the given input table DAY_INDICATOR field indicates the day of the week with the first character being Monday, followed by Tuesday, and so on.
Write a query to filter the dates column to showcase only those days where the day_indicator character for that day of the week is 1

----

##  Data
```SQL
-- Micrososft SQL Server
drop table if exists Day_Indicator;
create table Day_Indicator
(
	Product_ID 		varchar(10),	
	Day_Indicator 	varchar(7),
	Dates			date
);
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'04-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'05-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'06-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'07-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'08-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'09-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'10-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'04-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'05-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'06-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'07-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'08-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'09-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'10-Mar-2024', 102));

select * from Day_Indicator;
```

## Input Table

| PRODUCT_ID | DAY_INDICATOR |    DATES    |
|------------|---------------|-------------|
|    AP755   |    1010101    | 04-03-2024  |
|    AP755   |    1010101    | 05-03-2024  |
|    AP755   |    1010101    | 06-03-2024  |
|    AP755   |    1010101    | 07-03-2024  |
|    AP755   |    1010101    | 08-03-2024  |
|    AP755   |    1010101    | 09-03-2024  |
|    AP755   |    1010101    | 10-03-2024  |
|    XQ802   |    1000110    | 04-03-2024  |
|    XQ802   |    1000110    | 05-03-2024  |
|    XQ802   |    1000110    | 06-03-2024  |
|    XQ802   |    1000110    | 07-03-2024  |
|    XQ802   |    1000110    | 08-03-2024  |
|    XQ802   |    1000110    | 09-03-2024  |
|    XQ802   |    1000110    | 10-03-2024  |

----
## Expected Output Table

| PRODUCT_ID | DAY_INDICATOR |    DATES    |
|------------|---------------|-------------|
|    AP755   |    1010101    | 04-03-2024  |
|    AP755   |    1010101    | 06-03-2024  |
|    AP755   |    1010101    | 08-03-2024  |
|    AP755   |    1010101    | 10-03-2024  |
|    XQ802   |    1000110    | 04-03-2024  |
|    XQ802   |    1000110    | 08-03-2024  |
|    XQ802   |    1000110    | 09-03-2024  |

---
## Here is a solution in the MS SQL server
```SQL
-- Create a common table expression (CTE) SplitDayIndicator to calculate the day of the week and split the day indicator string
WITH SplitDayIndicator AS (
    SELECT 
        Product_ID,
        Dates,
        Day_Indicator,
        -- Calculate the day of the week using the DATEPART function, adding 5 and taking the modulo 7 to get the correct index
        ((DATEPART(dw, Dates) + 5) % 7 + 1) AS day_of_week
    FROM 
        Day_Indicator
),
-- Create another CTE SplitDay to split the day indicator string into individual characters based on the day of the week
SplitDay AS (
    SELECT 
        Product_ID,
        Dates,
        Day_Indicator,
        -- Use the SUBSTRING function to extract the indicator for the corresponding day of the week
        SUBSTRING(Day_Indicator, day_of_week, 1) AS Indicator
    FROM 
        SplitDayIndicator
)
-- Finally, select the desired columns from the SplitDay CTE where the indicator is '1'
SELECT 
    Product_ID,
    Day_Indicator,
    Dates
FROM 
    SplitDay
WHERE 
    Indicator = '1';

```
---
## Explanation




### Step 1: Calculate Day of the Week and Split Day Indicator

```sql
-- Create a common table expression (CTE) SplitDayIndicator to calculate the day of the week and split the day indicator string
WITH SplitDayIndicator AS (
    SELECT 
        Product_ID,
        Dates,
        Day_Indicator,
        -- Calculate the day of the week using the DATEPART function, adding 5 and taking the modulo 7 to get the correct index
        ((DATEPART(dw, Dates) + 5) % 7 + 1) AS day_of_week
    FROM 
        Day_Indicator
)
```

Explanation:
- This CTE calculates the day of the week for each date in the input table using the `DATEPART` function. It adds 5 to the result and takes the modulo 7 to adjust the index to start from Monday (1) instead of Sunday (0).

### Step 2: Split Day Indicator String

```sql
-- Create another CTE SplitDay to split the day indicator string into individual characters based on the day of the week
SplitDay AS (
    SELECT 
        Product_ID,
        Dates,
        Day_Indicator,
        -- Use the SUBSTRING function to extract the indicator for the corresponding day of the week
        SUBSTRING(Day_Indicator, day_of_week, 1) AS Indicator
    FROM 
        SplitDayIndicator
)
```

Explanation:
- This CTE uses the `SUBSTRING` function to split the `Day_Indicator` string into individual characters based on the calculated `day_of_week` index obtained from the previous step.

### Step 3: Filter Rows with Indicator '1'

```sql
-- Finally, select the desired columns from the SplitDay CTE where the indicator is '1'
SELECT 
    Product_ID,
    Day_Indicator,
    Dates
FROM 
    SplitDay
WHERE 
    Indicator = '1';
```

Explanation:
- This final query selects the `Product_ID`, `Day_Indicator`, and `Dates` columns from the `SplitDay` CTE.
- It filters the rows where the `Indicator` column is equal to '1', indicating that the corresponding day of the week has a value of 1 in the day indicator string.

---

This query effectively filters the dates column to showcase only those days where the `Day_Indicator` character for that day of the week is 1.


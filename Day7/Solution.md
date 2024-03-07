
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
WITH SplitDayIndicator AS (
    SELECT 
        Product_ID,
        Dates,
        Day_Indicator,
        ((DATEPART(dw, Dates) + 5) % 7 + 1) AS day_of_week
    FROM 
        Day_Indicator
), SplitDay AS (
    SELECT 
        Product_ID,
        Dates,
        SUBSTRING(Day_Indicator, day_of_week, 1) AS Indicator
    FROM 
        SplitDayIndicator
)
SELECT 
    Product_ID,
    Day_Indicator,
    Dates
FROM 
    SplitDay
WHERE 
    Indicator = '1';
```

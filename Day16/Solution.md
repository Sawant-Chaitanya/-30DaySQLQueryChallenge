

### SQL Query:

```sql
-- Given table contains reported covid cases in 2020. 
-- Calculate the percentage increase in covid cases each month versus cumulative cases as of the prior month.
-- Return the month number, and the percentage increase rounded to one decimal. Order the result by the month.


-- Common Table Expression (CTE) to calculate total cases reported per month
WITH MonthlyCases AS (
    SELECT 
        MONTH(dates) AS MonthNumber,
        YEAR(dates) AS Year,
        SUM(cases_reported) AS TotalCases
    FROM 
        covid_cases
    GROUP BY 
        MONTH(dates), YEAR(dates)
),
-- CTE to calculate cumulative cases for each month
CumulativeCases AS (
    SELECT 
        mc.MonthNumber,
        mc.Year,
        mc.TotalCases,
        SUM(mc.TotalCases) OVER (PARTITION BY mc.Year ORDER BY mc.MonthNumber) AS CumulativeCases
    FROM 
        MonthlyCases mc
)

-- Main query to calculate percentage increase in cases each month versus cumulative cases as of the prior month
SELECT 
    c.MonthNumber,
    -- Calculate percentage increase and round to one decimal place
    ROUND(((c.TotalCases * 100.0) / NULLIF(c.CumulativeCases - c.TotalCases, 0)), 1) AS PercentageIncrease
FROM 
    CumulativeCases c
ORDER BY 
    c.Year, c.MonthNumber;
```

### Explanation:

1. **MonthlyCases Common Table Expression (CTE)**:
   - Calculates the total cases reported for each month.
   - Groups the data by month and year, summing up the reported cases.

2. **CumulativeCases Common Table Expression (CTE)**:
   - Calculates the cumulative cases for each month.
   - Uses the `SUM()` window function to calculate the cumulative cases over the months, partitioned by the year and ordered by the month number.

3. **Main Query**:
   - Calculates the percentage increase in cases for each month compared to the cumulative cases of the previous month.
   - The percentage increase is calculated as `(TotalCases / (CumulativeCases - TotalCases)) * 100`, rounded to one decimal place.
   - `NULLIF()` is used to handle division by zero cases.

4. **Output**:
   - Returns the month number, and percentage increase rounded to one decimal place.
   - The result set is ordered by year and month number.

This query calculates the percentage increase in COVID cases each month compared to the cumulative cases as of the prior month. It's organized using Common Table Expressions (CTEs) to break down the calculation steps into manageable parts.

## Problem Statement
### **Remove redundant pairs from the brands table based on certain conditions.**

- For pairs of brands in the same year (e.g. apple/samsung/2020 and samsung/apple/2020) 
    - if custom1 = custom3 and custom2 = custom4 : then keep only one pair
- For pairs of brands in the same year 
    - if custom1 != custom3 OR custom2 != custom4 : then keep both pairs
- For brands that do not have pairs in the same year : keep those rows as well

											

#### Input Table: brands
| brand1   | brand2   | year | custom1 | custom2 | custom3 | custom4 |
|----------|----------|------|---------|---------|---------|---------|
| apple    | samsung  | 2020 | 1       | 2       | 1       | 2       |
| samsung  | apple    | 2020 | 1       | 2       | 1       | 2       |
| apple    | samsung  | 2021 | 1       | 2       | 5       | 3       |
| samsung  | apple    | 2021 | 5       | 3       | 1       | 2       |
| google   | NULL     | 2020 | 5       | 9       | NULL    | NULL    |
| oneplus  | nothing  | 2020 | 5       | 9       | 6       | 3       |


For pairs of brands in the same year:
- If `custom1 = custom3` and `custom2 = custom4`, keep only one pair.
- If `custom1 != custom3` OR `custom2 != custom4`, keep both pairs.
- For brands that do not have pairs in the same year, keep those rows as well.
---
### Approach to Solve:

1. **CTE with Row Numbers:**
   - Create a Common Table Expression (CTE) to assign row numbers partitioned by `brand1`, `brand2`, and `year`.
   - Use `ROW_NUMBER()` function to generate row numbers for each partition.

2. **Selecting Rows:**
   - Select rows from the CTE based on the given conditions to remove redundant pairs and keep rows without pairs in the same year.

3. **Ordering Results:**
   - Order the results by `brand1`, `brand2`, and `year` for consistent presentation.
---  
### Solution
```SQL
-- Remove Redundant Pairs in SQL Server
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY CASE WHEN brand1 < brand2 THEN CONCAT(brand1, brand2, year)
                                                ELSE CONCAT(brand2, brand1, year)
                                           END ORDER BY (SELECT NULL)) AS rn
    FROM brands
)
SELECT brand1, brand2, year, custom1, custom2, custom3, custom4
FROM cte
WHERE rn = 1 OR (custom1 <> custom3 AND custom2 <> custom4);

```
---
### SQL Query Explanation:

#### **Step 1: Create CTE with Row Numbers**

```sql
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY CASE WHEN brand1 < brand2 THEN CONCAT(brand1, brand2, year)
                                                ELSE CONCAT(brand2, brand1, year)
                                           END ORDER BY (SELECT NULL)) AS rn
    FROM brands
)
```

In this step:
- A Common Table Expression (CTE) named `cte` is created.
- `ROW_NUMBER()` function assigns row numbers within each partition defined by `brand1`, `brand2`, and `year`.
- The `CASE` statement constructs a unique identifier `pair_id` for each pair of brands and year.

#### **Step 2: Selecting Rows**

```sql
SELECT brand1, brand2, year, custom1, custom2, custom3, custom4
FROM cte
WHERE rn = 1 OR (custom1 <> custom3 AND custom2 <> custom4);
```

In this step:
- Selects rows from the CTE where the row number is 1 within each partition or where the condition for non-redundant pairs is met.

#### **Step 3: Ordering Results**

```sql
ORDER BY brand1, brand2, year;
```

In this step:
- Orders the result set by `brand1`, `brand2`, and `year` for consistent presentation.
---
### Result Table
| brand1   | brand2   | year | custom1 | custom2 | custom3 | custom4 |
|----------|----------|------|---------|---------|---------|---------|
| apple    | samsung  | 2020 | 1       | 2       | 1       | 2       |
| google   | NULL     | 2020 | 5       | 9       | NULL    | NULL    |
| oneplus  | nothing  | 2020 | 5       | 9       | 6       | 3       |
| apple    | samsung  | 2021 | 1       | 2       | 5       | 3       |
| samsung  | apple    | 2021 | 5       | 3       | 1       | 2       |
---
### Conclusion
The SQL query efficiently removes redundant pairs from the brands table based on the specified conditions, resulting in a clean and concise output.







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

### Approach to Solve:

1. **CTE with Row Numbers:**
   - Create a Common Table Expression (CTE) to assign row numbers partitioned by `brand1`, `brand2`, and `year`.
   - Use `ROW_NUMBER()` function to generate row numbers for both `brand1` and `brand2`.

2. **Selecting Rows:**
   - Select rows from the CTE where either of the following conditions is met:
     - Keep only one pair if `custom1 = custom3` and `custom2 = custom4`.
     - Keep both pairs if `custom1 != custom3` OR `custom2 != custom4`.
     - Keep rows without pairs in the same year.

3. **Ordering Results:**
   - Order the results by `brand1`, `brand2`, and `year`.

### SQL Query Explanation:


#### **Step 1: Create CTE with Row Numbers**

```sql
WITH CTE AS (
    SELECT 
        *,
        ROW_NUMBER() OVER(PARTITION BY brand1, brand2, year ORDER BY brand1) AS rn1,
        ROW_NUMBER() OVER(PARTITION BY brand1, brand2, year ORDER BY brand2) AS rn2
    FROM 
        brands
)
```

In this step:
- A Common Table Expression (CTE) named `CTE` is created.
- `ROW_NUMBER()` function is used to assign row numbers within each partition defined by `brand1`, `brand2`, and `year`.
- Two row number columns `rn1` and `rn2` are generated, one for each brand.

#### **Step 2: Selecting Rows**

```sql
SELECT 
    brand1,
    brand2,
    year,
    custom1,
    custom2,
    custom3,
    custom4
FROM 
    CTE
WHERE 
    (rn1 = 1 AND rn2 = 1)  -- Keep only one pair if custom1 = custom3 and custom2 = custom4
    OR (rn1 > 1 AND rn2 > 1) -- Keep both pairs if custom1 != custom3 OR custom2 != custom4
    OR (rn1 = 1 AND rn2 > 1) -- Keep rows without pairs in the same year
```

In this step:
- Selects rows from the CTE where the conditions are met according to the problem statement.
- Keeps only one pair if `custom1 = custom3` and `custom2 = custom4`.
- Keeps both pairs if `custom1 != custom3` OR `custom2 != custom4`.
- Keeps rows without pairs in the same year.

#### **Step 3: Ordering Results**

```sql
ORDER BY 
    brand1, brand2, year;
```

In this step:
- Orders the result set by `brand1`, `brand2`, and `year` to ensure consistent ordering.



### Result Table
| brand1   | brand2   | year | custom1 | custom2 | custom3 | custom4 |
|----------|----------|------|---------|---------|---------|---------|
| apple    | samsung  | 2020 | 1       | 2       | 1       | 2       |
| google   | NULL     | 2020 | 5       | 9       | NULL    | NULL    |
| oneplus  | nothing  | 2020 | 5       | 9       | 6       | 3       |
| apple    | samsung  | 2021 | 1       | 2       | 5       | 3       |
| samsung  | apple    | 2021 | 5       | 3       | 1       | 2       |

### Conclusion
The query efficiently removes redundant pairs from the brands table based on the specified conditions, resulting in a clean and concise output.







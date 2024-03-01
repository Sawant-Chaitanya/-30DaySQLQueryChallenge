```SQL
WITH CTE AS (
    SELECT 
        *,
        ROW_NUMBER() OVER(PARTITION BY brand1, brand2, year ORDER BY brand1) AS rn1,
        ROW_NUMBER() OVER(PARTITION BY brand1, brand2, year ORDER BY brand2) AS rn2
    FROM 
        brands
)
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
ORDER BY 
    brand1, brand2 , year;
```

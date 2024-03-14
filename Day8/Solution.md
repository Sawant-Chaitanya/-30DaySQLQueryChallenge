```sql
-- Selecting row_id, job_role, and skills from the job_skills table
SELECT js.row_id,
       ISNULL(js.job_role, prev.job_role) AS job_role, -- If job_role is null, fill it with the previous non-null job_role
       js.skills
FROM job_skills js
-- Using CROSS APPLY to find the previous non-null job_role for each row
CROSS APPLY (
    -- Selecting the top 1 job_role from job_skills table where row_id is less than or equal to the current row_id
    SELECT TOP 1 job_role
    FROM job_skills AS j
    WHERE j.row_id <= js.row_id
        AND job_role IS NOT NULL -- Ensuring the selected job_role is not null
    ORDER BY j.row_id DESC -- Ordering by row_id in descending order to get the most recent non-null job_role
) AS prev;
```

### Explanation:

This SQL query retrieves data from the `job_skills` table while handling cases where the `job_role` column is null. Here's how it works:

1. **SELECT Statement**:
   - It selects `row_id`, `job_role`, and `skills` from the `job_skills` table.

2. **ISNULL Function**:
   - If the `job_role` is null for a particular row, the `ISNULL` function substitutes it with the `job_role` from the previous row (if available).

3. **CROSS APPLY Subquery**:
   - This subquery finds the previous non-null `job_role` for each row. It does so by:
     - Selecting the top 1 `job_role` from the `job_skills` table where the `row_id` is less than or equal to the current row's `row_id`.
     - Ensuring that the selected `job_role` is not null.
     - Ordering the results by `row_id` in descending order to get the most recent non-null `job_role`.

4. **Result**:
   - The final result contains `row_id`, `job_role` (possibly filled with the previous non-null value), and `skills` for each row in the `job_skills` table.



## [Problem Statement:](https://youtu.be/rM1BVoBke04?si=b2ulqueTHrF0sT3M)

A ski resort company is planning to construct a new ski slope using a pre-existing network of mountain huts and trails between them. A new slope has to begin at one of the mountain huts, have a middle station at another hut connected with the first one by a direct trail, and end at the third mountain hut which is also connected by a direct trail to the second hut. The altitude of the three huts chosen for constructing the ski slope has to be strictly decreasing.

You are given two SQL tables, `mountain_huts` and `trails`, with the following structure:

```sql
CREATE TABLE mountain_huts (
    id INTEGER NOT NULL,
    name VARCHAR(40) NOT NULL,
    altitude INTEGER NOT NULL,
    UNIQUE(name),
    UNIQUE(id)
);

CREATE TABLE trails (
    hut1 INTEGER NOT NULL,
    hut2 INTEGER NOT NULL
);

INSERT INTO mountain_huts VALUES (1, 'Dakonat', 1900);
INSERT INTO mountain_huts VALUES (2, 'Natisa', 2100);
INSERT INTO mountain_huts VALUES (3, 'Gajantut', 1600);
INSERT INTO mountain_huts VALUES (4, 'Rifat', 782);
INSERT INTO mountain_huts VALUES (5, 'Tupur', 1370);

INSERT INTO trails VALUES (1, 3);
INSERT INTO trails VALUES (3, 2);
INSERT INTO trails VALUES (3, 5);
INSERT INTO trails VALUES (4, 5);
INSERT INTO trails VALUES (1, 5);
```

### mountain_huts table

| Id | Name     | Altitude |
|----|----------|----------|
| 1  | Dakonat  | 1900     |
| 2  | Natisa   | 2100     |
| 3  | Gajantut | 1600     |
| 4  | Rifat    | 782      |
| 5  | Tupur    | 1370     |


### trails table:

| Hut1 | Hut2 |
|------|------|
| 1    | 3    |
| 3    | 2    |
| 3    | 5    |
| 4    | 5    |
| 1    | 5    |


Each entry in the table `trails` represents a direct connection between huts with IDs `hut1` and `hut2`. Note that all trails are bidirectional.

---
### Main Question
Create a query that finds all triplets(startpt,middlept,endpt) representing the mountain huts 
that may be used for construction of a ski slope.
Output returned by the query can be ordered in any way.

### Expected Output:

| startpt  | middlept | endpt |
|----------|----------|-------|
| Dakonat  | Gajantut | Tupur |
| Dakonat  | Tupur    | Rifat |
| Gajantut | Tupur    | Rifat |
| Natisa   | Gajantut | Tupur |

### Assumption

    - there is no trail going from a hut back to itself;
    - for every two huts there is at most one direct trail connecting them;
    - each hut from table trails occurs in table mountain_huts
---
## Solution:

```sql
-- CTE to select start and end huts along with their names and altitudes
with cte_trails1 as (
    select t1.hut1 as start_hut, h1.name as start_hut_name
    , h1.altitude as start_hut_altitude, t1.hut2 as end_hut
    from mountain_huts h1
    join trails t1 on t1.hut1 = h1.id
),
-- CTE to calculate altitude difference and assign a flag
cte_trails2 as (
    select t2.*, h2.name as end_hut_name, h2.altitude as end_hut_altitude
    , case when start_hut_altitude > h2.altitude then 1 else 0 end as altitude_flag
    from cte_trails1 t2
    join mountain_huts h2 on h2.id = t2.end_hut
),
-- CTE to filter out valid triplets based on altitude conditions
cte_final as (
    select case when altitude_flag = 1 then start_hut else end_hut end as start_hut
    , case when altitude_flag = 1 then start_hut_name else end_hut_name end as start_hut_name
    , case when altitude_flag = 1 then end_hut else start_hut end as end_hut
    , case when altitude_flag = 1 then end_hut_name else start_hut_name end as end_hut_name
    from cte_trails2
)
-- Joining final CTE with itself to get all possible triplets and selecting required columns
select c1.start_hut_name as startpt
, c1.end_hut_name as middlept
, c2.end_hut_name as endpt
from cte_final c1
join cte_final c2 on c1.end_hut = c2.start_hut;
```
---
## Approach:

We solve this problem using Common Table Expressions (CTEs). The approach involves:

1. **Creating CTEs for trails**: We first create a CTE (`cte_trails1`) to join the mountain huts with their connected trails. Then, another CTE (`cte_trails2`) is created to calculate the altitude difference between huts.
   
2. **Final CTE for filtering**: We create a final CTE (`cte_final`) to filter out the valid triplets based on altitude conditions.

3. **Joining and selecting**: Finally, we join the final CTE with itself to get all possible triplets and select the required columns.

---

## Step-by-Step Explanation:


We'll break down the provided SQL query step by step, explaining each part along with relevant code snippets.

#### Step 1: CTE to Select Start and End Huts

```sql
with cte_trails1 as (
    select t1.hut1 as start_hut, h1.name as start_hut_name
    , h1.altitude as start_hut_altitude, t1.hut2 as end_hut
    from mountain_huts h1
    join trails t1 on t1.hut1 = h1.id
)
```

In this step, we create a Common Table Expression (CTE) named `cte_trails1`. This CTE selects the start and end huts along with their names and altitudes by joining the `mountain_huts` table with the `trails` table using the `id` and `hut1` columns, respectively.

#### Step 2: CTE to Calculate Altitude Difference and Assign a Flag

```sql
cte_trails2 as (
    select t2.*, h2.name as end_hut_name, h2.altitude as end_hut_altitude
    , case when start_hut_altitude > h2.altitude then 1 else 0 end as altitude_flag
    from cte_trails1 t2
    join mountain_huts h2 on h2.id = t2.end_hut
)
```

In this step, we create another CTE named `cte_trails2`. This CTE calculates the altitude difference between the start and end huts and assigns a flag based on whether the altitude of the start hut is greater than that of the end hut.

#### Step 3: CTE to Filter Out Valid Triplets Based on Altitude Conditions

```sql
cte_final as (
    select case when altitude_flag = 1 then start_hut else end_hut end as start_hut
    , case when altitude_flag = 1 then start_hut_name else end_hut_name end as start_hut_name
    , case when altitude_flag = 1 then end_hut else start_hut end as end_hut
    , case when altitude_flag = 1 then end_hut_name else start_hut_name end as end_hut_name
    from cte_trails2
)
```

In this step, we create another CTE named `cte_final`. This CTE filters out the valid triplets based on altitude conditions. It selects the start and end huts along with their names, ensuring that the altitude is strictly decreasing.

#### Step 4: Joining Final CTE with Itself and Selecting Required Columns

```sql
select c1.start_hut_name as startpt
, c1.end_hut_name as middlept
, c2.end_hut_name as endpt
from cte_final c1
join cte_final c2 on c1.end_hut = c2.start_hut;
```

In this step, we join the `cte_final` CTE with itself to get all possible triplets of mountain huts that satisfy the altitude conditions. We then select the required columns: start point, middle point, and end point.

This completes the explanation of the provided SQL query. It efficiently finds all valid triplets of mountain huts that can be used to construct the ski slope, ensuring the altitude decrease along the slope.

---

## Result Table:


| startpt  | middlept | endpt |
|----------|----------|-------|
| Dakonat  | Gajantut | Tupur |
| Dakonat  | Tupur    | Rifat |
| Gajantut | Tupur    | Rifat |
| Natisa   | Gajantut | Tupur |

---
## Conclusion:

The SQL query efficiently finds all valid triplets of mountain huts that can be used to construct the ski slope, satisfying the given conditions of altitude decrease along the slope.

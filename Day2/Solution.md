## Solution

```SQL
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
## Result Table

| startpt  | middlept | endpt |
|----------|----------|-------|
| Dakonat  | Gajantut | Tupur |
| Dakonat  | Tupur    | Rifat |
| Gajantut | Tupur    | Rifat |
| Natisa   | Gajantut | Tupur |

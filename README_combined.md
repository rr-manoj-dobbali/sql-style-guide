# SQL Style Guide
Guide for formatting SQL scripts

# Why Style Guide

1. Increase readability
2. Be consistent

# Guidelines with examples

```
-- Good
select distinct
    tx.member_id,
    tx.store_id,
    tx.cashback_rate,
    tx.sales,
    ftb.num_orders,
    ftb.commission,
    ftb.cashback as cashback_perc,
    tx.num_stores as store_count,
    tx.signup_date as date_signup,
    tx.member_engaged,
    tx.lifecycle_stage, 
    case tx.application_type
        when 'Webpage' then 'Website'
        when 'App' then 'Mobile'
        else 'Other'
    end as page_name
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on
    tx.member_id = ftb.member_id
    and tx.store_id = ftb.store_id
where 
    tx.member_id = 1234
    and tx.store_id = 345
    and ftb.cashback > 10
    and lifecycle_stage !='lst'

-- Bad 
SELECT distinct tx.member_id, tx.store_id, tx.cashback_rate
    , tx.sales
    , ftb.num_orders      
    , ftb.commission
    , ftb.cashback      AS cashbackPerc
    , tx.num_stores     AS storecount
    , tx.signup_date    AS dateSignup
    , tx.member_engaged
    , tx.lifecycle_stage
    , case when tx.application_type = 'Webpage' then 'Website'
        when tx.application_type = 'App' 
            then 'Mobile'
        else 'Other'
    end as page_name
FROM ebates_prod.dw.order_transactions tx
INNER JOIN ebates_prod.summary.member_ftbs_by_store ftb 
    ON ftb.member_id = tx.member_id AND tx.store_id = ftb.store_id
WHERE tx.member_id = 1234
    and tx.store_id = 345
    and ftb.cashback > 10
    and lifecycle_stage <> 'lst'
```
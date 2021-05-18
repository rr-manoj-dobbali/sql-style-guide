# SQL Style Guide
Guide for formatting SQL scripts

# Guidelines 

## Use lowercase SQL

```
-- Good
select distinct order_merchant_id as store_id, member_id
from ebates_prod.dw.order_transactions

-- Bad : Dont use upper case & lower case combinations 
SELECT DISTINCT order_merchant_id AS store_id, member_id
FROM ebates_prod.dw.order_transactions

-- Bad : Dont use upper case for queries 
SELECT DISTINCT ORDER_MERCHANT_ID AS STORE_ID, MEMBER_ID
FROM EBATES_PROB.DW.ORDER_TRANSACTIONS
```

## Column selection

Keep columns selected as one row per column name if there are more than three columns

```
-- Good
select
    member_id,
    cashback_rate,
    sales,
    num_orders,
    commission,
    cashback,
    number_of_stores_with_orders,
    signup_date,
    member_engaged,
    lifecycle_stage
from ebates_prod.temp.some_table

-- Bad : Dont add few columns in a line and a few other in next line with one column per line

select member_id, cashback_rate, sales,
    num_orders,
    commission,
    cashback,
    number_of_stores_with_orders,
    signup_date,
    member_engaged,
    lifecycle_stage
from ebates_prod.temp.some_table

-- Bad : Dont add all column names in a single line
select member_id, cashback_rate, sales, num_orders, commission, cashback, number_of_stores_with_orders, signup_date, member_engaged, lifecycle_stage
from ebates_prod.temp.some_table
```

If number of columns in selection is less than three, write it down in a single line or write one column per line

```
-- Good
select
    member_id,
    cashback_rate,
    sales,
from ebates_prod.temp.some_table

-- Good
select member_id, cashback_rate, sales,
from ebates_prod.temp.some_table
```

## Dont start column names with commas

```
-- Good
select distinct
    interest_area,
    interest_score,
    i.member_id,
    order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx

-- Bad
select distinct
    interest_area
    , interest_score
    , i.member_id
    , order_merchant_id
from ebates_prod.dw.order_transactions
```

## Inner join vs join

For the sake of consistency across all the queries, lets use `join` instead of `inner join`

```
-- Good
select tx.*, ftb.*
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb on
    tx.member_id = ftb.member_id

-- bad
select tx.*, ftb.*
from ebates_prod.dw.order_transactions tx
inner join ebates_prod.summary.member_ftbs_by_store ftb on
    tx.member_id = ftb.member_id
```

## Join order & Key order same

```
-- Good
select distinct
    interest_area,
    interest_score,
    i.member_id as user_id 
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on tx.member_id = ftb.member_id

-- bad : 
select
    tx.order_merchant_id as store_id,
    to_date(tx.click_date) as ds,
    sum(tx.ot_amount) as sales_net,
    round(ort_rebate_percentage * 100 * sales_net, 1) as commission_dummy
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on ftb.member_id = tx.member_id
```

## Indent should be four spaces

```
-- Good
select distinct
    interest_area,
    interest_score,
    i.member_id as user_id 
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on ftb.member_id = tx.member_id

-- Bad
select distinct
 interest_area,
 interest_score,
 i.member_id as user_id 
 tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on tx.member_id = ftb.member_id
```

## No database or schema



## Multiple nesting

if you have to do nesting more than once or use multiple sub queries within a query, use CTEs instead

```
-- Bad
select
    count(buy) / count(l.member_id) as cvr,
    l.interest_area as interest_area,
    l.interest_score as interest_score
from (
    select distinct interest_area, interest_score, member_id
from ff_last_year_member_interest_area
where
    store_id != 1234
    and interest_area in (select interest_area, rank  as top10_rank
from ff_store_top_10_interest_area_last_1_yr
where store_id = 1234)
    and member_id not in (
        select member_id
        from ff_store_shopped_members_before_2_yrs
        where store_id = 1234 

-- Good

with top10_interests as (
        select distinct interest_area, interest_score, member_id
from ff_last_year_member_interest_area
),

last_year_store_member_interest_area(
    select interest_area, rank  as top10_rank
from ff_store_top_10_interest_area_last_1_yr
where store_id = 1234
)

select
    count(buy) / count(l.member_id) as cvr,
    l.interest_area as interest_area,
    l.interest_score as interest_score
from last_year_store_member_interest_area

```

## Misc

- Lines of SQL should be no longer than 80 characters
- Use tabs instead of spaces, It's easier to keep things consistent in version control when only space characters are used.
- Commas should be at the end of lines, except in where condition
- `distinct` should be in the samerow as `select`
- `as` keyword should be used when creating aliases 
- Prefer `!=` to `<>`. This is because != is more common in other programming languages and reads like "not equal" which is how we're more likely to speak
- Identifiers such as aliases and CTE names should be in lowercase snake_case.


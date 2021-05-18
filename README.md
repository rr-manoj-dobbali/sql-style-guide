# SQL Style Guide
Guide for formatting SQL scripts

# Why Style Guide

1. Increase readability
2. Be consistent

# Guidelines with examples

## General 

- Use all lowercase SQL

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

- Fieldnames or Variable names or aliased table names should be in `snake_cased`

```
-- Good
select count(*) as item_count

-- Bad
select count(*) as itemCount

-- Bad
select count(*) as itemcount
```

- For Indentation, use 4 spaces for one level of indentation and 8 for deeper and so on..
- Lines of SQL should be no longer than 100 characters. TODO : How do we make sure of that? 
- No trailing white space
- If queries/models are run by DBT then do not mention schema or database name, just use table name
- Use `!=` instead of `<>` since it is more popular in other programming languages
- Ordering and grouping by a number (eg. GROUP BY 1, 2) is preferred
- For the sake of consistency across all the queries, lets use `join` instead of `inner join`
- Use `--` for inline single line comments
- Use `/* */` for multi line comments


## OTHER SPECIFIC EXAMPLES

## Use `as` to rename columns or tables

```
-- Good
select
    table1.column1 as table1_column1, 
    count(members.user_id) as members_count
from ebates.dw.order_transactions as orders

-- Bad
select
    table1.column1 table1_column1, 
    count(members.user_id) members_count
from ebates.dw.order_transactions 
```


## Column selection

- Keep columns selected as one row per column name
- `select` should be in its own row

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

If number of columns in selection is less than three or if you are selecting all the columns using `*`, write it down in a single line or write one column per line

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

## Join order & key order same

Specify the order of a join with the FROM table first and JOIN table second. In the below example, `order_transactions` is joined first on `member_ftbs_by_store`, so the `on` condition should be in the same order, `tx.member_id = ftb.member_id` instead of `ftb.member_id = tx.member_id`

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

-- Bad
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

Use four spaces instead of tabs for indentation. Configure editor to convert tabs to spaces

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

## Use CTE's

- Queries within the CTE's should be one level indented (4 spaces)
- All the queries within CTE's should follow rest of the guidelines 
- Commas seperating queries/CTE's should be at the end of each query/CTE instead of begining of next CTE, For example

```
-- Good
with top10_interests as (

    select distinct interest_area, interest_score, member_id 
    from ff_last_year_member_interest_area

    ),

last_year_store_member_interest_area as (

    select interest_area, rank as top10_rank 
    ...
    ...

-- Bad
with top10_interests as (
    select distinct interest_area, interest_score, member_id 
    from ff_last_year_member_interest_area
    )

, last_year_store_member_interest_area as (
    select interest_area, rank as top10_rank 
    ...
    ...
```

- If you have to do nesting more than once or use multiple sub queries within a query, use CTEs instead. They are easy to read. Give meaningful names to them
- Leave an empty row above and below the query statement
- Closing CTE parentheses and comma  should use same indentation level as `with` and other CTE names

```
-- Good
with top10_interests as (

    select distinct interest_area, interest_score, member_id 
    from ff_last_year_member_interest_area

),

last_year_store_member_interest_area as ( -- CTE comments go here

    select interest_area, rank as top10_rank 
    from ff_store_top_10_interest_area_last_1_yr
    where store_id = 1234

),

select
    count(buy) / count(l.member_id) as cvr,
    l.interest_area as interest_area,
    l.interest_score as interest_score
from last_year_store_member_interest_area


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

```

## Do not align aliases `as`

```
-- Good
select 
    cashback as cashbackPerc
    num_stores as storecount
    signup_date as dateSignup
from ebates_prod.dw.order_transactions


-- Bad
select 
    cashback       as cashbackPerc
    num_stores     as storecount
    signup_date    as dateSignup
from ebates_prod.dw.order_transactions

```

## Aligning case/when statements

- `case`, `end` should be in its own line with one indentation. `end` line will have aliased column name
- Each `when` should be on its own line with one level deeper indentation

-- Good
select
    case
        when application_type = 'Webpage' then 'Website'
        when application_type = 'App' then 'Mobile'
        else 'Other'
    end as page_name
from events

-- Bad
select
    case
        when application_type = 'Webpage'
            then 'Website'
        when application_type = 'App'
            then 'Mobile'
        else 'Other'            
    end as page_name
from events

-- Bad 
select
    case when application_type = 'viewed_homepage' then 'Homepage'
        when application_type = 'viewed_editor' then 'Editor'
        else 'Other'        
    end as page_name
from events


## Joining `on` & 'where' condition

- Write `on` in its own line if there are multiple joining conditions
- When multiple conditions for joining, all `and`s should be one level deeper than `on`
- same logic applies to `where` condition
- Only exception is when there is one 

```
-- Good
select distinct
    interest_area,
    interest_score,
    i.member_id as user_id 
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on 
    tx.member_id = ftb.member_id
    and tx.store_id = ftb.store_id
where
    tx.member_id = 1234
    and tx.store_id = 345

-- Bad 
select distinct
    interest_area,
    interest_score,
    i.member_id as user_id 
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb 
on tx.member_id = ftb.member_id
    and tx.store_id = ftb.store_id
where tx.member_id = 1234
    and tx.store_id = 345
```



## Window functions

Keep window functions all in single line

```
-- Good
select
    member_id,
    store_id,
    row_number() over (partition by user_id order by order_date desc) as order_rank
from ebates_prod.dw.shopping_trips

-- Okay
select
    user_id,
    name,
    row_number() over (
        partition by user_id
        order by order_date desc
    ) as order_rank
from ebates_prod.dw.shopping_trips
```

## Simplify case statments name

```
  -- Good
  case
    when lifecycle_stage = 1 then 'stage1_Newbie'
    when lifecycle_stage = 2 then 'stage2_Browser'
    when lifecycle_stage = 3 then 'stage3_Browser_Toolbar'
    when lifecycle_stage = 4 then 'stage4_Shopper_App'
  end as lifecycle

  -- Better
  case lifecycle_stage
    when 1 then 'stage1_Newbie'
    when 2 then 'stage2_Browser'
    when 3 then 'stage3_Browser_Toolbar'
    WHEN 4 then 'stage4_Shopper_App'
  end as lifecycle
```
## Putting it all together

```
-- Good
with members as (

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

), 

stores as (

    select 
        store_id,
        ...
        ...
    from ebates_prod.dw.stores

)

select 
    m.*,
    s.*
from members m 
join stores s
    on m.member_id = s.member_id
    and m.store_id = s.store_id

-- Bad 
WITH members AS 
(SELECT distinct tx.member_id, tx.store_id, tx.cashback_rate
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
    and lifecycle_stage <> 'lst'), 

stores as (select 
        store_id,
        ...
        ...
    from ebates_prod.dw.stores
)

select m.*, s.*
from members m 
join stores s
on m.member_id = s.member_id
    and m.store_id = s.store_id
```
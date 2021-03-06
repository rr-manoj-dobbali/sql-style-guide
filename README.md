# SQL Style Guide
Guide for formatting SQL scripts

# Why Style Guide

1. Increase readability
2. Be consistent

# Guidelines with examples

## General

- Use all lowercase SQL

```sql
-- Good
select distinct
    order_merchant_id as store_id,
    member_id
from ebates_prod.dw.order_transactions

-- Bad : Dont use upper case & lower case combinations
SELECT DISTINCT order_merchant_id AS store_id, member_id
FROM ebates_prod.dw.order_transactions

-- Bad : Dont use upper case for queries
SELECT DISTINCT ORDER_MERCHANT_ID AS STORE_ID, MEMBER_ID
FROM EBATES_PROB.DW.ORDER_TRANSACTIONS
```

- Fieldnames or Variable names or aliased table names should be in `snake_cased`

```sql
-- Good
select count(*) as item_count

-- Bad
select count(*) as itemCount

-- Bad
select count(*) as itemcount
```

- For Indentation, use 4 spaces for one level of indentation and 8 for deeper and so on..
- Lines of SQL should be no longer than 120 characters. TODO : How do we make sure of that?
- No trailing white space
- If queries/models are run by DBT then do not mention schema or database name, just use table name
- Use `!=` instead of `<>` since it is more popular in other programming languages
- Ordering and grouping by a number (eg. GROUP BY 1, 2, 3, 4) is not preferred since it leads to confusion. Always use names instead.
  Only exception is when there are only one or two columns for group by or order by
- For the sake of consistency across all the queries, lets use `join` instead of `inner join`
- Use `--` for inline single line comments
- Use `/* */` for multi line comments
- Use single quotes for variable values, in where conditions, case statements etc


## OTHER SPECIFIC EXAMPLES

## Use `as` to rename columns or tables

```sql
-- Good
select
    orders.column1 as orders_column1,
    count(orders.user_id) as members_count
from ebates.dw.order_transactions as orders

-- Bad
select
    orders.column1 orders_column1,
    count(orders.user_id) members_count
from ebates.dw.order_transactions
```


## Column selection

- Keep columns selected as one row per column name
- `select` should be in its own row

```sql
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

If you are selecting all the columns using `*`, write it down in a single line or write one column per line

```sql
-- Good
select
    member_id,
    cashback_rate,
    sales
from ebates_prod.temp.some_table
```

## Dont start column names with commas

```sql
-- Good
select distinct
    interest_area,
    interest_score,
    member_id,
    order_merchant_id as store_id
from ebates_prod.dw.order_transactions

-- Bad
select distinct
    interest_area
    , interest_score
    , member_id
    , order_merchant_id
from ebates_prod.dw.order_transactions
```

## Join order & key order same

Specify the order of a join with the FROM table first and JOIN table second. In the below example, `order_transactions` is joined first on `member_ftbs_by_store`, so the `on` condition should be in the same order, `tx.member_id = ftb.member_id` instead of `ftb.member_id = tx.member_id`

```sql
-- Good
select distinct
    tx.interest_area,
    tx.interest_score,
    ftb.member_id as user_id,
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions as tx
join ebates_prod.summary.member_ftbs_by_store as ftb
    on tx.member_id = ftb.member_id

-- Bad
select distinct
    tx.interest_area,
    tx.interest_score,
    ftb.member_id as user_id,
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb
on ftb.member_id = tx.member_id
```

## Indent should be four spaces

Use four spaces instead of tabs for indentation. Configure editor to convert tabs to spaces

```sql
-- Good
select distinct
    tx.interest_area,
    tx.interest_score,
    ftb.member_id as user_id,
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions as tx
join ebates_prod.summary.member_ftbs_by_store as ftb
    on tx.member_id = ftb.member_id

-- Bad
select distinct
 tx.interest_area,
 tx.interest_score,
 ftb.member_id as user_id,
 tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions tx
join ebates_prod.summary.member_ftbs_by_store ftb
on tx.member_id = ftb.member_id
```

## Use CTE's

- Queries within the CTE's should be one level indented (4 spaces)
- All the queries within CTE's should follow rest of the guidelines
- Commas separating queries/CTE's should be at the end of each query/CTE instead of beginning of next CTE, For example

```sql
-- Good
with top10_interests as (
    select distinct
        interest_area,
        interest_score,
        member_id
    from ff_last_year_member_interest_area
),

last_year_store_member_interest_area as (
    select
        interest_area,
        rank as top10_rank
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
- Closing CTE parentheses and comma should be at beginning of the line separating queires in CTEs
- Separate each query within CTE with a blank line

```sql
-- Good
with top10_interests as (
    select distinct
        interest_area,
        interest_score,
        member_id
    from ff_last_year_member_interest_area
),

last_year_store_member_interest_area as (  -- CTE comments go here
    select
        interest_area,
        rank as top10_rank
    from top10_interests
    where store_id = 1234
)

select
    interest_area as interest_area,
    interest_score as interest_score,
    count(buy) / count(member_id) as cvr
from last_year_store_member_interest_area

-- Bad
select
    count(l.buy) / count(l.member_id) as cvr,
    l.interest_area as interest_area,
    l.interest_score as interest_score
from (
    select distinct interest_area, interest_score, member_id
from ff_last_year_member_interest_area
where
    store_id != 1234
    and interest_area in (select interest_area, rank as top10_rank
from ff_store_top_10_interest_area_last_1_yr
where store_id = 1234) l
    and member_id not in (
        select member_id
        from ff_store_shopped_members_before_2_yrs
        where store_id = 1234
```

## Do not align aliases `as`

```sql
-- Good
select
    cashback as cashback_perc,
    num_stores as store_count,
    signup_date as date_signed_up
from ebates_prod.dw.order_transactions

-- Bad
select
    cashback       as cashback_perc,
    num_stores     as store_count,
    signup_date    as date_signed_up
from ebates_prod.dw.order_transactions

```

## Formatting case/when statements

- `case`, `end` should be in its own line with one indentation. `end` line will have aliased column name
- Each `when` should be on its own line with one level deeper indentation

```sql
-- Good
select
    case lifecycle_stage
        when 1 then 'stage1_Newbie'
        when 2 then 'stage2_Browser'
        when 3 then 'stage3_Browser_Toolbar'
        WHEN 4 then 'stage4_Shopper_App'
    end as lifecycle

-- Bad : Do not repeat the column name on which case statement is being implemented
select
    case
        when lifecycle_stage = 1 then 'stage1_Newbie'
        when lifecycle_stage = 2 then 'stage2_Browser'
        when lifecycle_stage = 3 then 'stage3_Browser_Toolbar'
        when lifecycle_stage = 4 then 'stage4_Shopper_App'
    end as lifecycle

-- Bad : when statements are in multiple lines
select
    case
        when lifecycle_stage = 1
            then 'stage1_Newbie'
        when lifecycle_stage = 2
            then 'stage2_Browser'
    ......
    .....
    ...

-- Bad : Case and first When statement is in the same line
select
    case when lifecycle_stage = 1 then 'stage1_Newbie'
        when lifecycle_stage = 1 then 'stage2_Browser'
    ...
    ..
    .
```

## `on` & `where` condition

- Write `on` in its own line if there are multiple joining conditions and keep it indented one level than `select`, `from`, `join`..
- When multiple conditions for joining, all `and`s should be one level deeper than `on`, each condition should end with `and` instead of starting with it
- same logic applies to `where` condition
- Only exception is when there is only one condition, in that case `on` condition can be in the same line as `on`

```sql
-- Good
select distinct
    tx.interest_area,
    tx.interest_score,
    ftb.member_id as user_id,
    tx.order_merchant_id as store_id
from ebates_prod.dw.order_transactions as tx
join ebates_prod.summary.member_ftbs_by_store as ftb
    on
        tx.member_id = ftb.member_id and
        tx.store_id = ftb.store_id and
        tx.column_x = ftb.column_y
where
    tx.member_id = 1234 and
    tx.store_id = 345

-- Good
select distinct
    ...
from ebates_prod.dw.order_transactions as tx
join ebates_prod.summary.member_ftbs_by_store as ftb
    on tx.member_id = ftb.member_id
where tx.member_id = 1234

-- Bad
select distinct
    interest_area,
    interest_score,
    i.member_id as user_id,
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

```sql
-- Good
select
    member_id,
    store_id,
    row_number() over (partition by user_id order by order_date desc) as order_rank
from ebates_prod.dw.shopping_trips

-- Bad
select
    user_id,
    name,
    row_number() over (
        partition by user_id
        order by order_date desc
    ) as order_rank
from ebates_prod.dw.shopping_trips
```

## Query designed with all the guidelines above

```sql
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
    from ebates_prod.dw.order_transactions as tx
    join ebates_prod.summary.member_ftbs_by_store as ftb
        on
            tx.member_id = ftb.member_id and
            tx.store_id = ftb.store_id
    where
        tx.member_id = 1234 and
        tx.store_id = 345 and
        tx.lifecycle_stage != 'lst' and
        ftb.cashback > 10
),

stores as (
    select
        store_id,
        ...,
        ...
    from ebates_prod.dw.stores
)

select
    m.*,
    s.*
from member as m
join stores as s
    on
        m.member_id = s.member_id and
        m.store_id = s.store_id
where m.member_id = 123
```

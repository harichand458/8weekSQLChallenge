https://8weeksqlchallenge.com/case-study-1/

### Solutions

**Query #1**

What is the total amount each customer spent at the restaurant?

````sql
select
    sales.customer_id,
    sum(price)
from dannys_diner.sales
inner join
    dannys_diner.menu on sales.product_id = menu.product_id
group by sales.customer_id order by 1;
````

***Output***

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
**Query #2**
How many days has each customer visited the restaurant?

```sql
select
    customer_id,
    count(distinct(order_date))
from dannys_diner.sales group by customer_id
```
***Output***

| customer_id | visits |
| ----------- | ------ |
| A           | 4      |
| B           | 6      |
| C           | 2      |

---
**Query #3**

What was the first item from the menu purchased by each customer?

```sql
with ordered_sales as (
    select
        customer_id,
        order_date,
        sales.product_id,
        product_name,
        rank() over (partition by customer_id order by order_date) as order_rank
    from dannys_diner.sales inner join dannys_diner.menu on
        sales.product_id = menu.product_id
)
select distinct
    customer_id,
    product_name
from ordered_sales where order_rank = 1
```

***Output***

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

The customer A ordered two items in the first visit: curry and sushi.
Customer B ordered curry and customer C ordered ramem.

---
**Query #4**

What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
select
    product_name,
    count(*)
from dannys_diner.sales inner join dannys_diner.menu on
    sales.product_id = menu.product_id
group by product_name order by 2 desc limit 1
```

***Output***

| product_name | times_purchased |
| ------------ | --------------- |
| ramen        | 8               |


---
**Query #5**

Which item was the most popular for each customer?

```sql
with customer_cte as (
    select
        customer_id,
        product_name,
        count(*) as item_quantity,
        rank() over (
            partition by customer_id
            order by
                count(*) desc
        ) as item_rank
    from
        dannys_diner.sales
    inner join dannys_diner.menu on sales.product_id = menu.product_id
    group by
        customer_id,
        product_name
)

select
    customer_id,
    product_name,
    item_quantity
from customer_cte where item_rank = 1
```

***Output***

| customer_id | product_name | purchases |
| ----------- | ------------ | --------- |
| A           | ramen        | 3         |
| B           | ramen        | 2         |
| B           | curry        | 2         |
| B           | sushi        | 2         |
| C           | ramen        | 3         |


---
**Query #6**

Which item was purchased first by the customer after they became a member?

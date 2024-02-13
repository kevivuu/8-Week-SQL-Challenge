# Case Study 1 - Danny's Diner

## Question 1: What is the total amount each customer spent at the restaurant?
````sql
select customer_id, sum(price) as total_amount_spent
from sales s join menu m on s.product_id = m.product_id
group by customer_id
order by customer_id;
````

#### Steps:
1. JOIN "menu" and "sales" tables
2. Display customer id and total amount spent by each: sum(price)

#### Answer:
| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A           |                 76 |
| B           |                 74 |
| C           |                 36 |

## Question 2: How many days has each customer visited the restaurant?
````sql
select customer_id, count(distinct order_date) as days 
from sales
group by customer_id
order by customer_id;
````

#### Steps:
1. Display customer id and total days visited by each from "sales" table

#### Answer:
| customer_id | days |
| ----------- | ---- |
| A           |    4 |
| B           |    6 |
| C           |    2 |

## Question 3: What was the first item from the menu purchased by each customer?
````sql
with ordered_date as (
	select customer_id, product_name, order_date, 
	dense_rank() over(partition by customer_id order by order_date) as rank
	from sales s left join menu m on s.product_id = m.product_id
)
select customer_id, product_name
from ordered_date
where rank = 1
group by customer_id, product_name;
````

#### Steps:
1. Create CTE "ordered_date" by ranking dates (ascending order) for each customer_id
2. Display customer_id and product_name for every entry with rank = 1 (earliest date for each customer)

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           |        curry |
| A           |        sushi |
| B           |        curry |
| C           |        ramen |

## Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
select product_name, count(*) as times_purchased
from sales s left join menu m on s.product_id = m.product_id
group by product_name
order by times_purchased desc
limit 1;
````

#### Steps:
1. Display product_name and number of times each is purchased
2. Order them (descending order) on times_purchased, then limit to 1 entry

#### Answer:
| product_name | times_purchased |
| ------------ | --------------- |
| ramen        |               8 |

## Question 5: Which item was the most popular for each customer?
````sql
with ordered_date as (
	select s.customer_id, product_name,
	dense_rank() over(partition by s.customer_id order by count(*) desc) as rank
	from sales s 
	left join menu m on s.product_id = m.product_id
	group by s.customer_id, product_name
)
select customer_id, product_name
from ordered_date
where rank = 1
group by customer_id, product_name;
````

#### Steps:
1. Create CTE "ordered_date" by ranking product count (descending order) for each customer_id
2. Display customer_id and product_name for every entry with rank = 1 (most popular product)

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           |        ramen |
| B           |        curry |
| B           |        ramen |
| B           |        sushi |
| C           |        ramen |

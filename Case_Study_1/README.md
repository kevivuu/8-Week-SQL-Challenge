# Case Study 1 - Danny's Diner

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Raw Data
````sql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
````

## Question 1: What is the total amount each customer spent at the restaurant?
````sql
select customer_id, sum(price) as total_amount_spent
from sales s join menu m on s.product_id = m.product_id
group by customer_id
order by customer_id;
````

#### Steps:
1. Join "menu" and "sales" tables
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

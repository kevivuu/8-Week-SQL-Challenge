# Danny's Diner

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

## Question 6: Which item was purchased first by the customer after they became a member?
````sql
with ordered_date as (
	select s.customer_id, product_name, order_date, 
	dense_rank() over(partition by s.customer_id order by order_date) as rank
	from sales s 
	left join menu m on s.product_id = m.product_id
	left join members ms on s.customer_id = ms.customer_id
	where order_date > join_date
)
select customer_id, product_name
from ordered_date
where rank = 1
group by customer_id, product_name;
````

#### Steps:
1. Create CTE "ordered_date" by ranking dates (ascending order) for each customer_id, subject to order dates after join dates
2. Display customer_id and product_name for every entry with rank = 1 (earliest date after join date)

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           |        ramen |
| B           |        sushi |

## Question 7: Which item was purchased just before the customer became a member?
````sql
with ordered_date as (
	select s.customer_id, product_name, order_date, 
	dense_rank() over(partition by s.customer_id order by order_date desc) as rank
	from sales s 
	left join menu m on s.product_id = m.product_id
	left join members ms on s.customer_id = ms.customer_id
		where order_date < join_date
)
select customer_id, product_name
from ordered_date
where rank = 1
group by customer_id, product_name;
````

#### Steps:
Assumption: order dates can only be BEFORE join dates (not the same day)
1. Create CTE "ordered_date" by ranking dates (descending order) for each customer_id, subject to order dates before join dates
2. Display customer_id and product_name for every entry with rank = 1 (latest date before join date)

#### Answer:
| customer_id | product_name |
| ----------- | ------------ |
| A           |        curry |
| A           |        sushi |
| B           |        sushi |

## Question 8: What is the total items and amount spent for each member before they became a member?
````sql
with ordered_rank as(
	select s.customer_id, count(*) as total_item, sum(price) as total_paid,
	from sales s 
	left join menu m on s.product_id = m.product_id
	left join members ms on s.customer_id = ms.customer_id
	where order_date < join_date
	group by s.customer_id
)
select customer_id, total_item, total_paid
from ordered_rank
order by customer_id;
````

#### Steps:
1. Create CTE "ordered_rank" with total_item and total_paid per customer_id, with order date before join date
2. Display customer_id, total_item and total_paid

#### Answer:
| customer_id | total_item | total_paid |
| ----------- | ---------- | ---------- |
| A           |          2 |         25 |
| B           |          3 |         40 |


## Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
with points as (
	select customer_id, product_name,
	case
		when product_name = 'sushi' then price*20
		else price*10
	end as point
	from sales s left join menu m on s.product_id = m.product_id
)
select customer_id, sum(point) as total_point
from points
group by customer_id
order by customer_id;
````

#### Steps:
1. Create CTE "points" where point is price x20 when product is sushi and price x10 otherwise
2. Display customer_id and total_point

#### Answer:
| customer_id | total_point  |
| ----------- | ------------ |
| A           |          860 |
| B           |          940 |
| C           |          360 |


## Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
with points as (
	select s.customer_id, product_name, order_date, join_date, price,
	case
		when order_date between join_date and join_date + interval '7 day' then price * 20
		when product_name = 'sushi' then price * 20
		else price * 10
	end point
	from sales s 
	left join menu m on s.product_id = m.product_id
	join members ms on s.customer_id = ms.customer_id
	where order_date < '2021-02-01'
)
select customer_id, sum(point) as total_point
from points
group by customer_id
order by customer_id;
````

#### Steps:
1. Create CTE "points" where point is price x20 when product is sushi or when order date is in range, and price x10 otherwise
2. Make a constraint of date being before February 2021 for "points"
3. Display customer_id and total_point

#### Answer:
| customer_id | total_point  |
| ----------- | ------------ |
| A           |         1370 |
| B           |          940 |

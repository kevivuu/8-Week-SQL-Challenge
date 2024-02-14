# Part A: Pizza Metrics

## Question 1: How many pizzas were ordered?
````sql
select count(*) as pizza_count
from customer_orders_temp;
````

#### Steps:
1. Count total entries in customer_orders_temp

#### Answer:
| pizza_count |
| ----------- |
|          14 |

## Question 2: How many unique customer orders were made?
````sql
select count(distinct order_id) as order_count
from customer_orders_temp;
````

#### Steps:
1. Count total distinct orders in customer_orders_temp

#### Answer:
| order_count |
| ----------- |
|          10 |

## Question 3: How many successful orders were delivered by each runner?
````sql
select runner_id, count(*) as successful_deliveries
from runner_orders_temp
where cancellation is null
group by runner_id
order by runner_id;
````

#### Steps:
1. Display runner id and total count of orders grouped by runners that are not cancelled

#### Answer:
| runner_id   | successful_deliveries |
| ----------- | --------------------- |
| 1           |                    4  |
| 2           |                    3  |
| 3           |                    1  |

## Question 4: How many of each type of pizza was delivered?
````sql
select pizza_id, count(*) as delivery_count
from customer_orders_temp c
left join runner_orders_temp r
on c.order_id = r.order_id
where cancellation is null
group by pizza_id
order by pizza_id;
````

#### Steps:
1. Join runner and customer tables
2. Display pizza id and total count of orders grouped by pizzas that are not cancelled

#### Answer:
| pizza_id    | delivery_count        |
| ----------- | --------------------- |
| 1           |                    9  |
| 2           |                    3  |

## Question 5: How many Vegetarian and Meatlovers were ordered by each customer?
````sql
select customer_id, pizza_name, count(*) as order_count
from customer_orders_temp c
left join pizza_names pn
on c.pizza_id = pn.pizza_id
group by customer_id, pizza_name
order by customer_id, pizza_name;
````

#### Steps:
1. Left join customer and pizza_names tables
2. Group by customer id and pizza name, and display total count for each group

#### Answer:
<img width="343" alt="Screenshot 2024-02-13 at 17 15 14" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/50f34bad-14af-4d1e-9caa-85f79d71c98b">

## Question 6: What was the maximum number of pizzas delivered in a single order?
````sql
select order_id, count(*) as pizza_count
from customer_orders_temp
group by order_id
order by pizza_count desc
limit 1;
````

#### Steps:
1. Group order_id and display total pizzas per order
2. Order by descending order and limit to 1 entry

#### Answer:
| order_id    | pizza_count  |
| ----------- | ------------ |
| 4           |        3     |

## Question 7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
with changes as (
	select customer_id, order_id,
	case
		when exclusions is null and extras is null then 0
		else count(*)
		end as change_count,
	case
		when exclusions is null and extras is null then count(*)
		else 0
		end as no_change_count
	from customer_orders_temp
	group by customer_id, order_id, exclusions, extras
)
select customer_id, sum(change_count) as changed, sum(no_change_count) as unchanged
from changes c
left join runner_orders_temp r
on c.order_id = r.order_id
where cancellation is null
group by customer_id
order by customer_id;
````

#### Steps:
1. Create CTE "changes" that counts number of changed pizzas and unchanged pizzas for each order
2. Join changes and runner tables
3. Group by customers and constraint on orders that are not cancelled

#### Answer:
<img width="321" alt="Screenshot 2024-02-13 at 17 20 44" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/6647d54c-1c57-4b50-93a5-cdafc89ba0f1">

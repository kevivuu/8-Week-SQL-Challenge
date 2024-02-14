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

## Question 8: How many pizzas were delivered that had both exclusions and extras?
````sql
with changes as (
	select order_id,
	case
		when exclusions is not null and extras is not null then count(*)
		else 0
		end as change_count
	from customer_orders_temp
	group by order_id, exclusions, extras
)
select count(*) as mod_count
from changes c
left join runner_orders_temp r
on c.order_id = r.order_id
where cancellation is null
and change_count != 0;
````

#### Steps:
1. Create CTE "changes" that counts number of pizzas that meets requirement for each order
2. Join changes and runner tables
3. Group by customers and constraint on orders that are not cancelled and has more than 1 pizza that meets requirement
4. Display number of pizzas as "mod_count"
   
#### Answer:
| mod_count   |
| ----------- |
|          1  |


## Question 9: What was the total volume of pizzas ordered for each hour of the day?
````sql
select date_part('hour', order_time) as hour, count(*) as pizza_ordered
from customer_orders_temp
group by hour
order by hour;
````

#### Steps:
1. Use **date_part** function (interval = 'hour') to separate the date from each entry
2. Display number of pizza orders for each hour of the day

#### Answer:
<img width="279" alt="Screenshot 2024-02-13 at 17 28 46" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/29555da1-ddbc-4481-9e23-f0d2e2008f87">

## Question 10: What was the volume of orders for each day of the week?
````sql
with days as (
	select date_part('dow', order_time) as day, count(*) as pizza_volume
	from customer_orders_temp
	group by day
	order by day
)
select 
case
	when day = 1 then 'Monday'
	when day = 2 then 'Tuesday'
	when day = 3 then 'Wednesday'
	when day = 4 then 'Thursday'
	when day = 5 then 'Friday'
	when day = 6 then 'Saturday'
	else 'Sunday'
end as day,
pizza_volume
from days;
````

#### Steps:
1. Use date_part function (interval = 'dow') to separate the day of the week (dow) from each entry
2. Create a CTE "days" that has the day id and pizza volume per each day
3. Assign the texts for each day ids

#### Answer:
<img width="239" alt="Screenshot 2024-02-13 at 17 32 44" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/9127cbee-5fd8-4233-a7da-a40dfa260cd8">

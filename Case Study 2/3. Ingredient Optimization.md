# Part C: Ingredient Optimization

## Cleaning "pizza_recipes" table
````sql
create temp table pizza_recipes_temp as (
	select 
		pizza_id, 
		cast(trim(unnest(string_to_array(toppings, ','))) as int) as toppings
	from pizza_recipes
);
````
**Actions:** split strings into array of values and convert them into integers

#### Temp table: pizza_recipes_temp
<img width="208" alt="Screenshot 2024-02-15 at 13 40 36" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/c7a2b733-6490-4b4d-a9bc-40b69438d2bd">

## Question 1: What are the standard ingredients for each pizza?
````sql
select pizza_id, topping_name
from pizza_recipes_temp r
join pizza_toppings t
on r.topping_id = t.topping_id
order by pizza_id, topping_name;
````

#### Steps:
1. Display pizza id and name of toppings by joining toppings and recipes tables

#### Answer:
<img width="237" alt="Screenshot 2024-02-15 at 13 48 22" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/4e9c5f28-83a5-4db5-8d22-4435398df985">

## Create "extras_temp" table
````sql
create temp table extras_temp as (
	select order_id, customer_id, 
	cast(trim(unnest(string_to_array(extras, ','))) as int) as extra_id
	from customer_orders_temp
	order by extra_id, order_id, customer_id
);
````
**Actions:** split strings into array of values and convert them into integers

#### Temp table: extras_temp
<img width="306" alt="Screenshot 2024-02-15 at 14 43 10" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/bf54594e-3561-475e-9994-8e32e91a5f39">

## Question 2: What was the most commonly added extra?
````sql
select topping_name, count(*)
from extras_temp e
join pizza_toppings t
on e.extra_id = t.topping_id
group by topping_name
order by count(*) desc
limit 1;
````

#### Steps:
1. Display topping name and count of each from extras_temp
2. Order by count (descending) then limit to 1 for most common

#### Answer:
| topping_name | count                 |
| ------------ | --------------------- |
|    Bacon     |             4         |

## Create "exclus_temp" table
````sql
create temp table exclus_temp as (
	select order_id, customer_id, 
	cast(trim(unnest(string_to_array(exclusions, ','))) as int) as exclu_id
	from customer_orders_temp
	order by exclu_id, order_id, customer_id
);
````
**Actions:** split strings into array of values and convert them into integers

#### Temp table: extras_temp
<img width="307" alt="Screenshot 2024-02-15 at 15 01 25" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/43712749-cc50-4059-88ed-911561e8242f">

## Question 3: What was the most commonly added exclusion?
````sql
select topping_name, count(*)
from exclus_temp e
join pizza_toppings t
on e.exclu_id = t.topping_id
group by topping_name
order by count(*) desc
limit 1;
````

#### Steps:
1. Display topping name and count of each from exclus_temp
2. Order by count (descending) then limit to 1 for most common

#### Answer:
| topping_name | count                 |
| ------------ | --------------------- |
|    Cheese    |             4         |

## Question 4: What was the average distance travelled for each customer?
````sql
with distances as (
	select customer_id, r.order_id, 
	cast(distance_km as real) as distance
	from runner_orders_temp r 
	left join customer_orders_temp c
	on r.order_id = c.order_id
	where distance_km is not null
	group by customer_id, r.order_id, distance_km
)
select customer_id, 
concat(cast(round(avg(distance)::numeric, 1) as text), ' km') as avg_distance
from distances
group by customer_id
order by customer_id;
````

#### Steps:
1. Join customer and runner tables
2. Modify value types to find average distance for each customer

#### Answer:
<img width="252" alt="Screenshot 2024-02-14 at 00 21 03" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/dcb5a156-2b0c-4e96-8b9c-9fc829a82c5a">

## Question 5: What was the difference between the longest and shortest delivery times for all orders?
````sql
with times as (
	select r.order_id, count(*) as pizza_count, order_time, pickup_time, 
	cast(pickup_time as timestamp without time zone) - order_time as timelapse
	from runner_orders_temp r 
	left join customer_orders_temp c
	on r.order_id = c.order_id
	where order_time is not null and pickup_time is not null
	group by r.order_id, order_time, pickup_time
)
select max(timelapse) - min(timelapse) as time_diff
from times;
````

#### Steps:
1. Calculate time difference similarly to q2
2. Calculate the difference between longest (max) and shortest (min) timelapse

#### Answer:
| time_diff   |
| ----------- |
| 00:19:15    |

## Question 6: What was the average speed for each runner for each delivery and do you notice any trend for these values?
````sql
with speeds as (
	select runner_id, count(*) as pizza_count,
	cast(distance_km as real)/ cast(duration_min as real)*60 as speed
	from runner_orders_temp r 
	left join customer_orders_temp c
	on r.order_id = c.order_id
	where distance_km is not null
	group by runner_id, distance_km, duration_min
)
select runner_id, pizza_count, 
concat(cast(round(avg(speed)::numeric, 2) as text), ' km/h') as speed
from speeds
group by runner_id, pizza_count
order by runner_id, pizza_count;
````

#### Steps:
1. Create CTE "speeds" which calculates km/h speed for each runner
2. Display speed for each runner sorted by number of pizzas per order

#### Answer:
<img width="313" alt="Screenshot 2024-02-14 at 00 24 45" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/c493aabc-6007-4b9c-a894-7e3da37ffbf6">

**Analysis**: Runner 2 is fastest with 1 pizza. There is no clear relationship across runners with increasing pizzas per order.

## Question 7: What is the successful delivery percentage for each runner?
````sql
with successes as (
	select runner_id, count(*) as success
	from runner_orders_temp
	where cancellation is null
	group by runner_id
),
pcts as (
	select s.runner_id, (cast(success as real)/ cast(count(*) as real))*100 as pct
	from successes s join runner_orders_temp r
	on s.runner_id = r.runner_id
	group by s.runner_id, success
)
select runner_id, concat(cast(pct as text), '%') as success_rate
from pcts
````

#### Steps:
1. Create CTE "successes" to show successful deliveries per runner
2. Create CTE "pcts" to calculate success rate
3. Display each runner and their success rate

#### Answer:
| runner_id   | success_rate          |
| ----------- | --------------------- |
| 1           |                 100%  |
| 2           |                  75%  |
| 3           |                  50%  |

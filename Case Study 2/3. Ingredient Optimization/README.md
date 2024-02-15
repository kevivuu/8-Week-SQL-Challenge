# Part C: Ingredient Optimization

## Question 1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
````sql
select date_part('week', registration_date) as week, count(*) as runner_count
from runners
group by week
order by week;
````

#### Steps:
1. Display weeks since 2021-01-01 and count registrations per each week

#### Answer:
| week        | runner_count          |
| ----------- | --------------------- |
| 1           |                    2  |
| 2           |                    2  |

## Question 2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
````sql
with times as (
	select r.order_id, runner_id, order_time, pickup_time, 
	cast(pickup_time as timestamp without time zone) - order_time as timelapse
	from runner_orders_temp r 
	left join customer_orders_temp c
	on r.order_id = c.order_id
	where order_time is not null and pickup_time is not null
	group by r.order_id, runner_id, order_time, pickup_time
)
select runner_id, date_trunc('second', avg(timelapse)) as avg_time
from times
group by runner_id
order by runner_id;
````

#### Steps:
1. Create CTE "times" which calculates the difference between pickup_time and order_time
2. Make sure all null values are not included in "times"
3. Display runners and the average time for each

#### Answer:
| runner_id   | avg_time              |
| ----------- | --------------------- |
| 1           |             00:14:19  |
| 2           |             00:20:00  |
| 3           |             00:10:28  |

## Question 3: Is there any relationship between the number of pizzas and how long the order takes to prepare?
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
select pizza_count, date_trunc('second', avg(timelapse)) as avg_time
from times
group by pizza_count
order by pizza_count;
````

#### Steps:
1. Calculate time difference similarly to q2
2. Find the average time for each number of pizzas per order

#### Answer:
| pizza_count | avg_time              |
| ----------- | --------------------- |
| 1           |             00:12:21  |
| 2           |             00:18:22  |
| 3           |             00:29:17  |

**Analysis**: Positive relationship (more pizzas take longer to prepare)

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

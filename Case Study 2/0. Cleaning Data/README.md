
## Cleaning "customer_order" table
````sql
create temp table customer_orders_temp as(
	select order_id, customer_id, pizza_id,
	case
		when exclusions like '%null%' or exclusions is null or trim(exclusions) = ''
		then null
		else exclusions
	end as exclusions,
	case
		when extras like '%null%' or extras is null or trim(extras) = ''
		then null
		else extras
	end as extras,
	order_time
	from customer_orders
);
````
**Actions:** turn all blank or null values into null

#### Temp table: customer_order_temp
<img width="750" alt="Screenshot 2024-02-13 at 12 28 13" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/04c7f691-2771-42c9-a60c-b27aaad029d4">

## Cleaning "runner_order" table
````sql
create temp table runner_orders_temp as(
	select order_id, runner_id,
	case
		when pickup_time like '%null%' then null
		else pickup_time
		end as pickup_time,
	case
		when distance like '%null%' then null
		when distance like '%km' then trim(trim('km' from distance))
		else distance
		end as distance_km,
	case
		when duration like '%null%' then null
		when duration like '%minutes' then trim(trim('minutes' from duration))
		when duration like '%minute' then trim(trim('minute' from duration))
		when duration like '%mins' then trim(trim('mins' from duration))
		else duration
		end as duration_min,
	case
		when trim(cancellation) in ('null', '') then null
		else cancellation
		end as cancellation
	from runner_orders
);
````
**Actions:** turn all blank or null values into null; trim excess strings from int/ real values

#### Temp table: runner_order_temp
<img width="751" alt="Screenshot 2024-02-13 at 12 40 58" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/1da95acb-6a16-48c6-9060-1a656d4f2c1e">




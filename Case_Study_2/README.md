# Case Study 2 - Pizza Runner
![2](https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/98ed9fb4-b01e-4aa8-b47e-c16e6dd53e96)
## Introduction
Did you know that over **115 million kilograms** of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

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



# Case Study 2 - Pizza Runner

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

#### Temp table: customer_order_temp
<img width="750" alt="Screenshot 2024-02-13 at 12 28 13" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/04c7f691-2771-42c9-a60c-b27aaad029d4">


# Case Study 1 - Danny's Diner
![1](https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/7686521d-c31d-4517-8b24-bddadd5ea43c)

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

## Entity Relationship Diagram
<img width="704" alt="Screenshot 2024-02-13 at 17 44 59" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/9257cf20-c34f-413d-adfa-b6f82c1b8682">

## Table 1: sales
<img width="383" alt="Screenshot 2024-02-13 at 17 59 10" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/b74b62be-19fe-462a-ab6c-4ab8da40db8d">

## Table 2: menu
<img width="348" alt="Screenshot 2024-02-13 at 17 59 39" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/38a1533a-80e4-44f2-824e-20c590b1a8d5">

## Table 3: members
<img width="259" alt="Screenshot 2024-02-13 at 18 00 11" src="https://github.com/kevivuu/8-Week-SQL-Challenge/assets/155116890/6c750e79-b7ce-433a-b2b2-242c642ac324">

# DannysDiner
Week 1 of the 8 week course challenge.

This is part of the 8 week SQL Challenge, by Danny Ma. You can read about his challenge here: https://8weeksqlchallenge.com/case-study-1/

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
  
  
    QUESTION 1:
    What is the Total Amount each customer spent at the restaurant:
    
    Answer: A	76
            B	74
            C	36

--First, I inner joined tables 'sales' and 'menu':
SELECT customer_iD, price
FROM dbo.sales
INNER JOIN dbo.menu 
ON dbo.sales.product_id = dbo.menu.product_id

--Then, I used SUM function with GROUP BY:
SELECT customer_iD, SUM (price) AS Total_spent 
FROM dbo.sales
INNER JOIN dbo.menu 
ON dbo.sales.product_id = dbo.menu.product_id 
GROUP BY customer_iD
    
  
  
    QUESTION 2:
    How many days has each customer visited the restaurant:
    
    Answer: A	6
            B	6
            C	3


-- I used the functions COUNT and GROUP BY:
SELECT customer_id, COUNT(order_date) AS Number_of_visits
  FROM sales
  GROUP BY customer_id

  
  
      QUESTION 3:
      What was the first item from the menu purchased by each customer
      
      Answer: 2021-01-01	A	sushi
              2021-01-01	A	curry
              2021-01-01	B	curry
              2021-01-01	C	ramen


  -- First, I joined tables sales and menu:

  SELECT customer_id, product_name, order_date
  FROM dbo.sales
  INNER JOIN dbo.menu
  ON dbo.sales.product_id = dbo.menu.product_id

  --Then, I organized by order date and customer_id:

  SELECT order_date, customer_id, product_name 
  FROM dbo.sales
  INNER JOIN dbo.menu
  ON dbo.sales.product_id = dbo.menu.product_id
  --WHERE order_date = '2021-01-01'
  ORDER BY order_date 

  --After finding out the first date, I added a WHERE clause to make the results clearer:

  SELECT order_date, customer_id, product_name 
  FROM dbo.sales
  INNER JOIN dbo.menu
  ON dbo.sales.product_id = dbo.menu.product_id
  WHERE order_date = '2021-01-01'
  ORDER BY order_date
  
  
  
      QUESTION 4:
      What is the most purchased item on the menu and how many times was it purchased by all customers
      
      Answer: ramen	8
              curry	4
              sushi	3
      


-- I used the joined tables 'sales' and 'menu':
  SELECT customer_id, product_name
  FROM dbo.sales
  INNER JOIN dbo.menu
  ON dbo.sales.product_id = dbo.menu.product_id
  --GROUP BY product_name
  
  
--Then, I used the COUNT function, combined with GROUP BY and ORDER BY:
  SELECT product_name, COUNT (product_name) AS Times_purchased
  FROM dbo.sales
  INNER JOIN dbo.menu
  ON dbo.sales.product_id = dbo.menu.product_id
  GROUP BY product_name
  ORDER BY Times_purchased DESC
  
  
  
  
  
  
  
  
  
  

  

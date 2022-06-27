# DannysDiner
Week 1 of the 8 week course challenge.

This is part of the 8 week SQL Challenge, by Danny Ma. You can find his challenge here: https://8weeksqlchallenge.com/case-study-1/

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
  
  
--Then, I used the COUNT function, combined with GROUP BY and ORDER BY:

        SELECT product_name, COUNT (product_name) AS Times_purchased
        FROM dbo.sales
        INNER JOIN dbo.menu
        ON dbo.sales.product_id = dbo.menu.product_id
        GROUP BY product_name
        ORDER BY Times_purchased DESC
  
  
  
  
  QUESTION 5:
  Which item was the most popular for each customer
  
        Answer:customer_id	product_name	item_bought_count
                  A	            ramen	          3
                  B	            sushi	          2
                  B	            curry	          2
                  B	            ramen	          2
                  C	            ramen	          3
  
  
  -- Using RANK and PARTITION BY Functions:
  
        with rank as(
        select sales.customer_id,
        menu.product_name,
        count(*) as order_count,
        dense_rank() over(partition by sales.customer_id
        order by count(sales.customer_id)desc) as rank
        FROM dbo.sales
        INNER JOIN dbo.menu
        ON dbo.sales.product_id = dbo.menu.product_id
        group by sales.customer_id,menu.product_name)
   
        Select Customer_id,Product_name,order_Count
        From rank
        where rank = 1
  
  
  QUESTION 6:
  Which item was purchased first by the customer after they became a member
  
        Answer:
  
  
  
  -- First, I joined the three tables, since the common key was customer_id on both members and sales table. 
  
  -- The product information was on the menu table.


        SELECT dbo.sales.customer_id, dbo.sales.order_date, dbo.menu.product_name, dbo.members.join_date
        FROM dbo.sales
        JOIN dbo.menu
        ON dbo.sales.product_id = dbo.menu.product_id
	    JOIN dbo.members
	    ON dbo.members.customer_id = dbo.sales.customer_id
  
  -- Then, I used ROW_NUMBER 
  
 	 SELECT ROW_NUMBER () OVER (PARTITION BY dbo.menu.product_name ORDER BY dbo.sales.customer_id) AS row_id, 
	dbo.sales.customer_id, dbo.sales.order_date, dbo.menu.product_name 
	FROM dbo.sales 
    JOIN dbo.menu
    ON dbo.sales.product_id = dbo.menu.product_id
    JOIN dbo.members
    ON dbo.members.customer_id = dbo.sales.customer_id
	WHERE dbo.sales.order_date > dbo.members.join_date
	ORDER BY row_id
	
-- However, this got me this result:

	row_id	customer_id	order_date	product_name
	1		A	2021-01-10	ramen
	1		B	2021-01-11	sushi
	2		A	2021-01-11	ramen
	3		A	2021-01-11	ramen
	4		B	2021-01-16	ramen
	5		B	2021-02-01	ramen
  
  
  -- And I got an error message when trying to filter by row_id.
  
  -- That's when I got the TOP 1 function:
  
  	SELECT top 1 WITH ties
	dbo.sales.customer_id, dbo.menu.product_name 
	FROM dbo.sales 
    	JOIN dbo.menu
    	ON dbo.sales.product_id = dbo.menu.product_id
    	JOIN dbo.members
    	ON dbo.members.customer_id = dbo.sales.customer_id
	WHERE dbo.sales.order_date > dbo.members.join_date 
	ORDER BY ROW_NUMBER () OVER (PARTITION BY dbo.menu.product_name ORDER BY dbo.sales.customer_id) 
	
-- Which gave me the result I was looking for:

	customer_id	product_name
		A	ramen
		B	sushi
  
  

  

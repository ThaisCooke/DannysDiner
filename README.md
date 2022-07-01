# DannysDiner
Week 1 of the 8 week SQL challenge.

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
  What is the Total Amount each customer spent at the restaurant?
    
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
  How many days has each customer visited the restaurant?
    
    Answer: A	6
            B	6
            C	3


-- I used the functions COUNT and GROUP BY:
    SELECT customer_id, COUNT(order_date) AS Number_of_visits
      FROM sales
      GROUP BY customer_id

  
  
    QUESTION 3:
    What was the first item from the menu purchased by each customer?
      
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
  What is the most purchased item on the menu and how many times was it purchased by all customers?
      
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
  Which item was the most popular for each customer?
  
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
  Which item was purchased first by the customer after they became a member?
  
        Answer:
	
  	customer_id	product_name
		A	curry
		B	sushi
  
  -- First, I joined the three tables, since the common key was customer_id on both members and sales table. 
  
  -- The product information was on the menu table.


        SELECT dbo.sales.customer_id, dbo.sales.order_date, dbo.menu.product_name, dbo.members.join_date
        FROM dbo.sales
        JOIN dbo.menu
        ON dbo.sales.product_id = dbo.menu.product_id
	    JOIN dbo.members
	    ON dbo.members.customer_id = dbo.sales.customer_id
  
  -- Then, I used ROW_NUMBER in a CTE:
  
 	 WITH cte AS (SELECT ROW_NUMBER () OVER (PARTITION BY dbo.members.customer_id ORDER BY dbo.menu.product) AS row_id, 
	dbo.sales.customer_id, dbo.sales.order_date, dbo.menu.product_name 
	FROM dbo.sales 
    JOIN dbo.menu
    ON dbo.sales.product_id = dbo.menu.product_id
    JOIN dbo.members
    ON dbo.members.customer_id = dbo.sales.customer_id
	WHERE dbo.sales.order_date >= dbo.members.join_date)

	SELECT * 
	FROM cte 
	WHERE row_id = 1

  
  
  
  
QUESTION 7:
Which item was purchased just before the customer became a member?

	Answer:
	
	customer_id	product_name
		A	sushi
		B	curry

	
	
-- This result can be accomplished by using CTE, modifying the PARTITION BY and switching the sign > to < when comparing the order date vs join date

	WITH cte AS (SELECT ROW_NUMBER () OVER (PARTITION BY dbo.members.customer_id ORDER BY dbo.sales.order_date) AS row_id, 
	dbo.sales.customer_id, dbo.sales.order_date, dbo.menu.product_name 
	FROM dbo.sales 
    	JOIN dbo.menu
    	ON dbo.sales.product_id = dbo.menu.product_id
    	JOIN dbo.members
    	ON dbo.members.customer_id = dbo.sales.customer_id
	WHERE dbo.sales.order_date < dbo.members.join_date)

	SELECT * 
	FROM cte 
	WHERE row_id = 1
	
	

QUESTION 8:

What is the total items and amount spent for each member before they became a member?

	Answer:
	
	customer_id	number_of_items	   Total
		A	  2	             25
		B	  2	             40
		


-- I used COUNT DISTINCT and SUM to get to the answer:

	SELECT  dbo.sales.customer_id, COUNT (DISTINCT dbo.menu.product_name) AS number_of_items, SUM (dbo.menu.price) AS Total
	FROM dbo.sales
    	JOIN dbo.menu
    	ON dbo.sales.product_id = dbo.menu.product_id
    	JOIN dbo.members
    	ON dbo.members.customer_id = dbo.sales.customer_id
	WHERE dbo.sales.order_date < dbo.members.join_date
	GROUP BY   dbo.sales.customer_id


QUESTION 9:

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

	Answer:

	customer_id	Points
		A	860
		B	940
		C	360
		
		
-- Using SUM with CASE function:

	SELECT
 	dbo.sales.customer_id,
  	SUM(
    	CASE
      	WHEN dbo.menu.product_name = 'sushi' THEN 20 * price
      	ELSE 10 * PRICE
    	END
  	) AS Points
	FROM dbo.sales
    	JOIN dbo.menu
    	ON dbo.sales.product_id = dbo.menu.product_id
	GROUP BY
  	dbo.sales.customer_id
	ORDER BY
  	dbo.sales.customer_id;
	

QUESTION 10:

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

	Answer:
	customer_id	Points
		A	1370
		B	940
	

-- Same query as the last question, using SUM with CASE function, but this time limiting the date:

	SELECT
	dbo.sales.customer_id,
	SUM(
		CASE
  		WHEN dbo.menu.product_name = 'sushi' THEN 20 * price
		WHEN order_date BETWEEN '2021-01-07' AND '2021-01-14' THEN 20 * price
  		ELSE 10 * PRICE
		END
	) AS Points
	FROM dbo.sales
    	JOIN dbo.menu
    	ON dbo.sales.product_id = dbo.menu.product_id
    	JOIN dbo.members
    	ON dbo.members.customer_id = dbo.sales.customer_id
	GROUP BY
	dbo.sales.customer_id
	ORDER BY
	dbo.sales.customer_id;


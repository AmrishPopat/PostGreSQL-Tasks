**Schema (PostgreSQL v12)**

    CREATE TABLE customers (
    id INT PRIMARY KEY
    , first_name VARCHAR(200)
    , city VARCHAR(200)
    , email VARCHAR(200)
    );
    
    CREATE TABLE orders (
    id INT PRIMARY KEY
    , customer_id INT
    , created_at DATE
    , amount_paid FLOAT
    );
    
    CREATE TABLE cities (
    city VARCHAR(200)
    );

---

**Query #1**

    INSERT INTO customers (id, first_name, city, email ) 
    VALUES (1, 'Matt', 'London', 'matt@mymail.com'),
    (2, 'Rob', 'Manchester', 'rob@mymail.com'),
    (3, 'Lisa', 'Birmingham', 'lisa@mymail.com'),
    (4, 'Monica', 'London', 'monica@mymail.com'),
    (5, 'Gary', 'LONDON', 'gary@mymail.com'),
    (6, 'Darren', 'London', 'darren@mymail.com'),
    (7, 'Debbie', 'Manchester', 'debbie@mymail.com'),
    (8, 'Nick', 'Birmingham', 'nick@mymail.com'),
    (9, 'Kristie', 'London', 'kristie@mymail.com'),
    (10, 'Steve', 'LONDON','steve@mymail.com'),
    (11, 'Ben', 'London', 'ben@mymail.com'),
    (12, 'Annie', 'Manchester', 'annie@mymail.com'),
    (13, 'Albert', 'Birmingham', 'albert@mymail.com'),
    (14, 'Maya', 'London', 'maya@mymail.com'),
    (15, 'Zak', 'LONDON', 'zak@mymail.com'),
    (16, 'Bob', 'London', 'bob@mymail.com'),
    (17, 'Jane', 'Manchester', 'jane@mymail.com'),
    (18, 'Sarah', 'Birmingham', 'sarah@mymail.com'),
    (19, 'Stu', 'London', 'stu@mymail.com'),
    (20, 'Harry', 'LONDON', 'harry@mymail.com'),
    (21, 'Pete', 'London', 'pete@mymail.com'),
    (22, 'Jo', 'Manchester', 'jo@mymail.com'),
    (23, 'Tiya', 'Birmingham', 'tiya@mymail.com'),
    (24, 'David', 'London', 'david@mymail.com'),
    (25, 'Kat', 'LONDON', 'kat@mymail.com'),
    (26, 'Ram', 'LONDON', 'ram@mymail.com');

There are no results to be displayed.

---
**Query #2**

    INSERT INTO orders 
    (id, customer_id, created_at, amount_paid) 
    VALUES 
    (201,18,'2020-02-03',2710.98),
    (202,8,'2020-01-21',349.85),
    (203,11,'2020-03-01',1421.82),
    (204,1,'2020-02-24',372.43),
    (205,4,'2020-01-25',1641.57),
    (206,14,'2020-03-17',2673.42),
    (207,24,'2020-02-05',2709.82),
    (208,15,'2020-01-27',2520.3),
    (209,9,'2020-02-23',2521.54),
    (210,4,'2020-02-17',767.49),
    (211,20,'2020-02-26',1640.45),
    (212,21,'2020-02-19',1244.22),
    (213,5,'2020-03-16',1683.41),
    (214,9,'2020-03-03',641.14),
    (215,5,'2020-03-11',642.62),
    (216,7,'2020-01-11',152.02),
    (217,18,'2020-03-12',2589.08),
    (218,13,'2020-02-08',1639.02),
    (219,24,'2020-02-10',611.18),
    (220,17,'2020-02-01',1402.14),
    (221,4,'2020-02-18',762.64),
    (222,25,'2020-03-08',2315.17),
    (223,15,'2020-01-29',1687.58),
    (224,20,'2020-02-14',865.24),
    (225,13,'2020-02-13',2240.2),
    (226,25,'2020-01-15',1296.84),
    (227,9,'2020-03-14',2348.64),
    (228,14,'2020-02-02',1388.5),
    (229,2,'2020-02-20',1918.84),
    (230,19,'2020-02-07',260.08),
    (231,11,'2020-02-09',226.15),
    (232,4,'2020-03-04',423.13),
    (233,17,'2020-03-03',464.43),
    (234,25,'2020-03-15',2330.39),
    (235,15,'2020-03-06',1085.36),
    (236,21,'2020-01-19',2160.09),
    (237,22,'2020-01-25',1831.01),
    (238,22,'2020-02-07',2458.05),
    (239,13,'2020-01-18',2089.71),
    (240,4,'2020-03-04',2161.44),
    (241,26,'2020-03-04',2161.44);

There are no results to be displayed.

---
**Query #3**

    /* Task A*/
    SELECT count(id) 
    FROM customers 
    WHERE LOWER(city) = LOWER('LONDON')  AND id IN 
    (
      	SELECT customer_id FROM orders 
    	GROUP BY customer_id 
    	HAVING COUNT(*) > 1
    );

| count |
| ----- |
| 10    |

---
**Query #4**

    /* Task B*/              
    SELECT 
    	date_trunc('week', created_at) AS week,  
    	COUNT(id) AS order_count, 
        COUNT(id) - COUNT(DISTINCT(customer_id)) AS cumulative_count_id
    FROM orders
    WHERE created_at >= date_trunc('week', CURRENT_TIMESTAMP - interval '4 weeks')  
    AND created_at < date_trunc('week', CURRENT_TIMESTAMP) 
    GROUP BY week
    ORDER BY week;

| week                     | order_count | cumulative_count_id |
| ------------------------ | ----------- | ------------------- |
| 2020-02-17T00:00:00.000Z | 5           | 1                   |
| 2020-02-24T00:00:00.000Z | 3           | 0                   |
| 2020-03-02T00:00:00.000Z | 7           | 1                   |
| 2020-03-09T00:00:00.000Z | 4           | 0                   |

---
**Query #5**

    /* Task C*/ 
    SELECT month, average_revenue, 
    (average_revenue - lag(average_revenue) over (ORDER BY month))*100/average_revenue 
    	AS monthly_revenue_difference
    FROM (
      SELECT my_month AS month, 
      AVG(first_order_amount) AS average_revenue
      FROM
      (
        SELECT my_month, first_order_amount 
        FROM
        ( 
          SELECT my_month, customer_id, id,
          FIRST_VALUE(amount_paid) 
          OVER
          (  PARTITION BY my_month, customer_id
              ORDER BY created_at
          ) first_order_amount
          FROM 
          (
            SELECT 
              date_trunc('month', created_at) AS my_month, 
              created_at, 
              customer_id, 
              amount_paid, 
              id FROM orders
          ) AS orders_new   
          ORDER BY created_at
        ) AS orders_new2 
        GROUP BY my_month, first_order_amount
      ) AS orders_new3 
      GROUP BY my_month 
      ORDER BY my_month
    ) AS orders_new4;

| month                    | average_revenue    | monthly_revenue_difference |
| ------------------------ | ------------------ | -------------------------- |
| 2020-01-01T00:00:00.000Z | 1505.17375         |                            |
| 2020-02-01T00:00:00.000Z | 1463.1785714285713 | -2.8701335155849566        |
| 2020-03-01T00:00:00.000Z | 1441.761           | -1.485514688535157         |

---
**Query #6**

    /* Task D - 3 Results */
    INSERT INTO cities (city ) VALUES 
    ('London'),
    ('Manchester'),
    ('Birmingham'),
    ('London'),
    ('LONDON'),
    ('London'),
    ('Manchester'),
    ('Birmingham'),
    ('London'),
    ('LONDON'),
    ('London'),
    ('Manchester'),
    ('Birmingham'),
    ('London'),
    ('LONDON'),
    ('London'),
    ('Manchester'),
    ('Birmingham'),
    ('London'),
    ('LONDON'),
    ('London'),
    ('Manchester'),
    ('Birmingham'),
    ('London'),
    ('LONDON'),
    ('LONDON');

There are no results to be displayed.

---
**Query #7**

    SELECT DISTINCT city FROM cities;

| city       |
| ---------- |
| Birmingham |
| LONDON     |
| Manchester |
| London     |

---
**Query #8**

    SELECT DISTINCT ON (city) * FROM cities;

| city       |
| ---------- |
| Birmingham |
| LONDON     |
| London     |
| Manchester |

---
**Query #9**

    SELECT city FROM cities GROUP BY city;

| city       |
| ---------- |
| Birmingham |
| LONDON     |
| Manchester |
| London     |

---


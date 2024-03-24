# SQL Intermediate to Advanced
Here is my SQL project

# Notes :
Dataset from Brazilian ecommerce public dataset of orders made at Olist Store
ERD table : https://drive.google.com/file/d/1x2yzEp44tZQp6n_wWhaYC3969zsDysyL/view?usp=sharing
Skill : Syntax, Date Function, Join, Agregate Function, Date Function, Multi Table Join


## Determine the number of unique customers who delivered in May 2018
SELECT COUNT(DISTINCT customer_id)
FROM 'Brazilian_Ecommerce.orders.dataset'
WHERE order_status = 'delivered' 
AND DATE TRUNC(order_purchase_timestamp,MONTH) = '2018-05-01'
ORDER BY 1

## City that have the highest of cancel order in Jan 2018 (Using Total of Order)
SELECT 
  c.customer_city as city
  , COUNT(o.order_id) as total_order
FROM `fsda-400313.Brazilian_Ecommerce.customers_dataset` as c
INNER JOIN `fsda-400313.Brazilian_Ecommerce.orders_dataset` as o
ON c.customer_id = o.customer_id
WHERE o.order_status like '%canceled%'
AND EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018
AND EXTRACT(MONTH FROM o.order_purchase_timestamp) = 1
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

## Month in the year 2017 had the lowest total order performance for the Sao Paulo customer city (consider total delivered orders and focusing on the month of purchased order)

SELECT
  purchase_month,
  total_orders
FROM (
  SELECT
    FORMAT_TIMESTAMP('%B', DATE_TRUNC(o.order_purchase_timestamp, MONTH)) AS purchase_month,
    COUNT(o.order_id) AS total_orders
  FROM
    fsda-400313.Brazilian_Ecommerce.orders_dataset o
    JOIN fsda-400313.Brazilian_Ecommerce.customers_dataset c 
    ON o.customer_id = c.customer_id
  WHERE
    c.customer_city = 'sao paulo'
    AND EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2017
    AND o.order_status = 'delivered'
  GROUP BY
    purchase_month
) subquery
ORDER BY
  total_orders ASC
LIMIT 10;

## Which of the following SQL script to retrieve the category with the highest number of buyers (use unique user) who made purchases on our platform during the year 2018?
SELECT t2.product_category_name, 
  COUNT(DISTINCT t3.customer_id) total_customer 
FROM `fsda-400313.Brazilian_Ecommerce.order_items_dataset` t1 
  LEFT JOIN `fsda-400313.Brazilian_Ecommerce.products_dataset` t2 ON t2.product_id = t1.product_id 
  LEFT JOIN `fsda-400313.Brazilian_Ecommerce.orders_dataset` t3 ON t3.order_id = t1.order_id WHERE t3.order_status = 'delivered' 
AND DATE(order_purchase_timestamp) >= '2018-01-01' 
AND DATE(order_purchase_timestamp) < '2019-01-01' 
GROUP BY 1 
ORDER BY 2 DESC

## Week and category that had the highest total purchased users (Considering the delivered orders that were purchased in the year 2018)
SELECT
  product.product_category_name
  , COUNT(DISTINCT orders.customer_id) AS total_purchase_user
  , EXTRACT(ISOWEEK FROM orders.order_purchase_timestamp) AS iso_week
  , orders.order_status AS status
FROM `fsda-400313.Brazilian_Ecommerce.order_items_dataset` AS item
JOIN `fsda-400313.Brazilian_Ecommerce.orders_dataset` AS orders
ON item.order_id = orders.order_id
JOIN `fsda-400313.raBzilian_Ecommerce.products_dataset` AS product
ON item.product_id = product.product_id
WHERE EXTRACT(YEAR from orders.order_purchase_timestamp) = 2018
AND order_status = 'delivered'
GROUP BY 1,3,4
ORDER BY 2 DESC 

##  Identify which seller city and product category name in english that have the highest total of order item price (Based on delivered order status and using order purchase date in the year of 2017)

SELECT 
  t3.seller_city AS seller_city
  , t5.product_category_name_english AS product_category
  , SUM(t1.price) AS total_order_price
FROM `fsda-400313.Brazilian_Ecommerce.order_items_dataset` AS t1
LEFT JOIN `fsda-400313.Brazilian_Ecommerce.orders_dataset` AS t2
ON t1.order_id = t2.order_id
LEFT JOIN `fsda-400313.Brazilian_Ecommerce.sellers_dataset` AS t3
ON t1.seller_id = t3.seller_id
LEFT JOIN `fsda-400313.Brazilian_Ecommerce.products_dataset` AS t4
ON t1.product_id = t4.product_id
LEFT JOIN `fsda-400313.Brazilian_Ecommerce.product_category_name_translation` AS t5
ON t4.product_category_name = t5.product_category_name
WHERE EXTRACT(YEAR FROM t2.order_purchase_timestamp) = 2017
AND order_status = 'delivered'
GROUP BY 1,2
ORDER BY 3 DESC



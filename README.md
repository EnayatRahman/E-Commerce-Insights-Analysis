# E-Commerce-Insights-Analysis

## Problem Statement
- Identify the customer with the highest number of purchases.
- List the top 5 products by total quantity sold.
- Calculate total revenue generated from all orders.
- Find customers with total purchases exceeding $500.
- Determine the most frequently used payment method.
- Calculate average order value across all orders.
- Assess the total quantity of products in stock.
- Track orders placed in the last 7 days.
- Find the product with the highest average order quantity.

## Database Schema Overview
```sql
SELECT id, name, email FROM customers;
SELECT id, name, price FROM products;
SELECT id, customer_id, order_date FROM orders;
SELECT id, order_id, product_id, quantity FROM order_items;
SELECT id, order_id, payment_method, amount FROM payments;
```

## Key Queries and Insights


### 1. Customer with the Most Purchases: Identified the customer with the highest number of purchases.

```sql
SELECT customer_id, COUNT(*) AS num_purchases
FROM orders
GROUP BY customer_id
ORDER BY num_purchases DESC
LIMIT 1;
```

### 2. Top 5 Products by Total Sales (Quantity): Listed the top 5 products by the total quantity sold.

```sql
SELECT product_id, SUM(quantity) AS total_quantity_sold
FROM order_items
GROUP BY product_id
ORDER BY total_quantity_sold DESC
LIMIT 5;
```

### 3. Total Revenue Generated: Calculated the total revenue from all orders.

```sql
SELECT SUM(price * quantity) AS total_revenue
FROM products;
```

### 4. Customers with Purchases Over $500: Found all customers whose total purchases exceed $500.
```sql
SELECT c.id, SUM(price) AS total_spent
From customers c
join orders o
on c.id = o.customer_id
join order_items oi
on oi.order_id = o.id
join products p
on p.id = oi.product_id 
GROUP BY customer_id
HAVING total_spent > 500;
```
### 5. Most Frequently Used Payment Method: Determined the most commonly used payment method.
```sql
SELECT method, COUNT(*) AS usage_count
FROM payments
GROUP BY method
ORDER BY usage_count DESC
LIMIT 1;
```
### 6. Average Order Value: Calculated the average order value across all orders.
```sql
SELECT AVG(order_total) AS average_order_value
FROM (
    SELECT o.id AS order_id, SUM(oi.quantity * p.price) AS order_total
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON p.id = oi.product_id
    GROUP BY o.id
) AS order_totals;
```

### 7. Top 3 Customers by Total Amount Spent: Listed the top 3 customers based on the total amount spent.
```sql
SELECT o.customer_id, SUM(p.price) AS total_spent
FROM products p
join order_items oi
on p.id = oi.product_id
join orders o
on o.id = oi.order_id
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 3;
```

### 8. Total Quantity of Products in Stock: Calculated the total quantity of products still available in stock.
```sql
SELECT p.id, p.name, (p.quantity - SUM(oi.quantity) ) AS unsold_quantity
FROM products p
JOIN order_items oi
ON p.id = oi.product_id
GROUP BY p.id, p.name
order by ## p.id Desc,
unsold_quantity;
```

### Alternative solution using window functions.
```sql
SELECT id, name, 
       (quantity - total_sold) AS unsold_quantity
FROM (
    SELECT p.id, p.name, p.quantity,
           SUM(oi.quantity) OVER (PARTITION BY p.id) AS total_sold
    FROM products p
    LEFT JOIN order_items oi ON p.id = oi.product_id
) product_totals;
```

### 9. Orders Placed in the Last 7 Days: Listed all the orders placed in the past 7 days.
```sql
SELECT *
FROM orders
WHERE date >= CURRENT_DATE - INTERVAL 7 DAY;
```
### 10. Product with Highest Average Order Quantity: Found the product with the highest average quantity per order.
```sql
SELECT product_id, AVG(quantity) AS avg_quantity_per_order
FROM order_items
GROUP BY product_id
ORDER BY avg_quantity_per_order DESC
LIMIT 1;
```
### Alternative solution using SUB QUERY and RANK() Function
```sql
SELECT product_id, avg_quantity_per_order
FROM (
    SELECT product_id, 
           AVG(quantity) AS avg_quantity_per_order,
           RANK() OVER (ORDER BY AVG(quantity) DESC) AS rank_
    FROM order_items
    GROUP BY product_id
) ranked_products
WHERE rank_ = 1;
```
### Techniques Used
- **SQL Aggregation Functions**: Utilized `SUM()`, `COUNT()`, and `AVG()` to summarize data effectively.
- **Joins**: Combined data from multiple tables using `JOIN` operations to create comprehensive reports.
- **Window Functions**: Used `RANK()` and `SUM() OVER()` for advanced analysis without collapsing the result set.
- **Filtering and Grouping**: Employed `WHERE`, `GROUP BY`, and `HAVING` clauses to refine data analysis.
- **Date Functions**: Analyzed recent sales trends using date filtering.

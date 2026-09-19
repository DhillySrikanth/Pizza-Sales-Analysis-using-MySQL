# 🍕 Pizza Sales Analysis using MySQL

## 📌 Project Overview
This project analyzes a year's worth of operational sales records for an urban pizza restaurant. Using MySQL, raw transaction logs across 2015 were transformed into structured business intelligence to evaluate total sales volume, revenue drivers, hourly peak operations, customer sizing preferences, and category trends.

---

## 🛠️ Tech Stack & Skills Demonstrated
- **Database Engine:** MySQL
- **SQL Concepts Used:**
  - Multi-table `INNER JOIN` operations
  - Aggregate functions (`COUNT`, `SUM`, `AVG`, `ROUND`)
  - Date & Time extraction functions (`HOUR()`)
  - Subqueries & scalar percentages
  - Common Table Expressions (`WITH ... AS`)
  - Window ranking & cumulative functions (`DENSE_RANK()`, `SUM() OVER()`)

---

## 🏗️ Database Schema & DDL Setup

The database uses 4 relational tables joined via common key attributes (without strict foreign key constraints):

```sql
CREATE DATABASE IF NOT EXISTS pizzahut;
USE pizzahut;

-- Table 1: orders
CREATE TABLE orders (
    order_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    PRIMARY KEY (order_id)
);

-- Table 2: order_details
CREATE TABLE order_details (
    order_details_id INT NOT NULL,
    order_id INT NOT NULL,
    pizza_id TEXT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_details_id)
);

-- Table 3: pizzas
CREATE TABLE pizzas (
    pizza_id TEXT NOT NULL,
    pizza_type_id TEXT NOT NULL,
    size TEXT NOT NULL,
    price DOUBLE NOT NULL,
    PRIMARY KEY (pizza_id(50))
);

-- Table 4: pizza_types
CREATE TABLE pizza_types (
    pizza_type_id TEXT NOT NULL,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    ingredients TEXT NOT NULL,
    PRIMARY KEY (pizza_type_id(50))
);
```

### Table Relationships Overview
- `orders.order_id` $\leftrightarrow$ `order_details.order_id`
- `order_details.pizza_id` $\leftrightarrow$ `pizzas.pizza_id`
- `pizzas.pizza_type_id` $\leftrightarrow$ `pizza_types.pizza_type_id`

---

## 📊 Dataset High-Level Summary (FY 2015)
- **Total Revenue:** $817,860.05
- **Total Pizzas Sold:** 49,574 units
- **Total Orders Placed:** 21,350 orders
- **Average Pizzas Per Order:** 2.32 pizzas
- **Average Daily Production:** ~138 pizzas/day

---

## 🔍 SQL Analysis & Business Queries

### Phase 1: Basic Analysis

#### 1. Retrieve the total number of orders placed
```sql
SELECT 
    COUNT(order_id) AS total_orders 
FROM 
    orders;
```
* **Output:** `21,350`

#### 2. Calculate the total revenue generated from pizza sales
```sql
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_revenue
FROM 
    order_details
JOIN 
    pizzas ON order_details.pizza_id = pizzas.pizza_id;
```
* **Output:** `$817,860.05`

#### 3. Identify the highest-priced pizza
```sql
SELECT 
    pizza_types.name, 
    pizzas.size, 
    pizzas.price
FROM 
    pizzas
JOIN 
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
ORDER BY 
    pizzas.price DESC
LIMIT 1;
```
* **Output:** `The Greek Pizza` (Size: XXL) at `$35.95`

#### 4. Identify the most common pizza size ordered
```sql
SELECT 
    pizzas.size, 
    SUM(order_details.quantity) AS total_quantity
FROM 
    order_details
JOIN 
    pizzas ON order_details.pizza_id = pizzas.pizza_id
GROUP BY 
    pizzas.size
ORDER BY 
    total_quantity DESC
LIMIT 1;
```
* **Output:** Size **L** (`18,956` units sold)

#### 5. List the top 5 most ordered pizza types along with their quantities
```sql
SELECT 
    pizza_types.name, 
    SUM(order_details.quantity) AS total_quantity
FROM 
    pizza_types
JOIN 
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN 
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY 
    pizza_types.name
ORDER BY 
    total_quantity DESC
LIMIT 5;
```
* **Output:**
  1. The Classic Deluxe Pizza (`2,453`)
  2. The Barbecue Chicken Pizza (`2,432`)
  3. The Hawaiian Pizza (`2,422`)
  4. The Pepperoni Pizza (`2,418`)
  5. The Thai Chicken Pizza (`2,371`)

---

### Phase 2: Intermediate Analysis

#### 6. Total quantity of each pizza category ordered
```sql
SELECT 
    pizza_types.category, 
    SUM(order_details.quantity) AS total_quantity
FROM 
    pizza_types
JOIN 
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN 
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY 
    pizza_types.category
ORDER BY 
    total_quantity DESC;
```
* **Output:** Classic: `14,888` | Supreme: `11,987` | Veggie: `11,649` | Chicken: `11,050`

#### 7. Determine the distribution of orders by hour of the day
```sql
SELECT 
    HOUR(order_time) AS order_hour, 
    COUNT(order_id) AS total_orders
FROM 
    orders
GROUP BY 
    HOUR(order_time)
ORDER BY 
    order_hour;
```
* **Peak Output:** 12:00 PM (`2,520`), 1:00 PM (`2,455`), 6:00 PM (`2,399`), 5:00 PM (`2,336`)

#### 8. Category-wise distribution of pizzas
```sql
SELECT 
    category, 
    COUNT(name) AS total_pizza_types
FROM 
    pizza_types
GROUP BY 
    category;
```
* **Output:** Veggie: `9` types | Supreme: `9` types | Classic: `8` types | Chicken: `6` types

#### 9. Group orders by date and calculate average number of pizzas ordered per day (Using CTE)
```sql
WITH daily_pizza_totals AS (
    SELECT 
        orders.order_date, 
        SUM(order_details.quantity) AS total_pizzas
    FROM 
        orders
    JOIN 
        order_details ON orders.order_id = order_details.order_id
    GROUP BY 
        orders.order_date
)
SELECT 
    ROUND(AVG(total_pizzas), 0) AS avg_pizzas_per_day
FROM 
    daily_pizza_totals;
```
* **Output:** `138` pizzas/day

#### 10. Top 3 most ordered pizza types based on revenue
```sql
SELECT 
    pizza_types.name, 
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_revenue
FROM 
    pizza_types
JOIN 
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN 
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY 
    pizza_types.name
ORDER BY 
    total_revenue DESC
LIMIT 3;
```
* **Output:**
  1. The Thai Chicken Pizza: `$43,434.25`
  2. The Barbecue Chicken Pizza: `$42,768.00`
  3. The California Chicken Pizza: `$41,409.50`

---

### Phase 3: Advanced Analysis

#### 11. Percentage contribution of each pizza category to total revenue
```sql
SELECT 
    pizza_types.category,
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS revenue,
    ROUND(
        (SUM(order_details.quantity * pizzas.price) / 
            (SELECT SUM(order_details.quantity * pizzas.price) 
             FROM order_details 
             JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id)
        ) * 100, 
        2
    ) AS revenue_percentage
FROM 
    pizza_types
JOIN 
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN 
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY 
    pizza_types.category
ORDER BY 
    revenue_percentage DESC;
```
* **Output:**
  - Classic: `$220,053.10` (**26.91%**)
  - Supreme: `$208,197.00` (**25.46%**)
  - Chicken: `$195,919.50` (**23.96%**)
  - Veggie: `$193,690.45` (**23.68%**)

#### 12. Analyze cumulative revenue generated over time (Using CTE + Window Function)
```sql
WITH daily_revenue AS (
    SELECT 
        orders.order_date,
        ROUND(SUM(order_details.quantity * pizzas.price), 2) AS revenue
    FROM 
        orders
    JOIN 
        order_details ON orders.order_id = order_details.order_id
    JOIN 
        pizzas ON order_details.pizza_id = pizzas.pizza_id
    GROUP BY 
        orders.order_date
)
SELECT 
    order_date,
    revenue,
    ROUND(SUM(revenue) OVER (ORDER BY order_date), 2) AS cumulative_revenue
FROM 
    daily_revenue;
```

#### 13. Top 3 most ordered pizza types based on revenue for each category (Using CTE + `DENSE_RANK()`)
```sql
WITH category_pizza_revenue AS (
    SELECT 
        pizza_types.category,
        pizza_types.name,
        ROUND(SUM(order_details.quantity * pizzas.price), 2) AS revenue,
        DENSE_RANK() OVER (
            PARTITION BY pizza_types.category 
            ORDER BY SUM(order_details.quantity * pizzas.price) DESC
        ) AS rank_num
    FROM 
        pizza_types
    JOIN 
        pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    JOIN 
        order_details ON pizzas.pizza_id = order_details.pizza_id
    GROUP BY 
        pizza_types.category, 
        pizza_types.name
)
SELECT 
    category,
    name,
    revenue,
    rank_num
FROM 
    category_pizza_revenue
WHERE 
    rank_num <= 3
ORDER BY 
    category, 
    rank_num;
```

---

## 💡 Strategic Business Takeaways & Recommendations

1. **Staffing & Operations Allocation:**
   - The kitchen experiences two distinct daily spikes: lunch (**12 PM – 1 PM**, ~4,975 orders) and dinner (**5 PM – 6 PM**, ~4,735 orders). Staffing shifts and dough batching should peak during these hours to minimize customer wait times.
2. **Menu Engineering & Category Balance:**
   - Chicken pizzas account for only 6 out of 32 menu varieties, yet they capture **all top 3 slots** in individual pizza revenue ($127.6K combined). Chicken items represent high-margin favorites that warrant prominent placement on digital and print menus.
3. **Inventory & Sizing Optimization:**
   - Size **Large (L)** generates 18,956 units sold, far outperforming Small, Medium, and extra sizes. Packaging, box procurement, and prep-line portion pans should prioritize Large sizing to prevent stock-outs during peak seasons.

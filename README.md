# Logistics Fleet Delivery Turnaround & Delay Analysis

## Project Overview
This project evaluates fulfillment speed, transit variance, and regional delivery bottlenecks across **96,400+ completed e-commerce orders** using the Brazilian E-Commerce Public Dataset (Olist). 

The goal is to diagnose operational bottlenecks across warehouse dispatch versus carrier transit and determine how freight cost structures impact delivery velocity.

---

## Executive Summary & Key Findings
* **Primary Bottleneck Identified:** For delayed shipments, carrier transit accounts for **82.8%** of total delay duration. While warehouse dispatch times double on delayed orders (from 2.58 to 5.32 days), carrier transit increases by **325%** (from 7.89 days to 25.68 days).
* **Regional Disparities:** Northern and Northeastern destination lanes suffer the highest failure rates, led by Maranhão (MA, 19.67% delay rate) and Ceará (CE, 15.32% delay rate), both averaging ~18 days in road transit.
* **Freight Cost vs. Transit Inversion:** Higher freight charges do not yield faster delivery speeds. Top-quartile freight orders average 11.76 days in transit compared to 4.79 days for the lowest-cost tier, showing freight rates reflect distance surcharges rather than expedited service tiers.

---

## Technical Stack
* **Language / Engine:** SQLite / SQL
* **Environment:** Python (Kaggle Notebooks) / Pandas
* **Core Techniques:** Common Table Expressions (CTEs), Window Functions (`NTILE`), Date Arithmetic (`JULIANDAY`), Aggregate Case Expressions.

---

## Analytical SQL Queries & Results

### 1. Regional Delivery Performance & Carrier Failure Rate
Evaluates line-haul performance and carrier failure rates across customer states with at least 500 orders.

```sql
WITH DeliveryMetrics AS (
    SELECT 
        c.customer_state,
        o.order_id,
        ROUND(JULIANDAY(o.order_delivered_customer_date) - JULIANDAY(o.order_delivered_carrier_date), 2) AS transit_days,
        CASE 
            WHEN JULIANDAY(o.order_delivered_customer_date) > JULIANDAY(o.order_estimated_delivery_date) THEN 1
            ELSE 0
        END AS is_delayed
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    WHERE o.order_status = 'delivered'
      AND o.order_delivered_customer_date IS NOT NULL
      AND o.order_delivered_carrier_date IS NOT NULL
)
SELECT 
    customer_state,
    COUNT(order_id) AS total_delivered_orders,
    ROUND(AVG(transit_days), 2) AS avg_transit_days,
    ROUND(100.0 * SUM(is_delayed) / COUNT(order_id), 2) AS delay_rate_pct
FROM DeliveryMetrics
GROUP BY customer_state
HAVING total_delivered_orders >= 500
ORDER BY delay_rate_pct DESC
LIMIT 10;

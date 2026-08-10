# Day 1 Solution: Loading, Exploring, and Joining Olist

This is a worked solution for `week4_day1_task.md` — real column names from the Olist Kaggle dataset, a working join query, and sample answers. Treat this as one valid solution, not the only correct one (e.g., the revenue definition or delivery-time calculation could reasonably be built differently).

## Block 1 — Loading and Profiling

```sql
-- Load every CSV into DuckDB as a table
CREATE TABLE orders AS SELECT * FROM read_csv_auto('data/olist_orders_dataset.csv');
CREATE TABLE order_items AS SELECT * FROM read_csv_auto('data/olist_order_items_dataset.csv');
CREATE TABLE order_payments AS SELECT * FROM read_csv_auto('data/olist_order_payments_dataset.csv');
CREATE TABLE order_reviews AS SELECT * FROM read_csv_auto('data/olist_order_reviews_dataset.csv');
CREATE TABLE customers AS SELECT * FROM read_csv_auto('data/olist_customers_dataset.csv');
CREATE TABLE products AS SELECT * FROM read_csv_auto('data/olist_products_dataset.csv');
CREATE TABLE sellers AS SELECT * FROM read_csv_auto('data/olist_sellers_dataset.csv');
CREATE TABLE category_translation AS SELECT * FROM read_csv_auto('data/product_category_name_translation.csv');
CREATE TABLE geolocation AS SELECT * FROM read_csv_auto('data/olist_geolocation_dataset.csv');

-- Row counts and grain check
SELECT 'orders' AS tbl, COUNT(*) FROM orders
UNION ALL SELECT 'order_items', COUNT(*) FROM order_items
UNION ALL SELECT 'order_payments', COUNT(*) FROM order_payments
UNION ALL SELECT 'order_reviews', COUNT(*) FROM order_reviews;

-- Confirm order_items is one-to-many against orders
SELECT order_id, COUNT(*) AS item_rows
FROM order_items
GROUP BY order_id
ORDER BY item_rows DESC
LIMIT 10;

-- Same check for payments — some orders split across multiple payment methods
SELECT order_id, COUNT(*) AS payment_rows
FROM order_payments
GROUP BY order_id
ORDER BY payment_rows DESC
LIMIT 10;

-- Order status values, to decide what counts as "revenue"
SELECT order_status, COUNT(*) FROM orders GROUP BY order_status ORDER BY 2 DESC;
```

Typical findings: `order_status` includes `delivered`, `shipped`, `canceled`, `unavailable`, and a few others — this project counts revenue only from `delivered` orders. `order_items` and `order_payments` are both genuinely one-to-many against `orders`, confirming the aggregation choices below are necessary, not optional.

## Block 3 — The Join Query

```sql
CREATE TABLE orders_summary AS
WITH payments_agg AS (
    SELECT
        order_id,
        SUM(payment_value) AS total_payment_value
    FROM order_payments
    GROUP BY order_id
),
reviews_dedup AS (
    -- Some orders have more than one review row; keep the most recent
    SELECT
        order_id,
        review_score,
        ROW_NUMBER() OVER (
            PARTITION BY order_id ORDER BY review_creation_date DESC
        ) AS rn
    FROM order_reviews
)
SELECT
    o.order_id,
    o.customer_id,
    c.customer_state,
    c.customer_city,
    oi.product_id,
    COALESCE(t.product_category_name_english, p.product_category_name, 'unknown') AS category_english,
    oi.price,
    oi.freight_value,
    oi.price + oi.freight_value AS item_revenue,
    o.order_purchase_timestamp,
    o.order_delivered_customer_date,
    DATE_DIFF('day', o.order_purchase_timestamp, o.order_delivered_customer_date) AS delivery_days,
    r.review_score,
    pay.total_payment_value
FROM orders o
JOIN order_items oi        ON o.order_id = oi.order_id
JOIN customers c            ON o.customer_id = c.customer_id
JOIN products p              ON oi.product_id = p.product_id
LEFT JOIN category_translation t ON p.product_category_name = t.product_category_name
LEFT JOIN payments_agg pay   ON o.order_id = pay.order_id
LEFT JOIN reviews_dedup r    ON o.order_id = r.order_id AND r.rn = 1
WHERE o.order_status = 'delivered'
  AND o.order_delivered_customer_date IS NOT NULL;
```

Design choices worth calling out explicitly:

- **Revenue = `price + freight_value`.** A defensible alternative is item price alone — either is fine as long as it's used consistently in every downstream measure.
- **Only `delivered` orders count.** Canceled or still-in-transit orders don't have a meaningful delivery time and shouldn't count toward revenue totals.
- **`payments_agg` collapses multi-payment orders to one row per order** before joining, so the join to `order_items` doesn't fan out payment rows across every item in the order.
- **`reviews_dedup` keeps one review per order** using `ROW_NUMBER()`, avoiding silent row duplication from orders with more than one review.

## Block 3, Task 7 — Spot-Check

```sql
SELECT *
FROM orders_summary
WHERE order_id = (SELECT order_id FROM orders_summary LIMIT 1);
-- Compare item_revenue, delivery_days, review_score, category_english
-- against the raw orders/order_items/order_reviews rows for that order_id manually.
```

## Block 4 — Export

```sql
COPY orders_summary TO 'exports/orders_summary.parquet' (FORMAT PARQUET);
```

## Sample Scope Note (Block 2)

> **Dashboard scope — Olist E-Commerce, Week 4**
> 1. **Revenue trend** — how has monthly revenue moved over the dataset's date range, and does it vary meaningfully by product category or customer state? *Needs:* `order_purchase_timestamp`, `item_revenue`, `category_english`, `customer_state`.
> 2. **Delivery time vs. review score** — do longer delivery times associate with lower review scores? *Needs:* `delivery_days`, `review_score`.
> 3. **Category/state performance** — which categories and states are the strongest and weakest by revenue? *Needs:* `category_english`, `customer_state`, `item_revenue`.
>
> **Out of scope this week:** geolocation/mapping, payment-method-level analysis, seller-level performance.

## Sample Reflection Answers

1. **Scope check:** Yes — all three questions map directly to columns already present in `orders_summary`; no additional joins are needed for the remaining days.
2. **Compared to Week 1:** The biggest difference is the absence of a course-provided schema — confirming grain (one-to-many on `order_items` and `order_payments`) had to be verified with actual `GROUP BY` queries rather than assumed from documentation.
3. **Biggest risk:** Running out of time on polish/presentation prep if Day 1's SQL runs long — mitigated by treating `orders_summary` as final once it passes the spot-check, rather than continuing to refine it into Day 2.
4. **Expected visuals:** The revenue trend line chart is the most natural fit (one date column, one revenue measure already built). The delivery-time-vs-review-score chart is expected to need the most extra work, since it requires a new bucketing measure not yet built.

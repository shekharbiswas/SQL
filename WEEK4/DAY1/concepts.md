# Day 1 Concepts: Getting to Know Olist, Then Connecting Power BI to It

**Estimated reading time: 35 min**

Weeks 1–3 ended with real fluency in SQL, and Power BI itself isn't new territory either. What *is* new this week is the dataset: Olist's Brazilian E-Commerce data, nine CSVs on Kaggle with no schema doc from the course, no prior queries written against it, and no dbt marts waiting to be connected to. That changes what "Day 1" means. It isn't a tools lesson — it's a fast, deliberate orientation into an unfamiliar dataset, done with enough discipline that by Thursday there's a dashboard, and by Friday 11am there's something worth presenting.

So this file is short on Power BI mechanics you already know, and spends its weight on two things that are genuinely new this week: understanding Olist's shape, and deciding — on Day 1, not Day 3 — exactly what the dashboard is going to say.

---

## Why Today Starts Slower Than It Looks

With four build days and a Friday-morning deadline, the instinct is to open Power BI immediately. Resist that. A dashboard built on a misunderstood schema — a join on the wrong key, a "revenue" figure that's actually just item price without freight, a state field that's actually the seller's state, not the customer's — is a dashboard that falls apart under the first real question in the room on Friday. An hour spent understanding Olist today is worth more than an hour spent building visuals on a table you don't fully trust yet.

The good news: because SQL and Power BI are already comfortable tools, all of today's time budget can go toward the dataset itself rather than being split with syntax you'd need to look up anyway.

## The Olist Schema, At a Glance

Olist's data ships as eight or nine separate CSVs, roughly:

- **`orders`** — one row per order, with status and a handful of timestamp columns (purchase, approval, delivery)
- **`order_items`** — one row per item within an order (an order with 3 products has 3 rows here), with price and freight value
- **`order_payments`** — one row per payment (an order can have more than one payment row)
- **`order_reviews`** — one row per review, with a score and optional comment
- **`customers`** — one row per customer, including their state and city
- **`sellers`** — one row per seller, including their state and city
- **`products`** — one row per product, including category (in Portuguese)
- **`product_category_name_translation`** — a small lookup table translating category names to English
- **`geolocation`** — zip-code-level lat/long, useful but not essential for a first pass

Notice what this shape implies: `orders` is the natural spine, but "revenue," "delivery time," "category," and "review score" all live in *different* tables, at different grains. `order_items` is one-to-many with `orders`. `order_payments` can also be one-to-many. `order_reviews` is close to one-to-one with `orders` but not guaranteed to be. Getting the grain of each table right — one row per what? — matters more here than it did with the bikeshare `trips` table, which was already one clean row per ride.

## Deciding the Dashboard's Scope, With Friday in Mind

Because this dataset is new and the deadline is fixed, scope has to be decided deliberately, not discovered by accident on Wednesday night. Before writing any SQL today, it's worth committing to two or three specific business questions the dashboard will answer — for example:

- How has revenue trended over time, and does it vary by product category or customer state?
- How does delivery time relate to review score?
- Which categories or states are the strongest and weakest performers?

Three sharp, well-answered questions make a stronger Friday presentation than eight shallow ones. Everything from here — which tables get joined, which columns get kept, which visuals get built Tuesday and Wednesday — should trace back to whichever two or three questions get chosen today.

## Import vs. DirectQuery, Quickly

This is a decision already familiar from other Power BI work, so just the conclusion for this week: **Import mode**, for the same reason as always — the dashboard doesn't need to be live-connected to a source that's only refreshed manually a few times this week, and Import gives faster, more predictable performance while building. DirectQuery would make more sense against a proper concurrent-query engine like Snowflake, which Friday morning's tutorials touch on conceptually — not against a single DuckDB file being rebuilt by hand.

## Connecting Power BI to DuckDB

Same two paths as any DuckDB-backed project:

1. **Export to Parquet, then import as files** — `COPY table_name TO 'exports/table_name.parquet' (FORMAT PARQUET)`, then Power BI's **Get Data → Folder** or **Get Data → Parquet**. This is the path for this week.
2. **DuckDB ODBC driver + Power Query custom connector** — more setup, worth knowing exists, not needed for this week's timeline.

## What "The Marts" Mean This Week

Week 3's bikeshare project had dbt building tested, documented marts automatically. There's no dbt project for Olist this week — the timeline doesn't allow for one, and that's a deliberate scope decision, not an oversight. Instead, today's SQL work *is* the marts step, done by hand: joining `orders`, `order_items`, `customers`, `products`, `order_payments`, and `order_reviews` into one or two clean, flat summary tables — order-level, with category, state, revenue, delivery time, and review score sitting together in one place. That table, not the raw nine, is what gets exported to Parquet and loaded into Power BI. Same principle as dbt's staging/marts split — raw tables stay raw, something else gets built specifically to be queried downstream — just done directly in SQL instead of through a dbt project.

---

## Learning Resources

- **Kaggle — Brazilian E-Commerce Public Dataset by Olist** — the dataset page itself includes an entity-relationship diagram showing how all nine tables connect; worth having open while exploring today: [kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Coursera — Extract, Transform and Load Data in Power BI (Microsoft)** — a quick refresher specifically on data source configuration and storage modes, if any of that feels rusty: [coursera.org/learn/extract-transform-and-load-data-in-power-bi](https://www.coursera.org/learn/extract-transform-and-load-data-in-power-bi)

## What's Next

Today's task file is where the schema exploration turns into an actual scope decision and a first SQL summary table. Day 2 picks up from there and starts building real visuals against whatever gets exported today — so the more clearly Day 1 ends, the faster Day 2 moves.
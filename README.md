# 📊 SQL & Modern Data Tools

At **Dataskools**, we believe that data skills are the foundation of every future-proof career.  
This 6-week learning path is designed for **analysts, career switchers, and business professionals** who want to become job-ready with SQL — and go beyond.

Using one powerful company (**TrendHive**), you’ll progressively master:
- 📄 SQL (Foundations to Advanced)
- 🧠 Analytics + Business Thinking
- 🔗 Modular Data Modeling with **DBT**
- ❄️ Cloud Data Warehousing with **Snowflake**
- 📊 Power BI for Interactive Dashboards

And the best part? You don’t just learn — you **build**, every week. 🚀

<br>

<p align="center">
  <img width="350" alt="TrendHive" src="https://github.com/user-attachments/assets/29ea16f0-d053-418c-aa36-856750f236fe" />
</p>

<br>

## 🧩 Challenge: _Can Data Help_ TrendHive Compete?

**TrendHive** is a rising eCommerce brand selling viral products — from solar-powered gear to eco-friendly accessories. But now it’s scaling fast, and **data is the only way to survive**.

They’ve hired you as a **junior data analyst** to support product, marketing, and operations teams. But there’s a catch — each team comes with **messy questions, evolving goals, and incomplete information.**

You won’t be told what to do.  
Instead, each week starts with a **realistic business problem**.  
You’ll investigate, explore multiple paths, and present what *you* think is the best insight.

<br>
<br>

## 🖥️ How the TrendHive Data System Works

At TrendHive, raw data flows in daily from various sources:

- 🛒 **Orders** from the eCommerce platform  
- 👤 **Customer data** from CRM systems  
- 🎯 **Campaign logs** from marketing tools  
- ⭐ **Reviews and returns** from product feedback forms  
- 📦 **Fulfillment data** from warehouse and shipping APIs

These datasets are:
- Stored in **Snowflake** (cloud warehouse)
- Modeled using **DBT** (for structured analytics)
- Explored via **SQL**, **Power BI**, and reverse ETL outputs

You’ll operate inside this system as a **hands-on analyst** — writing SQL models, testing assumptions, and delivering insights that plug into real dashboards and business workflows.


<br>
<br>


## 🧠 Week 1: Where Are We Selling the Most?

> 📦 *“Sales just started coming in — but we don’t know what’s actually selling well.”*

TrendHive has launched. Orders are flowing. But no one has a clear answer to:
- Which cities are performing best?
- What products are trending?
- What’s the average order size?

### 🔍 Your task:
Use basic SQL with **DuckDB** to analyze the `orders` and `products` datasets. Identify key patterns, then create a **Top Cities by Sales** report.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- SQL foundations: SELECT, WHERE, ORDER BY  
- Filtering text, numbers, and dates  
- Running queries locally without setup (DuckDB or SQLite)  
</details>

<details>
<summary>✅ By the End of Week 1</summary>

- Write and run basic queries  
- Sort, filter, and group data  
- Identify where TrendHive should focus next  
</details>

<br>

## 🔗 Week 2: Who Are Our Best Customers?

> 👥 *“We’ve added a loyalty program. Can we track which customers are worth retaining?”*

Customer and support ticket data are now available. The Customer Support team wants insights into:
- Which customers are loyal?
- Who raises the most tickets?
- Are our VIPs getting the best experience?

### 🔍 Your task:
Use SQL joins and aggregates to uncover **top customers, churn risks, and support patterns**.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- SQL JOINs (INNER, LEFT)  
- Aggregations: SUM, COUNT, AVG  
- Handling NULLs and grouping by customer behavior  
</details>

<details>
<summary>✅ By the End of Week 2</summary>

- Join multi-table datasets  
- Segment customers by activity and value  
- Build `customer_segments` mart  
</details>

<br>

## 📈 Week 3: Are Our Marketing Campaigns Working?

> 📣 *“We’ve run 3 ad campaigns this month. Which one worked best?”*

Marketing wants clarity — but the data is messy. Clicks don’t always mean purchases.

### 🔍 Your task:
Use **advanced SQL** to attribute sales to campaigns and determine what truly worked.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- Common Table Expressions (CTEs)  
- CASE WHEN and conditional logic  
- Window functions: ROW_NUMBER, RANK, LAG  
</details>

<details>
<summary>✅ By the End of Week 3</summary>

- Model campaign attribution logic  
- Build `marketing_attribution` mart  
- Recommend data-backed investment decisions  
</details>

<br>

## 📊 Week 4: How Do Customers Feel About Our Products?

> ⭐ *“We just got hundreds of reviews. Are our bestsellers also the best-rated?”*

TrendHive wants to understand the emotional side of product performance.

### 🔍 Your task:
Clean and analyze `reviews` and `returns` data to surface **quality and sentiment insights**.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- Text pattern filtering with LIKE  
- Basic sentiment scoring using keywords  
- Dashboarding in Power BI  
</details>

<details>
<summary>✅ By the End of Week 4</summary>

- Create `product_feedback` mart  
- Show which products delight vs. disappoint  
- Visualize insights with Power BI  
</details>

<br>

## 🧱 Week 5: Can We Make Our SQL Reusable?

> 🔧 *“Our queries are growing — and hard to manage. Time to modularize.”*

You’ll help TrendHive structure analytics using **DBT** — the modern standard for SQL modeling.

### 🔍 Your task:
Refactor raw queries into modular **DBT models**, then build analytics marts with testing and docs.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- DBT workflow: sources → staging → marts  
- Jinja templating and DRY logic  
- Data tests and documentation  
</details>

<details>
<summary>✅ By the End of Week 5</summary>

- Build `fulfillment_analytics` DBT project  
- Modularize SQL pipelines  
- Track KPIs like shipping delays and inventory risk  
</details>

<br>

## ❄️ Week 6: Can We Build a CEO Dashboard?

> 💼 *“Our CFO wants to see profit trends, ROI, and CLV. Can you deliver an exec dashboard by Friday?”*

Now it’s time to **combine everything** into a real executive dashboard.

### 🔍 Your task:
Use **Snowflake + DBT + Power BI** to create a full pipeline and final report for TrendHive’s leadership.

<details>
<summary>💡 Concepts You’ll Explore</summary>

- Cloud warehousing with Snowflake  
- End-to-end pipeline automation  
- Integrating DBT models with Power BI  
</details>

<details>
<summary>✅ By the End of Week 6</summary>

- Automate data refreshes in Snowflake  
- Deliver an executive dashboard  
- Complete your final capstone project  
</details>

<br>

---

## 🧠 Modern SQL Concepts You’ll Touch Along the Way

Even as a beginner, you’ll gradually build intuition for powerful, real-world SQL concepts:

- **Materialized Views** — Cache expensive queries to speed up dashboards  
- **Data Modeling** — Apply dimensional design (star/snowflake schemas)  
- **Reverse ETL** — Use SQL results to power external tools (e.g., CRM, ads)  
- **Streaming SQL** — Understand live-querying of real-time data (Kafka SQL, Materialize)

These tools are used by top data teams at companies like Airbnb, Shopify, and Notion — and now, by you.

---

> 🚀 **Your mission:** Go from SQL learner to modern data pro — fluent in cloud-native workflows, analytics tools, and real-world challenges.

# 📘 Overview: _Where_ are we selling the most?

Welcome to Week 1 of your SQL journey at **TrendHive Bootcamp**. This week, you join the team just after TrendHive’s e-commerce site has launched. Orders are rolling in — but the business has no idea where success is coming from.

Instead of starting with SQL theory, we start with a **problem**:

> 💬 *“We know orders are happening... but which cities are driving our sales? What products are popular? And what exactly is our average order size?”*

Your job this week is to **investigate, not memorize**. You'll explore real datasets, ask the right questions, and build your first insight: *Top Cities by Sales*. Each day, the complexity increases as we unlock new angles of the same problem.

---

<p align="center">
  <img width="600" alt="TrendHive dashboard" src="https://github.com/user-attachments/assets/trendhive-orders-week1.png" />
</p>

---

## 🗓️ Weekly Breakdown

| Day | Theme                             | Discovery Focus                                                      |
|-----|-----------------------------------|-----------------------------------------------------------------------|
| 1   | **What’s in Our Orders?**         | Understand the structure of the `orders` table. What is each column for? |
| 2   | **Let’s Look at Products**        | Join `products` with `orders` — are we selling cheap items or high-margin ones? |
| 3   | **Sales by City**                 | GROUP BY and ORDER BY to see top cities. Revenue or volume? You decide. |
| 4   | **Trends Over Time**              | Slice data by date — which days are busiest? How’s momentum changing?  |

---

## 🎯 Learning Objectives

By the end of this week, you will be able to:

- Read and explore structured SQL datasets (`orders`, `products`)
- Write basic SQL queries: `SELECT`, `WHERE`, `ORDER BY`
- Join two tables using keys (foreign key: `product_id`)
- Use `GROUP BY` and simple aggregates to summarize trends
- Build a basic city-wise sales report for business decision-making

---

## 📈 Difficulty Snapshot – Week 1

You’ll ramp up from data exploration to insights, building confidence in querying.

| 🧠 Topic Area         | ⚙️ Difficulty | 💬 What to Expect                                                       |
|----------------------|----------------|-------------------------------------------------------------------------|
| 🔍 SELECT basics      | 🟢 Easy         | Reading data is intuitive — like a spreadsheet view                     |
| 🔎 WHERE filters      | 🟢 Easy         | Simple conditions help slice your dataset quickly                       |
| 📊 GROUP BY & SUM     | 🟡 Medium       | Start thinking in terms of aggregation, not just rows                   |
| 🔗 Table JOINs        | 🟡 Medium       | Merging `orders` with `products` may feel new — focus on matching keys |
| 📅 Date-based trends  | 🟡 Medium       | You'll format and group by date — learn to think temporally             |

> 💡 **Tip:** You’ll run into messy rows or unexpected results — pause, debug, and rethink. That’s how real analysts grow.

---

## 🧰 Tools Used This Week

- **SQL Playground / Jupyter Notebook** (or DB of your choice)
- Provided CSV datasets: `orders.csv`, `products.csv`
- Optional: Any database with SQL support (SQLite, Postgres, BigQuery)

> 🗒️ *We recommend using an online SQL editor if you're just starting — no installation needed.*

---

## 🏁 End-of-Week Outcomes

By Friday, you should be able to:

✅ Query raw transactional data using `SELECT`, `WHERE`, `ORDER BY`  
✅ Join product info to order lines for richer insights  
✅ Use `GROUP BY` to find top cities, products, or dates  
✅ Compare different types of metrics (revenue vs. quantity)  
✅ Present a mini-report to the business team — including what you’d recommend next

---

> 🚀 **Reflection Prompt:** What surprised you in the sales patterns? Were the top cities what you expected? Did you focus on volume or value — and why?


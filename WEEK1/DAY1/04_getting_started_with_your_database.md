# 🗃️ Getting Started with Your Database

Before we run any SQL, let’s get clear on something basic:

> 💬 **What is a database? And what exactly is a table?**

If you’ve used spreadsheets like Excel or Google Sheets before, you already understand more than you think.

<br>

## 💾 What Is a Database?

A **database** is a structured way to store information on a computer.

Instead of one big file, it organizes data into different **tables** — kind of like folders inside your computer, or separate sheets in Excel.

In this course, we use a simple format called **SQLite**, which stores all your data in one `.db` file:  
👉 `trendhive_week1.db`

You’ll open this file using **Beekeeper Studio**, a SQL editor where you can write queries and view your results.

<br>

## 📄 What Is a Table?

A **table** is a collection of similar data — like a single spreadsheet with rows and columns.

Each table has:

- **Columns** → The types of information (like name, price, or city)  
- **Rows** → Each individual record (like one product or one order)

<br>

## 📦 Example from TrendHive

When you connect to your database, you’ll see two tables:

### 1. `orders`

This table shows each customer’s order.

Example columns:

- `order_date` → When the order was placed  
- `city` → Where it was delivered  
- `price` → How much the customer paid  
- `product_id` → A reference to what was purchased

<br>

### 2. `products`

This table lists the items TrendHive sells.

Example columns:

- `name` → Product name (like “Solar Lamp”)  
- `category` → Type of item (Home, Gadgets, etc.)  
- `base_cost` → Internal cost before markup  
- `launch_date` → When it was added to the catalog

<br>

## 🧭 What You’ll Do Next

You’ll open this `.db` file in Beekeeper Studio, explore the tables, and begin writing your first SQL query.

It’s just like asking questions about the data — but using SQL instead of filters or pivot tables.

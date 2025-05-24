# 👋 Welcome to TrendHive

Welcome to your first day as a **Junior Data Analyst** at **TrendHive**, one of the most exciting eCommerce startups of the year.

TrendHive sells viral, eco-friendly, and seasonal products — from solar-powered camping gear to minimalist home gadgets. The product line changes frequently to match trends. The team moves fast. But that speed comes at a cost: **they’re flying blind without data**.

That’s where you come in.

<br>

## 🧠 Your Role

You've just joined the analytics team — and here’s what you need to know:

- You won't be given step-by-step instructions.
- You won't be solving made-up textbook puzzles.
- You’ll get real datasets, messy questions, and limited context.

Why? Because that’s exactly how data works in the real world.

Each week, you’ll sit "in the room" with a different business team — product, marketing, ops — and help them answer questions like:

- Where are our sales really coming from?
- Are we keeping our best customers?
- Is that campaign actually working?
- Why are returns going up?
- Can we ship smarter?

These aren’t just SQL tasks — they’re **business problems waiting for your insight**.

<br>

## 🏢 A Quick Look at TrendHive

Here’s the lay of the land inside the company:

| Team          | What They Do                                     | Why They Need You                     |
|---------------|--------------------------------------------------|----------------------------------------|
| 🛒 Product     | Launches and updates trendy SKUs                | Wants to know what sells vs. what returns |
| 📢 Marketing   | Runs ad campaigns, tracks traffic               | Needs proof that campaigns drive real sales |
| 📦 Operations | Handles fulfillment and stock                   | Needs alerts for delays or overstocking |
| 💬 Support     | Handles tickets, reviews, and complaints        | Wants to know if high-paying customers are happy |
| 🧑‍💻 Data Team  | You + a few analysts                           | Your job is to translate messy data into clean insights |

> You’ll rotate through teams during the bootcamp. Each week = new dataset + new mission.

<br>

## 📦 What Data Will You Use?

This week, you’ll work with two key files:

- `orders.csv`: every individual purchase made through the website  
- `products.csv`: details about each item being sold (price, category, margin, etc.)

Each row in `orders.csv` includes:
- `order_id`
- `product_id`
- `order_date`
- `city`
- `price` (what the customer paid)

Each row in `products.csv` includes:
- `product_id`
- `name`
- `category`
- `base_cost`
- `launch_date`

Your goal this week is to **connect these dots** and answer one big question:

> **Where should TrendHive focus their marketing efforts based on current sales?**

<br>

## 🔧 Tools You’ll Use

You'll write and run SQL queries using **Beekeeper Studio** — a clean, beginner-friendly SQL editor that works seamlessly with local databases like **SQLite**.

> 🔗 Download it here: [www.beekeeperstudio.io](https://www.beekeeperstudio.io)

### Your workflow:
1. We’ll load the CSV files (`orders.csv`, `products.csv`) into a **SQLite database**.
2. You’ll open that SQLite file in **Beekeeper Studio**.
3. From there, you’ll write SQL queries, explore the data, and uncover insights.

Why Beekeeper Studio?

- ✅ Easy setup and clean UI
- ✅ Works offline with no registration
- ✅ Browse tables and results visually
- ✅ Feels close to real-world tools used by analysts

> 🧪 Bonus: This workflow teaches you how data professionals really query local databases.

<br>

## 🧭 What Success Looks Like This Week

You are not expected to be perfect. You are expected to be curious.

Here’s what you’ll do:

✅ Run your first SELECT and WHERE queries  
✅ Explore the shape of the data: what’s missing, what’s messy  
✅ Start thinking in metrics (revenue vs. volume vs. margin)  
✅ Join data across tables  
✅ Use SQL to build your first **real business insight**

In the end, you’ll present:
- Top cities by sales
- A short insight: **where should we double down, and why?**

<br>

## 🧠 Keep This in Mind

- You are here to explore — not to memorize.
- Getting stuck is part of the job. Real analysts **debug more than they query**.
- Sometimes there’s **no right answer**, only better assumptions.
- Ask yourself often: *Would this insight help someone make a decision?*

<br>

> 📝 **Journal prompt:** What kind of decisions would you want to help make as an analyst? If you were the CEO, what would you want to know from this data?

---

🎉 You're ready to dive in. Start with loading the data and exploring your first 100 rows.

Your next step: **`02_intro_to_orders_table.md`**

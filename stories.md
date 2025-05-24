# 📅 Week 1: Where Are We Selling the Most?

**Focus:** City Trends, Product Demand, and Emerging Patterns

## 🔍 10 Micro-Stories (from `orders` + `products`)

- 💡 **Solar Lamps** are consistently ordered from **Berlin** and **Cologne**, likely for balcony and garden use.  
- ♻️ **Compost Bins** are surprisingly popular in smaller eco-conscious cities like **Freiburg** and **Tübingen**.  
- 🏃 **Reusable Bottles** spike in **Munich** and **Hamburg** — hinting at fitness culture or sports usage.  
- 🔧 **Bike Repair Kits** are a hit in **Leipzig**, **Stuttgart**, and **Münster** — cities with strong cycling communities.  
- 🎄 **LED Fairy Lights** surge in **late November and December**, especially in **Cologne** — likely tied to gift-giving and Advent.  
- 🧴 **Organic Soap** sees weekend spikes in **Stuttgart** and **Nürnberg** — possibly used as gifts or bundle items.  
- 🌡️ **Smart Thermometers** are often ordered on weekdays from **Frankfurt** — tied to hybrid home-office life.  
- 🛍️ **Foldable Shopping Bags** are trending in **Berlin** and **Hamburg** — correlating with local plastic bag bans.  
- 🌱 **Desk Plant Kits** sell steadily in **Frankfurt** and **Munich** — linked to workspace decor or gifting.  
- 🔋 **Portable Chargers** sell more on weekends in **Frankfurt** and **Düsseldorf** — airports suggest travel use.

<br>

## 🔗 5 Behavior Patterns (Grouped Insights)

| Theme                   | Pattern                                                                 |
|------------------------|-------------------------------------------------------------------------|
| 🏙️ Urban Lifestyle      | Compact, sustainable products like lamps, bags, and desk plants are popular in dense cities.  
| ♻️ Eco-Centric Living   | Green-focused cities prefer compost bins, toothbrushes, and soaps.  
| 🎁 Gifting Seasonality  | Products like fairy lights and soap peak around weekends and holidays.  
| 🚴 Mobility & Commuting | Cities with cycling culture favor repair kits and portable gear.  
| 🕰️ Weekday vs. Weekend  | Some products trend by day type — work vs. leisure usage.

<br>

## 🎯 2 Strategic Decision Points

> **“Should TrendHive segment marketing by city behavior instead of by product category?”**  
> → Yes. City context (fitness, eco, urban lifestyle) is a stronger driver than product type alone.

> **“Can timing tell us what people use vs. gift?”**  
> → Yes. Order timing (weekday vs. weekend, seasonal trends) reveals **why** people buy — not just **what**.

<br>

# 📅 Week 2: Who Are Our Best Customers?

**Focus:** Loyalty, Value, and Friction

## 🔍 10 Micro-Stories (Hidden in `customers`, `orders`, `support_tickets`)

- 🛒 Customer #238 has made **14 small orders**, all under €20 — but never opened a support ticket.  
- 📉 Customer #482 spent over €300 in total — but left **3 bad ratings** via support tickets.  
- 🏙️ Customers from **Leipzig** have a higher average spend but **slower repeat purchase rate**.  
- 📦 Many top customers in **Munich** consistently order within **10 days of each other**.  
- 🛑 Customer #901 unsubscribed from marketing but still places **monthly orders** — why?  
- 🧑‍💻 Customers in the **Gold tier** log the **fewest complaints**, but also contact support *fast*.  
- ⏳ Customers in **Hamburg** open tickets for **delayed shipping** — many wait **4+ days** for resolution.  
- 🎁 New customers acquired via **gift referrals** spend **less in month 2** than others.  
- 🧼 Frequent buyers of **Organic Soap** tend to open **“product quality”** tickets.  
- 🧾 Several customers place **bulk orders only once per quarter** — possibly businesses?  

<br>


## 🔗 5 Behavior Patterns (Grouped Insights)

| Theme                        | Pattern                                                                 |
|-----------------------------|-------------------------------------------------------------------------|
| 🏅 Loyalty ≠ High Spend      | Some loyal repeat customers never spend big — but they’re consistent.   |
| 😕 Silent churn signals      | Multiple tickets + longer gaps between purchases = drop risk.           |
| 🧭 Location matters          | Cities like Hamburg and Leipzig show very different support rhythms.    |
| 📬 Marketing opt-out ≠ lost  | Some buyers don’t need emails — they return on habit or need.           |
| 🚨 Support = product signal  | Tickets often expose product issues faster than reviews.                |

<br>

## 🎯 2 Strategic Decision Points

> **“Should we treat high-frequency buyers the same as high-spending buyers?”**  
> → Loyalty program redesign needed — focus on frequency, not just euros.

> **“Can support data help reduce churn before it happens?”**  
> → Early intervention based on time-between-orders + complaint type.

<br>

# 📅 Week 3: Are Our Marketing Campaigns Working?

**Focus:** Campaign Attribution, ROI, and Conversion Patterns

## 🔍 10 Micro-Stories (Using `campaigns`, `clicks`, `orders`)

- 📱 The **Instagram campaign** brought **12,000 clicks** — but only **300 orders** (low conversion).  
- ✉️ The **email campaign** had fewer clicks (3,000) but generated **€18,000+** in revenue.  
- ⏱️ Many customers ordered **2–3 days after clicking** an ad — not immediately.  
- 🔁 800 customers clicked **more than once**, but only ordered after the **second click**.  
- 🚫 Some orders had **no click trace at all** — maybe direct traffic or word of mouth?  
- 🎯 Campaign #6 (targeting Berlin) had **highest ROI per euro spent**.  
- 📉 Facebook campaign led to **returns spike** — possibly oversold product quality.  
- ⏳ Mobile clicks converted **slower than desktop** — often over **3+ days delay**.  
- 📦 Bulk orders came from **search campaigns**, not social.  
- 🧍 Repeat customers often **ignored campaigns** — bought on habit or reminders.  

<br>

## 🔗 5 Grouped Insights

| Theme                       | Pattern                                                             |
|----------------------------|---------------------------------------------------------------------|
| 🛒 High clicks ≠ high value | Instagram was flashy but not profitable.                           |
| ⌛ Delayed conversions       | Most purchases happen **24–72 hrs** after click.                   |
| 🧭 Channel fit matters       | Email = loyalty, Search = large basket, Social = browsing.         |
| 🔂 Multi-touch is real      | Repeat exposure increases conversion chances.                      |
| 🤖 Organic buyers exist      | Some customers convert **without any campaign touch** at all.      |

<br>

## 🎯 2 Strategic Decision Points

> **“Should we credit campaigns only for immediate conversions?”**  
> → No — attribution windows must account for **delayed purchases** and **multi-touch logic**.

> **“Are we spending the most where we’re earning the least?”**  
> → Rebalance budgets based on **ROI per channel**, not just engagement or clicks.



# 📅 Week 5: Everything Works... Until It Doesn’t

**Role:** You’re no longer just an analyst. You’ve just been promoted to **Analytics Engineer**.  
**Mission:** Refactor TrendHive’s fragile SQL setup into a modular DBT project.  
**Reality:** You’re inheriting a patchy pipeline built by 4 people, half documented, and none tested.

---

## 🔧 What You Discover (Real Problems, Real Fast)

- 📊 Revenue queries show **different totals across dashboards** — and no one knows why.  
- 🔁 Six copies of the same JOIN logic across different queries — each with a unique bug.  
- 📦 Filters for “active products” are written 5 different ways — leading to misaligned KPIs.  
- 🧱 Staging models are nonexistent — everyone queries raw CSVs or views.  
- ⚠️ A dashboard broke last week... but with **no data tests**, no one noticed.  
- 🧾 Returns are sometimes **added back to inventory**, sometimes not.  
- 🕵️ No one can answer: “Where is this number coming from?”  
- ⛔ A junior accidentally deleted 3 months of logic thinking it was “unused.”  
- 📉 Shipping KPIs don’t match operations reality — fulfillment delays go unreported.  
- 🧮 Even margin calculation changes slightly depending on who runs the report.

---

## 🧠 Why It Matters

You’re not solving one business problem —  
You’re solving **how your company solves problems.**

If your DBT models are:

- ❌ messy  
- ❌ duplicated  
- ❌ undocumented  
- ❌ untested  

Then nothing else downstream — not dashboards, not decisions — will ever be trustworthy.

---

## 🛠️ What You’ll Do This Week

- Audit existing SQL logic  
- Build a layered DBT project: **sources → staging → marts**  
- Add reusable macros for key metrics  
- Write tests for nulls, joins, and business rules  
- Document every model clearly

---

## 🎯 End Goal

By the end of the week, TrendHive’s SQL will feel like a **well-oiled machine**:  
- 🧱 Clean models  
- 📐 Standardized KPIs  
- 🧪 Tests that catch problems  
- 🧾 Dashboards that execs actually trust

---

> **Final Reflection Prompt:**  
> “What does reusable SQL mean to you — and how did it change your confidence this week?”

---

# 📅 Week 6: The CFO Is Waiting in Room 4B

**Role:** Lead Analyst  
**Mission:** Present your first **executive dashboard**.  
**Reality:** The CFO doesn’t trust last quarter’s numbers and needs a fresh view before Friday’s board meeting.

---

## 🧨 The Problems Stack Up Fast

- 📉 Margin report shows different numbers than what marketing presented.  
- 🔁 CLV formula doesn’t account for **refunded orders** — inflates performance.  
- ⏳ Shipping delays spiked in January, but no one reported it because the field was null.  
- 🧾 Product cost increased in Snowflake but wasn’t updated in DBT models.  
- 🔗 Attribution logic is a mess — some campaigns double count revenue.  
- 🕳️ Returns data is missing from `returns_2023_q4.csv` — no alerts fired.  
- 🔐 Power BI dashboard is broken due to permission changes in Snowflake.  
- 🧠 Region segmentation is needed — but there’s **no `region` field** in the model.  
- 🗂️ Finance, marketing, and product each use **separate pipelines** for the same KPIs.  
- 🫥 And you? You’re the only one who understands the pipeline.  

---

## 🧭 What the CFO Actually Wants

- 📊 Revenue by channel  
- 💰 Profit margin by category  
- 🔁 CLV by loyalty tier  
- 📦 Return rate trends  
- 🚚 Delays vs NPS risk  
- 📉 Inventory turnover by SKU class  
- 🧾 Campaign ROI (but only attributed revenue)  
- 🧍 Top 10% of customers by total profit  

All of it **accurate**, **drillable**, and **updated daily**.

---

## 🛠️ What You’ll Do This Week

- Finalize and QA all DBT models  
- Clean up logic across finance + ops  
- Build and deploy the Power BI dashboard  
- Write a 1-pager that answers:  
  > “If the CFO asks: how do you know this is right?”  
- Record a short walkthrough (async, optional)

---

## 🎯 End Goal

- A working executive dashboard  
- A tested, trusted pipeline  
- A confident response to high-level questions  
- A system the entire company can build on

---

> **Final Reflection Prompt:**  
> “When did you feel the switch from answering questions… to being trusted with decisions?”


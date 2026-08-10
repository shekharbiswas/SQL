# Day 1 Task: Understanding Olist, Scoping the Dashboard, and Getting Data Into Power BI

**Total time: 6 hours**

Four build days stand between today and an 11am Friday presentation. Today isn't about Power BI mechanics — that part's familiar already — it's about getting fast, real fluency in a dataset seen for the first time, locking in a scope that's actually achievable by Thursday, and getting a first clean table into Power BI so tomorrow can start on visuals immediately.

Read `week4_day1_concepts.md` first if you haven't — this task assumes the Olist table list and the Import-mode reasoning from that file.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Getting oriented in the Olist schema | 75 min |
| 2 | Scoping the dashboard for Friday | 45 min |
| 3 | Writing the SQL: building the summary table | 90 min |
| 4 | Exporting and connecting Power BI | 45 min |
| 5 | Model: relationships and date handling | 45 min |
| 6 | Sanity check and reflection | 30 min |

---

## Block 1: Getting Oriented in the Olist Schema (75 min)

**Goal:** Understand what's actually in each table before writing a single JOIN.

### Do this: 7 tasks

1. Download the Olist dataset from Kaggle and load all eight or nine CSVs into a fresh DuckDB database, one table per file.
2. Open the entity-relationship diagram on the Kaggle dataset page and keep it visible for reference.
3. For each table, run a quick `SELECT * ... LIMIT 10` and a `SELECT COUNT(*)`, noting the grain of each — what does one row actually represent?
4. Identify the join keys connecting the tables: `order_id`, `customer_id`, `product_id`, `seller_id` — and confirm with a quick query that they actually match across tables (e.g., do all `order_items.order_id` values exist in `orders`?).
5. Check `order_items` and `order_payments` specifically for one-to-many behavior relative to `orders` — run a query counting rows per `order_id` in each and confirm your assumption about the grain.
6. Look at `products` and `product_category_name_translation` together, and confirm how the Portuguese category names map to English ones.
7. Note anything that looks off — nulls in unexpected places, an unfamiliar status value in `orders`, a category that didn't translate — as things to handle explicitly in Block 3, not ignore.

---

## Block 2: Scoping the Dashboard for Friday (45 min)

**Goal:** Decide, in writing, exactly what the dashboard will answer — before any more SQL gets written.

### Do this: 5 tasks

1. Based on what Block 1 turned up, write down two or three specific business questions the dashboard will answer (e.g., revenue trend over time by category, delivery time vs. review score, top/bottom states or categories by performance).
2. For each question, list the specific tables and columns it needs — this becomes the contract for Block 3's SQL.
3. Explicitly write down what's *out of scope* for this project — for example, geolocation mapping, or payment-method-level analysis — so it stops competing for time later in the week.
4. Sanity-check the scope against the remaining time: three build days left, one of which (Thursday) is polish and export, not new analysis. If the scope feels heavy for that, cut a question now rather than on Wednesday.
5. Save this scope as a short written note in the project repo — it's also the backbone of Thursday's reflection and Friday's presentation narrative.

---

## Block 3: Writing the SQL — Building the Summary Table (90 min)

**Goal:** Turn the raw, joined Olist tables into one or two clean, flat tables that directly serve Block 2's questions.

### Do this: 7 tasks

1. Write a query joining `orders`, `order_items`, `customers`, `products`, and `product_category_name_translation` into a single order-item-level result, with category in English.
2. Join in `order_payments`, aggregating to one payment total per order where multiple payment rows exist, so the join doesn't silently duplicate order-item rows.
3. Join in `order_reviews`, handling orders with no review or more than one review row explicitly (`COALESCE`, or picking the most recent review per order) rather than letting a join quietly drop or duplicate rows.
4. Add a delivery-time calculation (delivered date minus purchase date) directly in this query, so it's ready to use as a measure without extra work in Power BI.
5. Filter out or explicitly flag orders with a status that shouldn't count toward revenue (e.g., canceled orders), based on what Block 1 found in the `orders.order_status` values.
6. Save the finished query as a `.sql` file in the project repo, and run it to produce the summary table this week's dashboard will be built on.
7. Spot-check five rows of the result against the raw tables directly, confirming revenue, category, state, delivery time, and review score all look correct for those orders.

---

## Block 4: Exporting and Connecting Power BI (45 min)

**Goal:** Get today's summary table into Power BI, cleanly.

### Do this: 5 tasks

1. Export the summary table (and a second one too, if Block 2's scope genuinely needs two separate grains — e.g., an order-level table and a lighter daily-aggregate table) to Parquet using `COPY ... TO ... (FORMAT PARQUET)`.
2. Open Power BI Desktop, start a new report, and load the exported file(s) via **Get Data → Parquet** or **Get Data → Folder**.
3. In Power Query, confirm data types landed correctly — dates as dates, numeric columns as numbers, not text — since this determines whether tomorrow's charts and slicers behave correctly.
4. Rename any messily-auto-named queries, then **Close & Apply**.
5. Confirm the row count in Power BI's Data view matches the row count of the exported table in DuckDB.

---

## Block 5: Model — Relationships and Date Handling (45 min)

**Goal:** Get the model into a state Day 2 can build on directly, with no relationship or date-table cleanup left to do mid-dashboard.

### Do this: 5 tasks

1. If Block 4 produced more than one table, build the relationship between them in Model view now, checking the cardinality Power BI infers against what's actually true of the data.
2. If the summary table has a proper date column (order purchase date), decide whether it needs a separate date dimension table or can be used directly — for a single-table model, a direct date column is often enough; only build a separate date table if Block 2's questions need real time-intelligence calculations.
3. If you do build a separate date table, mark it as the official date table (Model view → right-click → "Mark as date table").
4. Either way, confirm a quick date-based slicer behaves correctly — continuous range, correct sort order, no text-sorted dates.
5. Write one sentence on why you did or didn't need a separate date table this time, compared to `dim_date_spine` in the bikeshare project.

---

## Block 6: Sanity Check and Reflection (30 min)

### Do this: 3 tasks

1. Build one temporary table visual pulling several columns from the model, spot-check it against the SQL summary table directly, then delete it before saving.
2. Save the `.pbix` file with a clear name identifying it as this week's Olist project.
3. Answer the reflection questions below and save them alongside the scope note from Block 2.

**Reflection questions:**

1. Given what today's exploration turned up, does Block 2's scope still feel achievable by Thursday, or does something need to be cut now rather than discovered mid-week?
2. Compare building today's summary table to Week 1's first joined table — what's different about doing this with no schema doc provided by the course, and no prior queries to lean on?
3. With Friday 11am fixed, what's the single biggest risk to being ready in time, and what's the plan for catching it early rather than on Thursday night?
4. Based on the shape of today's summary table, which specific visual do you expect to be the most natural fit tomorrow, and which do you expect to take the most extra work?

**Deliverable:** A `.sql` file with the join/summary-table query from Block 3, the resulting table(s) exported to Parquet, a `.pbix` file with that data loaded via Import mode (relationships and date handling resolved, no leftover sanity-check visuals), and a short written project scope from Block 2 saved in the repo.
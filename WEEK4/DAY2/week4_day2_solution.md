# Day 2 Solution: Measures, Trend, Category, and State Charts

Worked solution for `week4_day2_task.md`, building on `orders_summary` from Day 1.

## Block 1 — Explicit DAX Measures

```dax
Total Revenue = SUM(orders_summary[item_revenue])

Order Count = DISTINCTCOUNT(orders_summary[order_id])

Avg Review Score = AVERAGE(orders_summary[review_score])

Avg Delivery Days = AVERAGE(orders_summary[delivery_days])
```

`Order Count` uses `DISTINCTCOUNT` rather than a row count, because `orders_summary` sits at order-item grain — a plain `COUNTROWS` would overcount any order with more than one item.

## Block 2 — Revenue Trend, Monthly Grain

```dax
Order Month = FORMAT(orders_summary[order_purchase_timestamp], "YYYY-MM")

Order Month Sort =
YEAR(orders_summary[order_purchase_timestamp]) * 100
    + MONTH(orders_summary[order_purchase_timestamp])
```

Set `Order Month`'s "Sort by Column" property to `Order Month Sort` (Data view → select `Order Month` → Column tools → Sort by Column) so months sort chronologically rather than alphabetically. Build the line chart with `Order Month` on the axis and `Total Revenue` as the value; add `Order Count` as a second line or tooltip field.

Solution title: **"Monthly Revenue, Sept 2016–Aug 2018 — Volume Ramps Up From Early 2017"** (exact wording depends on the dataset's real min/max dates, confirmed with `MIN`/`MAX` on `order_purchase_timestamp`).

## Block 3 — Category Bar Chart, Top N

```dax
Category Revenue Rank =
RANKX(
    ALL(orders_summary[category_english]),
    [Total Revenue]
)

Category Group (Top 10) =
VAR CurrentCategoryRevenue = [Total Revenue]
VAR CurrentRank =
    RANKX(
        ALL(orders_summary[category_english]),
        CALCULATE([Total Revenue], ALLEXCEPT(orders_summary, orders_summary[category_english]))
    )
RETURN
    IF(CurrentRank <= 10, SELECTEDVALUE(orders_summary[category_english]), "Other")
```

Simpler alternative used in practice: apply Power BI's built-in **Top N** visual-level filter on the category bar chart (Filters pane → Filter type: Top N → 10 → By value: Total Revenue) rather than a custom DAX bucket — faster to set up and equally correct for a static Top N chart. The DAX version above is worth knowing for cases needing an explicit "Other" bar in the same visual.

Sort the chart by `Total Revenue` descending (chart's own sort control, not alphabetical).

## Block 4 — State Bar Chart

```dax
-- No bucketing needed for 27 states; built directly:
```
Bar chart: `customer_state` on axis, `Total Revenue` as value, sorted descending, `Order Count` as tooltip. If the full 27-state chart still feels crowded, apply the same Top N filter pattern as Block 3.

## Block 5 — Spot-Check Queries

```sql
-- Top category check
SELECT category_english, SUM(item_revenue) AS total_revenue
FROM orders_summary
GROUP BY category_english
ORDER BY total_revenue DESC
LIMIT 10;

-- One state check
SELECT customer_state, SUM(item_revenue) AS total_revenue
FROM orders_summary
WHERE customer_state = 'SP'  -- swap in the state being checked
GROUP BY customer_state;

-- One month check
SELECT
    STRFTIME(order_purchase_timestamp, '%Y-%m') AS order_month,
    SUM(item_revenue) AS total_revenue
FROM orders_summary
GROUP BY order_month
ORDER BY order_month;
```

Run each of these and compare against the equivalent number shown on the matching Power BI visual — they should match exactly since `orders_summary` is the single source both are built from.

## Sample Reflection Answers

1. The daily-grain version showed a long, noisy near-zero stretch for the dataset's first several months before volume picked up — switching to monthly grain smoothed this into a readable ramp-up rather than a chart that looked broken.
2. The category chart needed the most cardinality management — roughly 70 raw categories were unreadable as a bar chart before applying the Top N filter.
3. Yes — an early version of `Order Count` used `COUNTROWS` instead of `DISTINCTCOUNT`, overcounting multi-item orders; traced back to the measure definition in Block 1 and fixed there rather than adjusting any visual.
4. Slicers and cross-filtering are expected to be straightforward since the model is a single flat table; the delivery-time-vs-review-score visual is expected to need real thought around bucket boundaries.

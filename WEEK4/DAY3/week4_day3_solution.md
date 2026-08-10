# Day 3 Solution: Slicers, Interactions, and Delivery Time vs. Review Score

Worked solution for `week4_day3_task.md`.

## Block 1 — Slicers

Built directly on `orders_summary` fields, no DAX required:
- Date slicer on `order_purchase_timestamp` (between-range slicer style)
- Category slicer on `category_english` (consider limiting to the Top 10 set from Day 2 using a slicer-level Top N filter, or leave full list if the slicer's own scroll makes 70 categories manageable)
- State slicer on `customer_state`

## Block 2 — Interactions

In Format → Edit Interactions:
- All three slicers → set to **Filter** on every visual on the page (default, confirmed rather than assumed).
- Category bar chart ↔ State bar chart → set to **None** (clicking a category bar shouldn't cross-filter the state chart) — these are peer breakdowns of the same total, and cross-filtering between them tends to confuse rather than clarify during a live walkthrough.
- Revenue trend line chart → left as **Filter** target from the bar charts, since "what does this category's trend look like over time" is a genuinely useful click-to-explore interaction.

## Block 3 — Delivery-Time Bucket

```dax
Delivery Bucket =
SWITCH(
    TRUE(),
    orders_summary[delivery_days] <= 3, "0-3 days",
    orders_summary[delivery_days] <= 7, "4-7 days",
    orders_summary[delivery_days] <= 14, "8-14 days",
    orders_summary[delivery_days] > 14, "15+ days",
    "Unknown"
)

Delivery Bucket Sort =
SWITCH(
    orders_summary[Delivery Bucket],
    "0-3 days", 1,
    "4-7 days", 2,
    "8-14 days", 3,
    "15+ days", 4,
    5
)
```

Set `Delivery Bucket`'s "Sort by Column" to `Delivery Bucket Sort` so buckets display in the correct order rather than alphabetically (which would put "0-3 days" before "15+ days" but "4-7 days" after "15+ days").

Bucket boundaries were chosen after checking quartiles:

```sql
SELECT
    approx_quantile(delivery_days, 0.25) AS p25,
    approx_quantile(delivery_days, 0.50) AS median,
    approx_quantile(delivery_days, 0.75) AS p75,
    approx_quantile(delivery_days, 0.90) AS p90
FROM orders_summary;
```

Typical Olist result: median delivery is roughly 10–12 days, with a long right tail — which is why "15+ days" is left as an open-ended bucket rather than capped, and why "0–3 days" (fast, but a small slice of orders) is kept separate from "4–7 days" rather than merged.

## Block 3, Task 6 — Volume Check

```sql
SELECT
    "Delivery Bucket",
    COUNT(*) AS order_count
FROM (
    SELECT
        CASE
            WHEN delivery_days <= 3 THEN '0-3 days'
            WHEN delivery_days <= 7 THEN '4-7 days'
            WHEN delivery_days <= 14 THEN '8-14 days'
            ELSE '15+ days'
        END AS "Delivery Bucket"
    FROM orders_summary
)
GROUP BY "Delivery Bucket"
ORDER BY 1;
```

Confirm no bucket has a suspiciously small count (a bucket under roughly 1% of total orders is too thin to present as a reliable pattern).

## Block 4 — The Chart

Column chart: `Delivery Bucket` (sorted via Task above) on axis, `Avg Review Score` as value, `Order Count` as tooltip.

Solution title: **"Average Review Score by Delivery-Time Bucket — Scores Drop Once Delivery Passes ~2 Weeks"**

Caption for the presentation script:
> "Orders that arrive within a week average noticeably higher review scores than orders taking two weeks or more. This is an association, not proof that slow delivery alone causes the lower score — other factors (seller reliability, distance, damaged shipments) likely move with delivery time too."

## Sample Reflection Answers

1. The state slicer turned out more useful than expected — several states show meaningfully different average delivery times, which wasn't obvious from the aggregate. The category slicer added less on its own since Day 2's Top N chart already surfaces the same information.
2. The category-vs-state interaction was the one default worth overriding — left as "Filter," clicking a category bar was cross-filtering the state chart in a way that made the two breakdowns look causally linked rather than two independent views of the same total.
3. Bucket boundaries were set from the actual quartiles (median ~10-12 days) rather than round numbers, so "8-14 days" roughly captures the middle of the distribution instead of splitting it arbitrarily.
4. Presentation sentence: "Orders delivered within a week score noticeably higher on average than orders taking two-plus weeks — worth watching, though this shows correlation, not a proven cause." This holds up to the caution in Day 3 Concepts since it explicitly avoids claiming delivery time causes the score difference.

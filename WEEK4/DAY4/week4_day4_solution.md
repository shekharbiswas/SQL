# Day 4 Solution: Theme, Titles, Layout, and Presentation Outline

Worked solution for `week4_day4_task.md`. This day is mostly design judgment rather than code, so this file gives concrete examples rather than a single "correct" answer.

## Block 1 — Sample Theme JSON

Power BI accepts a custom theme file (View → Themes → Browse for themes) in this format — a minimal starting point:

```json
{
  "name": "Olist Dashboard",
  "dataColors": [
    "#2E5266", "#6E8898", "#D9BF77", "#C44536", "#9AA6B2"
  ],
  "background": "#FFFFFF",
  "foreground": "#1A1A1A",
  "tableAccent": "#2E5266",
  "visualStyles": {
    "*": {
      "*": {
        "title": [{ "fontSize": 14, "bold": true }],
        "legend": [{ "fontSize": 10 }]
      }
    }
  }
}
```

`#C44536` (a muted red) is reserved as the single accent color for the headline revenue figure per the concepts file's guidance; the other four colors rotate across category/state bars.

## Block 2 — Sample Titles (Before → After)

| Chart | Before | After |
|---|---|---|
| Trend | "Total Revenue by Order Date" | "Monthly Revenue, 2016–2018 — Volume Ramps Up From Early 2017" |
| Category | "Revenue by Category" | "Top 10 Categories by Revenue — Health & Beauty Leads" |
| State | "Revenue by State" | "Revenue by State — São Paulo Drives Over a Third of Total Sales" |
| Delivery/Review | "Avg Review Score by Delivery Bucket" | "Review Scores Drop Once Delivery Passes ~2 Weeks" |

(Exact leading category/state will depend on the actual data pulled — confirm the real top performer before finalizing these titles rather than assuming Health & Beauty/São Paulo.)

## Block 3 — Layout Guidance

A working four-visual layout:

```
┌─────────────────────────────────────────────┐
│  Page title + one-line subtitle              │
├───────────────────────────┬───────────────────┤
│  Revenue trend (line)      │  KPI cards:        │
│  full width, top half      │  Total Revenue      │
│                             │  Order Count         │
│                             │  Avg Review Score    │
├──────────────┬──────────────┴───────────────────┤
│  Category Top10  │  State bar    │ Delivery/Review │
│  (bar)            │  (bar)       │  (column)        │
└──────────────┴──────────────┴───────────────────┘
   Slicers pinned in a left rail or a top filter bar, consistent across the page.
```

## Block 5 — Sample Presentation Outline

> **Slide/section 1 — Revenue trend.** "Revenue grew steadily from early 2017 through mid-2018, tracking overall order volume rather than any single spike." *(Chart: monthly trend line)*
>
> **Slide/section 2 — Category and state performance.** "A handful of categories and São Paulo specifically account for a disproportionate share of revenue — worth prioritizing in any category- or region-specific strategy." *(Chart: Top 10 category bar + state bar)*
>
> **Slide/section 3 — Delivery time and reviews.** "Orders delivered within about two weeks score noticeably better on average than slower ones. This is a correlation worth watching operationally, not a proven cause — other factors likely move with delivery time too." *(Chart: delivery bucket column chart)*
>
> **Closing.** "With more time, the next step would be separating out seller-level performance from category/state effects, since right now those are somewhat entangled in a single flat table."

## Sample Written Reflection

1. The biggest shift was in Block 1 on Day 1 — the assumption going in was that Olist would behave like a single flat table similar to Capital Bikeshare's `trips`; discovering how much the grain varies across `order_items`, `order_payments`, and `order_reviews` changed how carefully every join needed to be checked.
2. With a full week and a dbt-style pipeline, the summary table would likely be split into proper staging models per source table plus a tested mart, with `dbt test` catching the payment/review duplication risk automatically instead of relying on a manual spot-check.
3. The Top N filter on the category chart made the biggest visible difference for the least effort — a two-minute filter turned an unreadable 70-bar chart into something presentable.
4. Knowing what's known now, the delivery-time bucket boundaries would be set from the quartile check on Day 1 rather than Day 3 — it would have shaped the summary table's columns earlier and saved a bit of Day 3 rework.

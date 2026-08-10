# Day 2 Concepts: From a Flat Table to Bar and Line Charts

**Estimated reading time: 30 min**

Day 1 ended with one clean, joined Olist table sitting in Power BI, related correctly, no visuals built yet. The mechanics of adding a bar or line chart aren't new — that part's already second nature. What's worth slowing down for today is the handful of decisions that are specific to *this* table and *this* deadline: which measures need to be explicit rather than implicit, how to handle a category field with dozens of values, and how to avoid a revenue trend chart that quietly lies because of how sparse Olist's early months are.

## Explicit Measures First, Visuals Second

Day 1's summary table sits at order-item grain. Every chart built directly on its raw columns will implicitly aggregate — usually fine, but it means the same "Total Revenue" logic gets silently redefined every time a new visual is dropped onto the canvas, field by field, with no single place to check or fix it. Before building anything today, it's worth writing a small number of explicit DAX measures once — `Total Revenue`, `Order Count`, `Avg Review Score`, `Avg Delivery Days` — and building every visual from those, not from raw columns. This is the same instinct as centralizing logic in a dbt mart instead of repeating a `CASE WHEN` in five different queries; it's just happening in DAX now instead of SQL.

One definitional decision has to be made explicitly and then never revisited quietly: does "revenue" mean item price alone, or price plus freight? Whichever gets chosen for the `Total Revenue` measure needs to be the same number used everywhere — trend chart, category chart, state chart — or Friday's numbers won't reconcile with each other if someone asks.

## The Revenue Trend Line Chart: Granularity Matters

Olist's order volume ramps up sharply over its date range — the first few months have very few orders. A daily-grain line chart will show a noisy, near-flat line for months before jumping to a real pattern, which reads as broken rather than as sparse. Aggregating to monthly grain (or explicitly starting the chart from the point volume stabilizes) produces something readable and honest. Either choice is defensible; what matters is making it on purpose today and being ready to explain it Friday if asked why the chart doesn't start at the dataset's actual first date.

## Category and State Charts: Managing Cardinality

Olist has on the order of 70 product categories and 27 Brazilian states. A bar chart with 70 bars isn't a chart, it's a list. The standard move — Top N by revenue, with everything else collapsed into an "Other" bucket — is worth setting up as a genuine Top N (Power BI's built-in Top N filter, or a DAX rank measure) rather than eyeballing which bars look biggest. States are a smaller set and might not need bucketing, but the same principle applies: pick a number of categories the audience can actually read at a glance, and be deliberate about it rather than defaulting to "however many happen to fit."

## What Belongs in a Table vs. What Belongs in a Chart

Not every number needs a bar. A quick check worth doing before building each visual: does this comparison need position/length encoding (a bar chart, a line), or would a simple KPI card or table communicate it just as clearly with less visual weight? Three or four charts, well chosen, will read better on Friday than eight charts competing for attention.

## What's Next

Today's task builds the explicit measures and the first three visuals — a revenue trend line, a category bar chart, and a state bar chart — cross-checked against the SQL summary table from Day 1. Day 3 adds interactivity (slicers, cross-filtering) and a second analytical angle: delivery time against review score.
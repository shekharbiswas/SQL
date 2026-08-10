# Day 3 Concepts: Interactivity, and a Second Analytical Angle

**Estimated reading time: 30 min**

Yesterday's three charts are correct but static. Today does two things: makes the dashboard genuinely explorable (slicers, cross-filtering), and adds the analytical angle Day 1's scope note flagged but didn't build yet — the relationship between delivery time and review score. Both parts are mechanically familiar; what's worth thinking through today is how they behave specifically on a single flat table, and how to present a correlation without overstating it.

## Slicers and Cross-Filtering on a Single-Table Model

Because Day 1 deliberately kept the model to one (or two) flat table(s) rather than building out a full star schema, filter propagation is simpler than it was in the bikeshare project — there's no relationship chain for a filter to travel across, since everything mostly lives in one place. That's a genuine advantage this week: a slicer on category or state filters every visual on the page automatically, with none of the "does this relationship even connect these two tables" debugging that a multi-table model can require. The judgment call today isn't about relationships, it's about which slicers actually serve the three questions from Day 1's scope note — a slicer that doesn't change how anyone reads the dashboard is just clutter.

Edit Interactions (Format → Edit Interactions) is worth a deliberate pass rather than leaving Power BI's defaults in place. Not every visual should filter every other visual by default — a slicer filtering the whole page makes sense, but two peer charts filtering each other on click can get confusing fast if it isn't intentional.

## Delivery Time vs. Review Score: What the Chart Can and Can't Claim

A scatter plot (or a binned bar chart — delivery-time buckets against average review score) showing that longer delivery times associate with lower review scores is one of Olist's most reliable, presentation-ready findings. It's also an easy place to overclaim. The chart shows association, not proof that slow delivery *causes* bad reviews — reviews could be depressed by other factors that also slow delivery (a struggling seller, a remote destination, a damaged package). Friday's presentation will land better with "delivery time and review score move together, and here's the pattern" than with a causal claim the data can't actually support. Deciding the phrasing today, while building the chart, is easier than improvising it under presentation pressure Friday.

Binning delivery time into a small number of buckets (e.g., 0–3 days, 4–7, 8–14, 15+) usually reads more clearly than a raw scatter of every order — fewer points, a clearer trend line, and buckets map more naturally to something a viewer can act on ("orders past two weeks see a real drop").

## What's Next

Today's task builds the slicers, sets deliberate interactions, and adds the delivery-time-vs-review-score visual with its supporting measure. Thursday shifts entirely to polish — no new analysis, just making everything built this week and yesterday presentable.
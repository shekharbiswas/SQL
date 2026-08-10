# Day 3 Task: Slicers, Interactions, and Delivery Time vs. Review Score

**Total time: 6 hours**

Today makes yesterday's static charts explorable, and adds the second analytical thread from Day 1's scope note. By the end of today, the dashboard should have working slicers, deliberately-set interactions, and a delivery-time-vs-review-score visual — everything needed content-wise before Thursday's polish pass.

Read `week4_day3_concepts.md` first — this task assumes the binning and correlation-phrasing reasoning from that file.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Building slicers | 60 min |
| 2 | Setting deliberate interactions | 45 min |
| 3 | Delivery-time bucket measure | 75 min |
| 4 | Delivery-time vs. review-score visual | 60 min |
| 5 | Integrating into yesterday's layout | 75 min |
| 6 | Reflection | 45 min |

---

## Block 1: Building Slicers (60 min)

**Goal:** Add slicers that serve Day 1's scope questions, not slicers for their own sake.

### Do this: 5 tasks

1. Revisit Day 1's scope note and, for each of the two or three business questions, identify what someone would actually want to filter by while exploring it.
2. Build a date range slicer on order date.
3. Build a category slicer, using the same Top N logic as yesterday's bar chart if the category field has high cardinality.
4. Build a state slicer.
5. Test each slicer individually against yesterday's three charts, confirming the numbers update correctly.

---

## Block 2: Setting Deliberate Interactions (45 min)

**Goal:** Decide, visual by visual, what should filter what — rather than leaving Power BI's defaults unexamined.

### Do this: 4 tasks

1. Open Format → Edit Interactions and review, for every pair of visuals, what the current default interaction is.
2. For each slicer, confirm it's set to filter every relevant visual on the page.
3. For the category and state bar charts, decide deliberately whether clicking a bar should cross-filter the other chart, and set it explicitly rather than leaving it ambiguous.
4. Write one sentence per visual pair where you changed the default, explaining why.

---

## Block 3: Delivery-Time Bucket Measure (75 min)

**Goal:** Turn raw delivery-time-in-days into buckets that read clearly in a chart.

### Do this: 6 tasks

1. Confirm the delivery-time calculation from Day 1's SQL summary table is available as a column in the model.
2. Decide on bucket boundaries (e.g., 0–3, 4–7, 8–14, 15+ days) based on the actual distribution — check delivery-time quartiles first rather than guessing round numbers.
3. Write a DAX calculated column or measure that assigns each order to a bucket.
4. Confirm the buckets sort in the correct order in a visual (not alphabetically) by setting a sort-by column if needed.
5. Spot-check five orders against their raw delivery-time value to confirm they landed in the correct bucket.
6. Add `Order Count` per bucket as a quick table visual, checking that the buckets have enough volume each to be meaningful (a bucket with three orders isn't a reliable pattern).

---

## Block 4: Delivery-Time vs. Review-Score Visual (60 min)

**Goal:** Build the chart that shows the relationship, phrased carefully.

### Do this: 5 tasks

1. Build a bar or column chart with the delivery-time bucket on one axis and `Avg Review Score` as the value.
2. Add `Order Count` as a tooltip or secondary label, so a bucket's reliability is visible alongside its average score.
3. Title the chart in language that describes association, not causation (e.g., "Average Review Score by Delivery-Time Bucket" rather than "How Delivery Time Affects Reviews").
4. Write a one-sentence caption or note for the presentation script explicitly naming the correlation-vs-causation caveat from Day 3 Concepts.
5. Confirm the chart responds correctly to the slicers built in Block 1.

---

## Block 5: Integrating Into Yesterday's Layout (75 min)

**Goal:** Bring today's additions into a coherent page with yesterday's three charts, without yet doing final formatting.

### Do this: 5 tasks

1. Place the slicers in a consistent location (commonly a left rail or top bar) rather than scattered around the canvas.
2. Add today's delivery-time-vs-review-score chart into the layout alongside yesterday's three.
3. Click through every slicer and confirm all four charts (trend, category, state, delivery/review) update correctly together.
4. Note any visual that feels crowded or redundant now that everything's on one page — flag it for Thursday rather than fixing it today.
5. Save the `.pbix` file.

---

## Block 6: Reflection (45 min)

Answer in a few sentences each:

1. Which slicer, once built, turned out to be more useful for exploring the data than expected — and was there one that turned out not to add much?
2. Block 2 asked you to set interactions deliberately rather than leaving defaults. What's one pair of visuals where the default would have been misleading if left alone?
3. How did you land on your delivery-time bucket boundaries, and what would change about the chart's story if you'd picked wider or narrower buckets?
4. Write the sentence you'd actually say Friday morning when presenting the delivery-time-vs-review-score chart — does it hold up to the correlation-vs-causation caution from today's concepts file?

**Deliverable:** A `.pbix` file with working date/category/state slicers, deliberately-set visual interactions, a delivery-time bucket measure, and a delivery-time-vs-review-score chart — all four of this week's core visuals now on one page and responding correctly to the slicers.
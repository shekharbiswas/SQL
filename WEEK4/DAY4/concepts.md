# Day 4 Concepts: Polish, and Building the Presentation Narrative

**Estimated reading time: 25 min**

## Two Different Objectives, Both Due Tomorrow

Today's work serves two separate goals, and it's worth knowing both clearly before starting, since they pull in slightly different directions:

**The project objective** is the dashboard itself — proof that the SQL and Power BI skill built across this course transfers to a dataset with no course-provided schema, no prior queries, and a fixed deadline. By tomorrow at 11am, that means a working, interactive dashboard that actually answers Day 1's three scope questions: the revenue trend, category/state performance, and delivery time vs. review score.

**The presentation objective** is different, and it's not just "show the dashboard." It's demonstrating *why* it looks the way it does — why monthly grain instead of daily, why the category chart shows ten bars instead of seventy, why the delivery/review chart is framed as a correlation rather than a cause. Anyone can click through slicers. What tomorrow is actually testing is whether the reasoning behind each design decision can be explained clearly, on the spot, to someone who didn't build it.

Today's blocks are ordered with both of those in mind: Blocks 1–4 protect the project objective (a dashboard that's correct and looks finished), and Block 5's presentation narrative is where the presentation objective gets built deliberately, rather than improvised tomorrow morning. Going into today already knowing this split changes how the polish work gets prioritized — a title that states *why* a chart looks the way it does (Block 2, below) is doing double duty for both objectives at once.

By the end of Day 3, every visual the dashboard needs is built and working: revenue trend, category and state breakdowns, and the delivery-time-vs-review-score chart, all responding to slicers correctly. Today adds no new analysis. The only job today is making what already exists presentable — and, just as importantly, deciding what gets said about it Friday morning.

## Polish Is Subtraction, Not Addition

The temptation on a polish day is to add — another chart, another slicer, one more metric that seems interesting. Resist it. A dashboard someone can read in ten seconds beats one that's technically more complete but takes explaining. Today's real work is mostly removing: default gridlines, unnecessary borders, a legend that repeats what the axis already says, a fourth decimal place on a number nobody needs precise to four decimals.

## A Consistent Visual Language

Three things carry more weight than they seem to: color, typography, and alignment.

- **Color** should mean something and mean it consistently — if categories get colored in the bar chart, the same category should be the same color anywhere else it appears. A single accent color for "the number that matters" (this week's headline revenue figure, say) reads more confidently than five competing bright colors fighting for attention.
- **Typography** — chart titles should say what the chart shows, not just label the field ("Monthly Revenue Trend, 2017–2018" beats "Total Revenue by Order Date"). Keep title/subtitle/axis-label sizing consistent across every visual on the page.
- **Alignment** — visuals snapped to a grid, consistent spacing, and matching heights across a row read as intentional even before anyone processes the content. Power BI's built-in gridlines and "Align" tools under the Format ribbon handle this quickly once you're looking for it.

## Writing Titles That State the Insight

A chart titled with just its field names is a fact. A chart titled with what the fact means is a finding. "Revenue drops sharply after 8+ day delivery times" says more than "Avg Review Score by Delivery Bucket," even though it's the same chart underneath. Since Friday's audience only has a few minutes with each visual, a good title does real work — it's often the only line someone reads before the presenter starts talking.

## From Dashboard to Presentation Narrative

The dashboard being finished and the presentation being ready are two different things. With 11am Friday fixed, it's worth spending real time today turning Day 1's two-or-three scope questions into an actual spoken narrative: which chart answers which question, in what order, and what's the one sentence of takeaway for each. A short, rehearsed walkthrough — "here's what we found about revenue, here's what we found about delivery and reviews, here's what we'd look at next" — lands better than opening the file live and narrating on the fly.

## What's Next

Today's task file works through the formatting pass, a final interactivity check across everything built this week, exporting the `.pbix` and screenshot, and drafting the presentation narrative and written reflection. Friday morning is the presentation itself — Friday afternoon shifts entirely to Snowflake, watched and read rather than built.
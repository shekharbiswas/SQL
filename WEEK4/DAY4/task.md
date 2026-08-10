# Day 4 Task: Polish, Export, and Presentation Prep

**Total time: 6 hours**

This is the last build day before Friday 11am. Nothing new gets analyzed today — the goal is a dashboard that looks finished, a `.pbix` and screenshot ready to hand in, and a presentation narrative that's been thought through rather than improvised in the room.

Read `week4_day4_concepts.md` first — this task assumes the polish-as-subtraction and title-writing guidance from that file, including the split between the **project objective** (a correct, finished dashboard) and the **presentation objective** (being able to explain the reasoning behind it) that today is building toward.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Consistent theme and color | 60 min |
| 2 | Titles and labels that state the insight | 45 min |
| 3 | Final layout and decluttering pass | 60 min |
| 4 | Final interactivity check | 45 min |
| 5 | Export and presentation narrative | 90 min |
| 6 | Written reflection | 60 min |

---

## Block 1: Consistent Theme and Color (60 min)

**Goal:** One coherent visual language across every visual on the page.

### Do this: 5 tasks

1. Pick a small color palette (three to five colors plus a neutral) and apply it consistently — same category, same color, everywhere it appears.
2. Choose or build a Power BI theme (View → Themes) rather than styling each visual individually, so the palette applies consistently in one step.
3. Pick one accent color reserved for the single most important number on the page (e.g., the headline revenue figure), and make sure nothing else competes with it.
4. Remove default gridlines, borders, or backgrounds that aren't adding information.
5. Confirm font sizes are consistent across chart titles, and consistent (smaller) across axis labels.

---

## Block 2: Titles and Labels That State the Insight (45 min)

**Goal:** Every chart title says what the chart shows, not just what field it plots.

### Do this: 4 tasks

1. Rewrite the revenue trend chart's title to state the actual trend, not just "Revenue Over Time."
2. Rewrite the category chart's title to name the standout finding (e.g., which category leads), not just "Revenue by Category."
3. Rewrite the delivery-time-vs-review-score chart's title to state the pattern, using the correlation-safe phrasing from Day 3.
4. Check axis labels and legends for anything redundant with the title, and remove duplication.

---

## Block 3: Final Layout and Decluttering Pass (60 min)

**Goal:** A page that reads top-to-bottom or left-to-right without instructions.

### Do this: 5 tasks

1. Align all visuals to a consistent grid, matching heights and spacing where visuals sit in the same row.
2. Reorder or resize visuals so the most important finding (per Day 1's scope) is the most visually prominent, not just first-built.
3. Remove or simplify any tooltip, legend, or label that repeats information already visible elsewhere on the page.
4. Add a single page title and, if useful, a one-line subtitle stating the dashboard's overall purpose.
5. Step back and view the whole page at actual presentation size (not zoomed into Power BI's editor) to check it reads clearly from a distance.

---

## Block 4: Final Interactivity Check (45 min)

**Goal:** Confirm everything still works correctly after a day of formatting changes.

### Do this: 4 tasks

1. Click through every slicer again, confirming all visuals update correctly — formatting changes can sometimes break a visual's filter response.
2. Test extreme slicer combinations (e.g., a state with very few orders) and confirm visuals handle sparse data gracefully rather than showing a broken or empty state.
3. Confirm every chart title and label still matches what the chart is showing after Block 2's edits.
4. Fix anything Block 4 turns up before moving on.

---

## Block 5: Export and Presentation Narrative (90 min)

**Goal:** Produce the actual deliverables (project objective), and a rehearsed walkthrough that explains the reasoning behind each chart (presentation objective) for Friday morning.

### Do this: 6 tasks

1. Save the final `.pbix` file with a clear name.
2. Take the dashboard screenshot required as a deliverable, at a resolution that will actually be readable in the room or on the call Friday.
3. Write a short presentation outline: which chart answers which of Day 1's scope questions, in what order, with one takeaway sentence per chart.
4. Time a full run-through of the narrative out loud, adjusting if it runs long for the available slot.
5. Prepare for the one or two questions most likely to come up (e.g., "does slow delivery *cause* bad reviews?") with the correlation-vs-causation phrasing from Day 3 already in hand.
6. Export or save the presentation outline alongside the `.pbix` and screenshot.

---

## Block 6: Written Reflection (60 min)

Answer in a few sentences each — this is also the raw material for the presentation's closing remarks:

1. Walk through the project from Monday's schema exploration to today. What changed the most about your understanding of the data between Day 1 and now?
2. Compare this build to Week 3's dbt project. What would change about this dashboard if you had a full week (and a dbt-style pipeline) instead of four days against a dataset you'd never seen?
3. Which of today's polish decisions made the biggest visible difference for the least effort?
4. What's the one thing you'd do differently if this project started over today, knowing what you know now about Olist?

**Deliverable:** A finished `.pbix` file, a dashboard screenshot, a written presentation outline with a takeaway sentence per chart, and the written reflection — everything needed to walk into Friday 11am ready.
# Day 5 Task: Presenting, Then Studying Snowflake

**Total time: 5 hours**

The morning is the week's actual milestone: presenting the Power BI dashboard at 11am. The afternoon is deliberately light on doing and heavy on understanding — no account setup, no hands-on lab, just working through Snowflake's core ideas and writing them down in a way that connects back to dbt and DuckDB.

Read `week4_day5_concepts.md` before the afternoon session — it covers the storage/compute separation and the dbt comparison this task builds on.

## Time Budget

| Block | Topic | Time |
|-------|-------|------|
| 1 | Final presentation buffer | 30 min |
| 2 | Presentation + Q&A (11am) | 45 min |
| 3 | Snowflake architecture: storage/compute | 75 min |
| 4 | Virtual warehouses and scaling | 60 min |
| 5 | Written comparison: Snowflake vs. dbt vs. DuckDB | 60 min |
| 6 | Reflection | 30 min |

---

## Block 1: Final Presentation Buffer (30 min)

**Goal:** Walk into 11am with nothing left to figure out.

### Do this: 4 tasks

1. Open the `.pbix` file and confirm it opens cleanly, with no error or stale-data warning.
2. Do one final click-through of every slicer, confirming everything still works exactly as it did Thursday.
3. Re-read the presentation outline from Thursday's Block 5 once, out loud, at speaking pace.
4. Have the dashboard screenshot and any backup export ready in case of a display or connectivity issue in the room.

---

## Block 2: Presentation + Q&A (45 min)

**Goal:** Present the dashboard, using the narrative built Thursday.

### Do this: 3 tasks

1. Present the dashboard following Thursday's outline — one chart per scope question, one takeaway sentence each.
2. Field questions using the prepared correlation-vs-causation phrasing where relevant (especially for the delivery-time-vs-review-score chart).
3. Note down, right after, any question that came up that wasn't anticipated — useful for future projects, and a natural bridge into this afternoon's reflection.

---

## Block 3: Snowflake Architecture — Storage/Compute Separation (75 min)

**Goal:** Build a correct mental model of the core architectural idea, from primary sources.

### Do this: 5 tasks

1. Read Snowflake's own "Key Concepts & Architecture" documentation in full.
2. In your own words (not copied from the docs), write two or three sentences explaining what "separation of storage and compute" actually means and why it matters.
3. Read or skim the "Snowflake in 20 Minutes" tutorial, paying attention to the objects it creates (database, table, virtual warehouse) even without running any of it.
4. Note which of those objects have a direct equivalent in what's already familiar from this course (e.g., a Snowflake database vs. a DuckDB file) and which don't have a clean equivalent at all.
5. Write down one question this reading raised that you'd want answered before ever actually building something on Snowflake.

---

## Block 4: Virtual Warehouses and Scaling (60 min)

**Goal:** Understand what a virtual warehouse actually is, and why elastic compute matters.

### Do this: 4 tasks

1. Find and read the section of Snowflake's documentation (or a tutorial video, if preferred) covering virtual warehouses specifically — sizing, auto-suspend, auto-resume.
2. Write a short explanation, in your own words, of what happens (in terms of cost and performance) if a virtual warehouse is sized too small for a workload, and what happens if it's sized too large.
3. Compare this mentally to how DuckDB has behaved all course — a single process using whatever resources the laptop has, with no independent "scale up for this one heavy query" option. Write one sentence capturing the practical difference.
4. Note one real scenario from your own work (Dataskools, SB Finance Research, or this course) where independently-scalable compute would have actually mattered, if any comes to mind.

---

## Block 5: Written Comparison — Snowflake vs. dbt vs. DuckDB (60 min)

**Goal:** Produce a clear, correct written note pinning down how these three tools relate — this week's closing deliverable.

### Do this: 5 tasks

1. Write one paragraph explaining what dbt is responsible for, referencing this course's own dbt project from Week 3 as the concrete example.
2. Write one paragraph explaining what DuckDB has been responsible for all course, and how that role differs from dbt's.
3. Write one paragraph explaining what Snowflake would be responsible for if it replaced DuckDB as this course's engine — what would change, and specifically what would *not* change about the dbt project itself.
4. Write a short closing statement: in one or two sentences, state plainly whether Snowflake and dbt compete with each other or work together, and why.
5. Save this note alongside the week's other deliverables.

---

## Block 6: Reflection (30 min)

Answer in a few sentences each:

1. How did the presentation go against what Thursday's rehearsal predicted — what landed as expected, and what came up differently?
2. Before today, what did you assume "cloud data warehouse" meant, and how did that assumption hold up against what the reading actually said?
3. Of the three tools compared in Block 5, which one's role was clearest to you before today, and which one took the most rereading to actually pin down?
4. Looking back at the full week — Monday's schema exploration through today — what's the one skill from this week you'd most want to practice again on a different dataset?

**Deliverable:** A completed presentation, and a short written comparison note explaining, in your own words, how dbt, DuckDB, and Snowflake each fit together — this week's final deliverable, alongside the `.pbix`, screenshot, and reflection already submitted Thursday.
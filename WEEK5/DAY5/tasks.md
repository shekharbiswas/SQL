# Day 5 — Task Checklist (Presentation Day)

**No new building today. Presentation at 2 PM.**

Goal: rehearsed, polished walkthrough of the completed project, plus final repo/docs polish.

| Step | Task | Time |
|------|------|------|
| 1 | Fresh clean run-through | 30 min |
| 2 | Re-generate and screenshot docs/DAG | 20 min |
| 3 | Finalize README | 30 min |
| 4 | Write presentation outline | 30 min |
| 5 | Rehearse DAG walkthrough out loud | 30 min |
| 6 | Prepare answers to likely questions | 30 min |
| 7 | Full dry run (timed) | 20 min |
| 8 | Buffer / final fixes | 20 min |
| — | **Presentations begin** | **2:00 PM** |

*(Morning session: ~3.5 hours, ending well before 2 PM to leave room for buffer and lunch.)*

---

## Step 1: Fresh clean run-through — *30 min*

- [ ] Run `dbt build` one final time on your existing project — confirm everything is still green
- [ ] Fix anything that broke overnight (rare, but check credentials/warehouse are still active)

## Step 2: Re-generate and screenshot docs/DAG — *20 min*

```bash
dbt docs generate
dbt docs serve
```
- [ ] Take a clean, full screenshot of the lineage graph
- [ ] Take a screenshot of one model's documentation page (showing column descriptions)
- [ ] Save both into your repo (e.g. `assets/` folder) for use in the presentation

## Step 3: Finalize README — *30 min*

- [ ] Re-read your project README from Day 4
- [ ] Confirm the DAG screenshot is embedded and displays correctly
- [ ] Confirm the "how to run it" instructions are accurate (test them if time allows)
- [ ] Fix any typos or unclear sections

## Step 4: Write presentation outline — *30 min*

- [ ] Draft a 5-10 minute outline following the story arc from `day5_concepts.md`:
  1. The problem/dataset
  2. The pipeline (DAG walkthrough)
  3. Data quality (tests)
  4. Highlight: snapshot + incremental model
  5. What's next
- [ ] Keep it to talking points, not a script — you should speak to your own DAG live, not read slides

## Step 5: Rehearse DAG walkthrough out loud — *30 min*

- [ ] Open your `dbt docs serve` site and practice narrating the lineage graph node by node, out loud, once through
- [ ] Time yourself — adjust if it's running long or too short

## Step 6: Prepare answers to likely questions — *30 min*

Write a one-line answer for each — these are the most common questions an evaluator asks:
- [ ] "Why is this model a view and that one a table?"
- [ ] "Why did you make this specific model incremental?"
- [ ] "What does your snapshot actually track, and how would you know if it's working?"
- [ ] "What would break if you removed this test?"
- [ ] "What would you add if you had another week?"

## Step 7: Full dry run (timed) — *20 min*

- [ ] Do one complete run-through of the presentation, timed, ideally to a peer or the mirror
- [ ] Note anything you stumbled on and fix the explanation, not just re-read it

## Step 8: Buffer / final fixes — *20 min*

- [ ] Fix any last typos, broken links, or screenshot issues
- [ ] Confirm your GitHub repo is public/shared correctly for the instructor to view
- [ ] Close unrelated tabs/apps, have `dbt docs serve` and your repo ready to pull up instantly at 2 PM

---

## 2:00 PM — Presentations begin

Each student presents their completed dbt + Snowflake + Jaffle Shop project: the DAG, the modeling choices, the tests, the snapshot, and the incremental model — live from their own docs site and repo.

**Reminder: the project itself is already done as of Day 4.** Today only makes the explanation of it as strong as the project itself.
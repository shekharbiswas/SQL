# Day 5 — Task Checklist (light day, presentation at 2 PM)

No new SQL to write. Everything below should take a morning, not a full day.

---

## 1. README (30–45 min)

- [ ] One-paragraph summary of the platform built this week
- [ ] Short section per day (Day 1–4), 2–3 sentences each — what it added, not a copy of the task
      checklist
- [ ] A "what broke" section — 3+ real examples from the week (schema drift, a bad UPDATE, a
      deliberately failed task, spillage found in a query profile, etc.) and how each was resolved

---

## 2. Architecture diagram (20–30 min)

- [ ] One diagram: sources → stage/load → raw tables → streams/tasks → summary tables, with
      TPC-H performance testing shown as a separate branch, not part of the pipeline
- [ ] Export as an image into `assets/screenshots/`

---

## 3. Pick your two or three best numbers (10 min)

- [ ] One clustering number (`average_depth` before/after)
- [ ] One caching number (cold vs. cached timing)
- [ ] Optionally one concurrency number (single-cluster vs. multi-cluster completion time)

These go on one slide or are simply said out loud — no need for a full slide deck if time's tight.

---

## 4. Dry run (15–20 min)

- [ ] Say the presentation out loud once, timed — aim for 5–8 minutes to leave room for questions
- [ ] Confirm Snowsight is open to the right worksheets/screens in advance so nothing is being
      found live during the presentation

---

## Presentation structure (5–10 min, suggested)

1. What we built, in one sentence (30 sec)
2. Architecture diagram walkthrough (1–2 min)
3. One or two things that broke and how they were fixed (2–3 min) — this is the strongest part,
   don't rush it
4. The two or three scale numbers (1–2 min)
5. What we'd do differently at real production scale (30 sec–1 min)
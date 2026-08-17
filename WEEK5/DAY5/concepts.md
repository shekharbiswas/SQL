# Day 5 — Concepts (Presentation Prep)

No new building today. The project was completed on Day 4. Today is about being able to **explain** what you built and why — which is its own skill, and the one that actually gets tested in interviews and on the job.

**Presentation at 2 PM.**

---

## 1. Why explaining a project matters as much as building it

Anyone can follow a checklist. What separates a real analytics/data engineer is being able to answer "why did you do it that way?" for every decision — why this materialization, why this test, why this layer. Today is about rehearsing those answers, not writing new code.

## 2. The story arc of a good project walkthrough

A strong 5-10 minute walkthrough follows a simple arc:

1. **The problem** — what data existed (Jaffle Shop raw tables) and what business questions it should answer (who are our customers, what are they ordering, how has pricing changed)
2. **The pipeline** — raw → staging → intermediate → marts, shown live on the DAG
3. **The quality layer** — tests and what they catch
4. **The interesting parts** — the snapshot (history tracking) and the incremental model (efficiency) are the two most "senior-sounding" concepts in the whole project — spend real time on these
5. **What you'd do next** — a sign of understanding, not just doing (e.g. "next I'd add more sources," "next I'd move this to CI")

## 3. Knowing your DAG cold

You should be able to point at any node in the lineage graph and say, in one sentence, what it does and why it's at that layer. This is the single most common thing an interviewer or evaluator will ask about a dbt project — "walk me through your DAG."

## 4. Explaining materializations without hesitation

Be ready to justify, out loud, in one sentence each:
- Why staging models are views
- Why marts are tables
- Why the intermediate model is ephemeral
- Why `fct_orders` specifically needed to be incremental (not just "because I was told to" — the real reason: avoiding full rebuilds of a growing fact table)

## 5. Explaining the snapshot without hesitation

Be ready to answer: "What happens if you run `dbt snapshot` and nothing in the source changed?" (Answer: nothing new is inserted — the existing valid row stays valid.) And: "What do `dbt_valid_from` and `dbt_valid_to` mean?" This shows you understand SCD Type 2, not just that you copy-pasted a snapshot block.

## 6. Explaining tests as a quality story, not a checklist

Instead of listing test names, frame it as: "Here's what could go wrong with this data, and here's the test that would catch it." E.g. "If a customer_id appeared twice in dim_customers, our BI tool would double-count revenue — that's why there's a `unique` test there."

## 7. What "good" looks like in the docs site

Your `dbt docs` site is itself a presentation asset. A well-documented project — clear model/column descriptions, a clean lineage graph — demonstrates the same care a real team would expect, without you saying a word about it.

---

**Presentation checkpoint (by 2 PM):** you can navigate your own DAG live, explain every materialization choice, explain the snapshot and incremental model in plain language, and connect every test back to a real data-quality risk it prevents.
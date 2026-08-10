# Day 5 Concepts: Presenting, Then a First Look at Snowflake

**Estimated reading time: 25 min**

Today has two unrelated halves. The morning is the payoff for the whole week — presenting the dashboard at 11am. The afternoon shifts entirely away from building: a first, deliberately hands-off look at Snowflake, read and watched rather than built, with the specific goal of understanding how it relates to something already familiar — dbt.

This file covers the afternoon's content. There's nothing to prepare for the presentation itself beyond what Thursday already built.

## Why Snowflake, and Why Just Watching Today

Every tool this course has used so far — DuckDB, dbt, Power BI's Import mode — has run on one machine, using resources that machine happened to have. Snowflake is the first time this course looks at what changes once a data platform is built, from the ground up, to run somewhere else entirely: a cloud environment where storage and compute aren't just conceptually separable, they're literally billed and scaled independently. That's a big enough idea that it's worth understanding clearly before ever touching it hands-on — which is exactly why today is a reading-and-watching day, not a lab.

## The Core Idea: Storage and Compute, Separated

In a traditional database — and in DuckDB, as used all course — the thing that stores the data and the thing that queries it are the same process, running on the same machine. Scale one, and you've scaled both, whether you needed to or not.

Snowflake splits these apart on purpose:

- **Storage** is a shared layer, holding data in Snowflake's own compressed, columnar format, independent of any particular query being run against it.
- **Compute** happens in **virtual warehouses** — clusters of compute resources that can be spun up, resized, or spun back down independently of storage and independently of each other. Two teams querying the same data can each run their own virtual warehouse, sized for their own workload, without competing for resources or paying for capacity they don't use.

This is the single idea worth taking away from today even if nothing else sticks: a Snowflake account isn't one server getting bigger or smaller, it's a shared pool of data with as many independently-scaled compute engines pointed at it as needed, billed separately from the data they're reading.

## Where dbt Fits Into This Picture

This is worth being precise about, because it's an easy thing to blur: **dbt and Snowflake are not competitors, and they don't do the same job.**

- **dbt is a transformation tool.** It doesn't store data and it doesn't execute queries itself — it generates SQL, manages the dependency graph between models via `ref()`, runs tests, and builds documentation. Every actual query dbt generates gets sent to *something else* to execute — and that something else is a database engine.
- **Snowflake is that something else** — a place where data is stored and where SQL actually runs. It's one of the most common engines dbt projects run against in production, the same role DuckDB has played for this entire course.

Put differently: dbt is the layer that organizes *what* SQL runs and *in what order*; Snowflake (or DuckDB) is the layer that actually stores the data and executes that SQL. A production dbt project pointed at Snowflake works exactly like this course's dbt project pointed at DuckDB — same `sources`, `staging`, `marts`, `ref()`, `dbt build` — the only thing that changes is the `profiles.yml` connection details and the engine underneath. The mental model from Week 3, Day 4–5 transfers directly; what's different is what's on the other end of that connection, and how much it can scale.

## A Few Concepts Worth Recognizing, Not Mastering

Today's goal is vocabulary and a correct mental model, not hands-on fluency — these will make more sense once (or if) there's ever a reason to actually build against Snowflake:

- **Virtual warehouse** — an independently-scaled compute cluster, sized in "T-shirt sizes" (X-Small through 6X-Large), that can auto-suspend when idle and auto-resume on the next query, so compute cost tracks actual usage rather than running constantly like a laptop-hosted DuckDB file would.
- **Time Travel** — the ability to query data as it existed at a past point in time, or recover accidentally dropped objects, something a single local file doesn't offer in the same way.
- **Micro-partitions** — how Snowflake automatically organizes stored data into small, columnar, compressed units behind the scenes, without requiring manual partitioning the way some older warehouses do.

## Suggested Viewing / Reading

- **Snowflake's own "Key Concepts & Architecture" documentation** — the clearest primary-source explanation of the storage/compute split: [docs.snowflake.com/en/user-guide/intro-key-concepts](https://docs.snowflake.com/en/user-guide/intro-key-concepts)
- **"Snowflake in 20 Minutes" tutorial** — worth reading through even without an account, to see what a real Snowflake session looks like end to end: [docs.snowflake.com/en/user-guide/tutorials/snowflake-in-20minutes](https://docs.snowflake.com/en/user-guide/tutorials/snowflake-in-20minutes)
- **Snowflake's tutorials index**, if there's time to look at more than one: [docs.snowflake.com/en/learn-tutorials](https://docs.snowflake.com/en/learn-tutorials)

## What's Next

Today's task file has no new SQL, no new Power BI, and no Snowflake account — just the presentation itself, then time to watch/read the material above and write a short comparison note pinning down, in your own words, how Snowflake, dbt, and DuckDB each fit together. That note is this week's actual closing deliverable.
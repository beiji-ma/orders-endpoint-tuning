# From Fetch Graphs to Fetch Language

## 1. From Patchwork to Pattern

We began this journey with scattered tools: annotations, batch sizes, two-step queries. Each one fixed a symptom. None addressed the structure.

Over time, we noticed a pattern. Every workaround we applied—every `@EntityGraph`, every `JOIN FETCH`, every pagination trick—was trying to express the same thing:

> "Please fetch this, but only when needed, and only in this shape."

The problem wasn’t our intent. The problem was **how we expressed it**. Our tools weren’t expressive enough.

We weren’t building queries. We were stitching patches.

---

## 2. What ORM Can’t Express

ORM frameworks give us models. Sometimes, they give us fetch hints. But what they **don’t** give us is a language.

They can’t express:

- Conditional associations ("only fetch if filter X is present")
- Projection vs. hydration decisions ("load partial fields vs. entity")
- Composable joins across dynamic inputs
- Priority of associations ("always eager for A, lazy for B unless nested")

And worst of all:

- They lack runtime observability
- They hard-code access paths at compile time

So we kept guessing. Tuning batch sizes. Flipping annotations. Rewriting queries. None of it addressed the real problem:

> **There’s no first-class concept of an access path.**

---

## 3. Fetch Path Is a Language

After enough pain, we stopped looking for configuration.

We started looking for a language.

A **fetch path** isn’t just a hint. It’s a declarative expression of data access intent:

- What do I want?
- What shape?
- At what cost?
- With what fallbacks?

This language must be:

- **Compositional** — so it can express partial graphs, joins, filters, pagination
- **Structural** — so it can be validated, observed, and optimized
- **Decoupled** — from both entity models and repository interfaces

And it must be **runtime-capable**. Because access paths depend on input, on users, on screens—not annotations.

---

## 4. What We Want to Express

Here’s what we needed to say—but couldn’t:

> "Give me all orders, paginated by ID. Include customer name and region only if the user is admin. Skip line items. Return summary fields only."

Or:

> "Fetch top-level objects for list view. If expanded, load full substructure with dependencies."

These aren’t exotic use cases. They’re everywhere—in admin panels, dashboards, mobile apps, APIs.

Yet ORM annotations can’t say any of it. Not without duplicating queries, fragmenting repository code, or falling back to raw SQL.

So we began designing a fetch expression DSL.

---

## 5. Looking Ahead

The next step in this journey is language.

Not a general-purpose language. A **query-shaping language**.

We’ll show how access paths become explicit, composable values. How runtime context drives structure. And how we replaced annotations with expressions.

Because real performance starts when **you stop configuring**, and start **designing** your access.

Next up: the design of the DSL itself.


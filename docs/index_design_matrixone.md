# Index as a First-Class Design Concern

## Inspired by MatrixOne’s B-Tree Philosophy

## 1. Why Indexing Deserves a Front-Row Seat

In most systems, indexing is an afterthought. Something you tweak once performance tanks. Something hidden behind annotations, migrations, or DBA scripts.

But what if indexes weren’t metadata? What if they were **part of the language of data access**?

We came to this realization the hard way: by tracing every performance problem to a missing or misaligned index. Not just in SQL—but across in-memory collections, JPA entities, REST responses.

So we asked: can we design **access paths** where indexing is not only visible—but explicit?

---

## 2. What MatrixOne Gets Right

MatrixOne treats indexes as structure—not optimization. Its core B-Tree engine is not a sidecar—it **is** the query layer. You query through it, not around it.

More specifically:

- Every table is organized around an OID (object ID), which acts like a primary access spine
- B-Tree layout is stable, predictable, and reflected in the plan
- Query plans are aware of index shape, key range, and layout cost
- Multi-index paths can be chosen by the runtime planner

This approach changes how we think about access:

- You're not querying "tables"—you're traversing structured paths
- Each access pattern reflects an intentional structure

---

## 3. Access Path ≠ Query Shape

In most ORMs and query builders, the query shape is the query. But that’s misleading.

Consider:

```sql
SELECT * FROM orders o JOIN customer c ON o.customer_id = c.id WHERE ...
```

You don’t know:

- If this uses a covering index
- If the join is index-based or nested loop
- If pagination affects index scan bounds

The query *looks* correct—but its access path is invisible.

What MatrixOne suggests is:

> Expose and design the access path—not just the query shape.

This means:

- Declaring when you want index-by-OID
- Hinting which layout matters (e.g. B-Tree range vs. hash lookup)
- Designing your schema to reflect actual traversal

---

## 4. Toward Index-Aware DSLs

Our DSL should not abstract away index concerns. It should embrace them.

This could mean:

- Allowing expressions like:

  ```
  fetch orders where created_at > X
    using index created_at_btree
  ```

- Or even:

  ```
  select fields(id, summary) from memory.orders
    index by oid
  ```

More importantly, it means the **language of access paths must know the cost model**:

- Index selection affects fetch shape
- Key ranges affect pagination and batch behavior
- Index presence affects runtime feasibility

And this isn’t limited to the database. It applies just as well to in-memory maps, cache trees, and remote lookups.

---

## 5. Closing the Loop: From Index to Intent

Index-aware design is about more than performance. It’s about intention.

If you know your access will always go through `user_id -> orders -> order_items`, then structure your schema—and your DSL—to reflect that path.

MatrixOne teaches us that index is not the engine’s job. It’s the model’s responsibility.

And if our DSL can express that structure—then performance isn’t magic. It’s design.


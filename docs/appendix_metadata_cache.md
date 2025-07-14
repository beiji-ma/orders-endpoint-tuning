# What Oracle Knows That ORM Forgot

## On Metadata, Library Caches, and the Death of TTL Thinking

## 1. The Cache We Forgot to Design

ORM developers cache data. Oracle caches *structure*.

That distinction explains years of pain.

In most systems, we forget that metadata — the shape, meaning, and dependencies of data — is expensive to recompute and rarely changes. This includes:

- Field definitions
- Join relationships
- Index layout
- Query plans
- Access path dependencies

In Oracle, these live in:

- **Dictionary Cache** — table, column, privilege, type metadata
- **Library Cache** — SQL text, parsed tree, execution plan, cursor

They’re not auxiliary. They are the engine.

---

## 2. TTL: A Symptom of Observability Failure

Why does every Redis guide teach you about TTL?

Because:

> The cache has no idea what it’s caching.

It doesn’t know:

- Which fields the data depends on
- Which user role affects the shape
- Which input parameters reshape the result
- Whether the result is partial or complete

So you do this:

```js
cache.set("user:123", userObject, ttl=300)
```

And pray that 5 minutes is the right amount of ignorance.

---

## 3. Oracle and InnoDB: Memory as Intent

Oracle doesn’t pray. It tracks.

- Each parsed SQL statement gets a cursor
- Cursor includes access path, dependencies, privileges, and schema version
- When schema changes, cache invalidation is triggered — *surgically*

InnoDB does the same for row layout, field offsets, clustered index roots.

They don’t expire structures. They invalidate by cause.

---

## 4. The ORM Developer’s Blind Spot

ORMs do not:

- Represent metadata as first-class citizens
- Track access path versions
- Expose dependency graphs
- Cache intent or execution structure

Every fetch is a recomputation. Every query plan is rebuilt. Every join path is rediscovered.

Caching becomes a battle against your own framework.

---

## 5. Toward Structural Cache Semantics

We need a better model:

- **Structure as cacheable** — Not just data, but access shape
- **Intent as versioned** — DSL expressions hashable, traceable
- **Dependencies as explicit** — Fields, filters, joins declared and tracked
- **Invalidation as targeted** — No more TTLs — just cause-based eviction

With DSLs, this becomes possible:

```dsl
fetch user(id) { name, roles { id, permissions } }
```

This can compile into:

- A structural hash (for caching)
- A dependency graph (for tracking changes)
- An access plan (for performance prediction)

---

## 6. TTL Should Be the Last Resort

Use TTL when:

- You have no structure
- You can’t track changes
- You don’t care about precision

But if you do care — about correctness, efficiency, structure — then TTL is a **fallback**, not a strategy.

The best caches don’t expire. They understand.


# Pagination That Works: Fetching in Two Acts

## 1. The Split That Saved Us

The warning signs had been there all along: cursor explosions, in-memory pagination, unpredictable performance. But it wasn’t until we zoomed out and looked at the whole data access picture that we saw the truth:

> We had conflated two concerns—pagination and hydration—into a single query.

This realization changed everything. Instead of trying to paginate over pre-hydrated entities, we flipped the process:

- First, **select only the IDs** that match the query, properly paginated.
- Then, **load the full entities** (and their associations) based on those IDs.

Two steps. Two clear responsibilities. Zero surprises.

---

## 2. Phase One: Paging the Primary Keys

The first step inverts the usual ORM approach. Instead of fetching full entities with joins, we ask the database:

```sql
SELECT o.id FROM orders o WHERE ... ORDER BY o.created_at DESC LIMIT 50 OFFSET 100
```

This query:

- Has no joins, so it’s stable and performant
- Returns only IDs, so the result is small
- Preserves pagination semantics, even if data is hierarchical

It brings us back to fundamentals: pagination is about navigating **a list of identifiers**, not hydrated objects.

The output: `List<Long> orderIds`

---

## 3. Phase Two: Hydrating the Payload

Now that we have a stable set of IDs, we can load the data in a controlled, batched way:

```java
SELECT o FROM Order o
LEFT JOIN FETCH o.customer
LEFT JOIN FETCH o.lineItems
WHERE o.id IN :ids
```

Here, we control exactly what to fetch. Even better, we can use JPA’s `@BatchSize` annotation (or similar config) to let Hibernate optimize how it groups the secondary queries.

This step can:

- Fetch only the entities in the page
- Fetch their associations (with clear intent)
- Avoid in-memory joins or broken pagination

The two queries together simulate what a single ORM query *wants* to be—but with predictability and structure.

---

## 4. When We Knew It Worked

After applying this two-phase strategy:

- Cursor count dropped to stable single-digit levels
- Memory usage became flat and predictable
- Latency variance decreased dramatically
- GC frequency reduced, with smoother JVM pressure

Zabbix graphs told the story better than words. The endpoint stabilized and remained performant—even under load—for over two years.

No more fetch join roulette. No more in-memory slicing. Just structure.

---

## 5. Beyond ORM

What we had discovered was more than just a workaround. It was a design pattern:

- **First, isolate the identity space** (the rows you care about)
- **Then, hydrate the structure you need** (on your terms)

Most ORMs can’t express this because they don’t model queries—they model objects.

But by stepping outside the ORM’s fetch abstraction, we regained:

- Control over data access paths
- Visibility into query behavior
- The ability to reason about performance

Two-step fetching wasn’t a JPA trick. It was **query composition by design**.

---

## 6. Looking Ahead

Two-step fetching gave us performance. But it also gave us clarity.

Now we could start asking deeper questions:

- What if access paths were defined declaratively?
- What if pagination, filters, and joins were **all** programmable?
- What if we could *describe* fetch intent—not just execute it?

In the next chapter, we’ll explore how these questions led us to design a **Fetch Path DSL**—a structural, composable way to express access semantics.

Because performance isn’t about tricks. It’s about structure. And structure starts with language.


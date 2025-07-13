# Declarative Fetch Plans, Declarative Headaches

## 1. The Promise of Declarative Fetching

When we first discovered `@EntityGraph` and `@Subgraph`, it felt like the answer we’d been waiting for. After all, JPA’s default behavior—lazy loading with surprise queries—wasn’t working.

Declarative fetch plans promised an elegant fix:

- Control over what to load and when
- Cleaner queries with fewer surprises
- A reusable, annotation-based way to hydrate graphs

It sounded ideal. And for small, flat entity graphs, it sometimes worked.

But then reality crept in.

---

## 2. Where It Breaks Down

Declarative fetch plans break down quickly when the real world intervenes:

- **They’re static** — declared via annotations, not logic
- **They’re non-composable** — can’t merge or extend easily
- **They’re invisible at runtime** — no observability
- **They fail silently** — fallback behavior is unpredictable
- **They’re brittle** — minor model changes break graph expectations

Worse still, they don’t solve the pagination problem. In fact, when `@EntityGraph` is combined with collection relationships, it often reproduces the same issue as a fetch join: row explosion and in-memory paging.

This is because declarative graphs are compiled into JPQL fetches. The structure may look clean, but the underlying SQL remains vulnerable.

> "`@EntityGraph` feels like configuration. But it behaves like hardcoding."

---

## 3. Example: When Subgraphs Can’t Save You

Imagine you have an `OrderEntity` that links to `CustomerEntity`, which in turn links to `RegionEntity`. You try this:

```java
@NamedEntityGraph(
  name = "Order.customer.region",
  attributeNodes = @NamedAttributeNode(value = "customer", subgraph = "customerWithRegion"),
  subgraphs = @NamedSubgraph(
    name = "customerWithRegion",
    attributeNodes = @NamedAttributeNode("region")
  )
)
```

But what happens if `CustomerEntity` also lazily links to `AccountEntity`? And only sometimes do you need `region`, not `account`?

There’s no conditional expression. No runtime toggle. You end up:

- Maintaining multiple overlapping graphs
- Writing brittle names for every combination
- Binding logic to annotations instead of actual needs

It becomes a full-time job just managing fetch graphs.

---

## 4. Attempts to Automate the Pain

To deal with this, some projects try to abstract fetch plan generation. There are libraries and internal tools that:

- Dynamically build entity graphs
- Compose graphs at runtime
- Read configuration or metadata to generate annotations

But this adds complexity without addressing the core flaw:

> The fetch plan lives outside the query plan.

You still don’t control access paths directly. You still can’t see what SQL will be emitted. You’re configuring fetch behavior without observability.

Declarative becomes delusional.

---

## 5. Why Declarative Isn’t Composable

Real-world APIs don’t follow static graphs. They’re dynamic:

- A user selects certain columns
- A query includes optional filters
- A page requires partial hydration

These are **runtime concerns**. They require **runtime fetch logic**.

But `@EntityGraph` lives in annotations. It’s evaluated at query build time. You can’t branch, merge, parameterize, or trace it.

You end up with:

- A tangle of annotations
- Hard-coded query assumptions
- A fetch plan that can’t express logic

The result? Either over-fetching, under-fetching, or unpredictable behavior.

---

## 6. Looking Ahead

Declarative fetch plans promised structure. But they delivered rigidity.

What we needed was not configuration—but **composition**. A way to express, combine, and reason about access paths at runtime.

In the next article, we’ll explore how we moved away from annotations entirely. And began designing access as a structural query expression.

Because in the end, a fetch plan shouldn’t be declared. It should be designed.


# When Code Smells Cry for Help

## 1. The Code Nobody Wanted to Touch

Before any structured tuning was attempted, one piece of code stood out—not because it was ingenious, but because it was universally feared and quietly resented. Buried deep in `CacheOperations.java` was a block of imperative traversal logic designed to force-load every association that might be needed later.

```java
// Simplified from CacheOperations.java
for (OrderEntity order : orders) {
  for (CustomerEntity c : order.getCustomerEntities()) {
    for (AccountEntity acc : c.getAccountEntities()) {
      for (RegionEntity region : acc.getRegionEntities()) {
        Hibernate.initialize(region);
      }
    }
  }
  for (Tag t : order.getTags()) {
    Hibernate.initialize(t);
  }
}
```

At first glance, it looks like brute-force initialization. But look closer, and it tells a deeper story:

- Nobody trusted the ORM to fetch what they needed.
- Nobody could explain where the query boundaries were.
- Everyone feared the cascading N+1 problem but lacked tools to prevent it.

This wasn't code written in confidence. It was code written in self-defense.

## 2. Why This Pattern Happens

This pattern didn't appear in a vacuum. It was a direct response to performance issues already being felt in production. A common reflex in such situations is to introduce caching—often Redis or an equivalent layer—to avoid repeat hits to the database.

But once caching is layered on top of an ORM with lazy-loading semantics, a new problem arises: **hydration fails when the entity is accessed outside the original transaction scope**. Without an active session, deferred fields remain inaccessible, triggering classic `LazyInitializationException` errors.

In other words:

> "You’ve cached an entity graph, but forgot that it only loads itself when you’re looking. And by then, the session’s gone."

Faced with this dilemma, developers often revert to imperative hydration.

> Note: While we cannot confirm the developer's exact motivation, the structure and timing of the code suggest it was a workaround for lazy-loading failures in cached contexts. It may not be elegant—but at least it works.

Manual hydration like this emerges in systems where the data access layer is under-specified, under-powered, or both.

- **Lazy loading sounds elegant** until it silently breaks at runtime.
- **Eager loading feels safe**, but causes memory bloat and query explosion.
- ``** feels promising**, but is rigid and annotation-bound.

Developers caught in the middle often fall back to imperative traversal because it's the only thing that works in all contexts—even if it's ugly.

> You don’t write this kind of code because you want to. You write it because nothing else works.

## 3. Hidden Costs of Manual Hydration

Manual traversal may feel deterministic, but it's a false sense of control. It introduces hidden costs that only surface under load:

### 3.1 Query Multiplication

Each nested call (e.g., `.getCustomerEntities()` → `.getAccountEntities()` → `.getRegionEntities()`) potentially triggers a new query. Multiply that by the number of orders per page, and the database load grows exponentially.

What looks like a single API call may explode into **hundreds of queries**, especially if Hibernate is configured to lazy-load by default.

### 3.2 Session Coupling

This traversal only works while Hibernate maintains an active session. Once the session is closed—or if the entity graph is cached and later deserialized—accessing these associations triggers runtime exceptions.

This means business logic becomes implicitly tied to ORM session semantics, a coupling that's hard to test and easy to break.

### 3.3 Non-observability

From the outside, there's no way to tell which queries are being fired or why. SQL logs may help, but there's no structural visibility into the hydration pattern.

As a result, developers are left blind to performance regressions until it's too late.

> "When every field might trigger a query, predictability dies."

### 3.4 Defensive Overreach

To avoid all the above, developers often over-hydrate: they load everything, just in case.

This leads to bloated memory use, wide query joins, and poor pagination performance—without even knowing if the downstream logic actually needed that data.

> Manual hydration doesn’t solve the problem. It just pushes the cost somewhere harder to see.

## 4. Declarative Alternatives? Not Quite.

Of course, someone eventually suggested trying `@EntityGraph`, or even `@Subgraph` to declaratively fetch the required structure.

It seemed promising—until it wasn’t.

These declarative options, while syntactically elegant, introduced their own set of problems:

- They are **static annotations**—not dynamically composable.
- They **fail silently** when misaligned with real usage patterns.
- They **don’t interact well with pagination** or conditional logic.

More importantly, they reinforced the illusion that fetch behavior can be configured without a structural access strategy.

So while they helped with some query planning on paper, in practice they only moved the guesswork from code to configuration.

> Declarative fetch plans are like traffic signs in a fog. You still need headlights—and a map.

We’ll revisit these mechanisms in detail in a later article. For now, it’s enough to say:

> They didn’t help here. And they couldn’t have.

## 5. This Wasn’t Just Bad Code

It’s tempting to look at `CacheOperations.java` and scoff. To see it as technical debt or legacy residue. But that would miss the real lesson.

This wasn’t just bad code. It was an **early warning**. It told us:

- That we lacked a structural way to describe access paths.
- That hydration was emergent, not planned.
- That business logic was reaching down into the persistence layer—because no one else was taking responsibility.

The real problem wasn’t that we wrote imperative loading code. The problem was that the system gave us no better alternative.

> "When systems fail structurally, the symptoms show up in code."

## 6. Looking Ahead

The imperative hydration pattern—ugly as it was—served its purpose. It bought us time. But it also bought us technical debt.

In the next chapter, we’ll explore how we stopped fighting the ORM—and started redesigning the access path itself.

Through a strategy we call **two-step fetching**, we introduced a new abstraction boundary between entity shape and data layout. It’s not a new feature. It’s a new way to think.

Let’s go there next.


## From Hotfixes to Structure: Order Endpoint Tuning as Architectural Inquiry

### 1. Introduction

This article revisits a seemingly routine performance tuning effort—optimizing the `/orders` endpoint under high load. But beyond Hibernate annotations or fetch strategy tweaks, it became an invitation to examine the underlying assumptions of how APIs access data, how fetch strategies are structured, and how architectural primitives (or their absence) shape performance outcomes.

### 2. Historical Attempts Before Refactoring

![Version Comparison of JDBC statements and RT(ms)](assets/jdbc-rt-comparison.png)

*Figure: Statement volume and response time before and after structured tuning.*

Before any formal tuning began, the system was already exhibiting signs of extreme stress — **at peak load, the database reported over 55,000 active cursors.**

This wasn’t a warning sign; it was a fire alarm. It meant the persistence layer had no guardrails, and the cost of a single API call was catastrophically high under concurrency.

Before any formal tuning began, the system was already exhibiting signs of extreme stress — **at peak load, the database reported over 55,000 active cursors.**

This wasn’t a warning sign; it was a fire alarm. It meant the persistence layer had no guardrails, and the cost of a single API call was catastrophically high under concurrency.

The issue long predated our formal intervention. A large, imperative code block in `CacheOperations.java` attempted to manually pre-load associated entities by deeply traversing and `Hibernate.initialize(...)` each nested relationship.

This code—clearly disliked even by its original author—reflected a desperate workaround in absence of structured access shaping mechanisms. It blurred boundaries between business logic, persistence logic, and data lifecycle assumptions.

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

This approach made three things obvious:

- Hibernate offered no extensible access path abstraction.
- Fetch behavior was baked into static annotations.
- The system lacked runtime-level observability or control of query shape.

### 3. Initial Structured Tuning with JPA

We began by formally replacing imperative eager-loading with declarative annotations.

#### Steps:

1. Add `@NamedEntityGraph` to `OrderEntity`
2. Apply it in the Spring Data repository
3. Introduce `@Transactional(readOnly = true)` at controller layer to preserve session
4. Use `@Subgraph` and `@BatchSize` to tune nested collections

#### Outcome:

- Initial improvement
- But revealed a deeper problem: `HHH000104`

### 4. The Breakage: HHH000104

Baeldung and Appsloveworld both note the consequence of using eager collection fetches with pagination:

> `HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!`

This meant Hibernate had to materialize *all* rows before slicing them in memory—a direct path to memory pressure and performance collapse.

### 5. The Turning Point: Two-Step Fetching

Instead of tuning annotations further, we redesigned the access path:

- Step 1: Query only IDs with pagination and filters
- Step 2: Fetch entity graph using `WHERE id IN (...)`
- Re-sort results post-hydration if needed

This removed the conflict between eager collection fetch and pagination.

### 6. Persistent Risk: 2,450 Cursors

Even after optimization, the database still reported **2,450 active cursors** under peak load. This is not sustainable.

This raised critical questions:

- Why so many cursors?
- Are ORM fetch semantics leaking into resource control?
- Can we tune Hibernate enough—or must we step outside its model?

### 7. Systemic Reflection: ORM is Not Enough

We concluded that tuning was merely alleviating symptoms. The real problem is architectural:

- No extensible access path planning mechanism
- No runtime observability of fetch strategies
- No dynamic shaping of hydration behavior based on query context

Most ORM frameworks, Hibernate included, simply don’t support:

- Runtime data shape negotiation
- Intent-driven query structuring
- Separation of identity resolution from data hydration

### 8. Looking Forward

The deployment of the two-step access path marked a dramatic turning point. For nearly two years, Zabbix metrics confirmed exceptional system stability: minimal slow queries, steady resource usage, and—perhaps most importantly—no more of those cryptic “database is gone” or connection timeout events. It was a period of quiet reliability.

This wasn’t just luck. The prediction—that the solution would hold for two years—was grounded in a structural insight: **as long as data volume grew at a linear rate, cursor pressure would remain bounded**. And for a time, it did.

But time doesn’t stand still. Roughly two and a half years later, symptoms began to resurface. Most of the outages observed in the past six months stemmed from infrastructure—not application—failures: intermittent network disruptions (local and regional), NFS dismounts on virtual machines, and other low-level instabilities. These are well-documented and explainable.

Still, the reappearance of database stress—however externally triggered—reminded us that the underlying architecture had never fully evolved. The tuning had worked, but only within a bounded regime. **Once that regime shifted, cracks reappeared.**

And that brings us to the real question:\
**Should developers even be trapped in this level of cursor math, session awareness, and hydration complexity in the first place?**

The fact that such deep tuning was needed at all is not a failure of execution—it is a symptom of architectural limitation.

This article doesn’t close the loop; it opens a new one. One that asks:\
**How do we design access layers that are structured, observable, adaptable—and fast, without manual babysitting?**

Future articles in this series will explore that question—with clarity, curiosity, and structural honesty.

### Appendix: Conventional JPA Techniques (Not Endorsed, But Documented)

These may help in some cases, but don’t address the architectural root:

- `@BatchSize`
- `JOIN FETCH` with limit caveats
- Native queries with projections
- DTO-based hydrators
- Projection interfaces in Spring Data

> Our position: these techniques are symptomatic, not structural. We document them, but advocate for architectural rewiring instead.

### References

- Baeldung: [NamedEntityGraph](https://www.baeldung.com/spring-data-jpa-named-entity-graphs)
- Appsloveworld: [Pagination with Lazy Load](https://www.appsloveworld.com/springboot/100/123/using-spring-data-jpa-entitygraph-with-lazy-load-mode-for-namedattributenode-fiel)
- Internal: p6spy logs, profiling dashboards
- MatrixOne (legacy system) – OID-first access model (not MatrixOrigin)


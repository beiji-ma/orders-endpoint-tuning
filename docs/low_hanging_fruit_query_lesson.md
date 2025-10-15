# When Low-Hanging Fruit Turns Rotten — Lessons from a Million-DB-Hit Query

## Background (what we inherited)

- The core query **filters in memory** after fetching large datasets.
- Query expressiveness is **extremely poor**: a half-built Boolean filter (no operator precedence, no parentheses/nesting, limited operators).
- **No predicate pushdown**: nothing (including the permission model) is enforced at the storage/index layer.
- Result: frequent logic mistakes in filtering, wrong results, sporadic **OOMs**, DB pressure spikes, and a steady stream of “fix one bug, wake another” hotfixes.
- A “cache layer” was added, but it only **hides symptoms** and amplifies inconsistency when underlying data changes.

---

## Diagram: Old vs New Architecture

```mermaid
graph LR
    subgraph Old_Architecture [Legacy Query Flow]
        UI[UI Query Request] --> API[App Logic]
        API -->|fetch all| DB[(Database)]
        API -->|filter in memory| Mem[In-Memory Filter]
        Mem -->|apply cache| Cache[Cache Layer]
        Cache --> Result[Result Returned]
    end

    subgraph New_Architecture [Proposed Predicate Pushdown Flow]
        UI2[UI Query Request] --> API2[Query Parser / DSL]
        API2 --> Plan[AST → Query Plan]
        Plan -->|pushdown| DB2[(Database / Indexed Views)]
        DB2 -->|filtered result| Policy[Policy-aware Cache]
        Policy --> Result2[Result Returned]
    end

    classDef old fill=#f4cccc,stroke=#d43f3f;
    classDef new fill=#d9ead3,stroke=#4a7f33;
    class Old_Architecture old;
    class New_Architecture new;
```

**Comparison**

| Aspect                 | Old Design                        | New Design                                 |
| ---------------------- | --------------------------------- | ------------------------------------------ |
| Filter Execution       | In memory after fetching all data | Predicate pushdown to DB layer             |
| Boolean Logic          | Limited (no nesting/precedence)   | Full AST-based logical model               |
| Permission Enforcement | App layer (post-query)            | Embedded in query plan (policy predicates) |
| Performance            | 1M+ DB hits, frequent OOMs        | Selective queries, reduced I/O & memory    |
| Maintainability        | Patch-driven, fragile             | Model-driven, testable, reusable           |
| Cache Strategy         | Blind object cache                | Policy-aware + TTL-based cache             |

---

## Why “low-hanging fruit” failed here

- **Local fixes vs. global debt:** Each patch focused on one symptom (a path, a field, a branch) while the real cause—*unbounded in-memory filtering without a real query model*—stayed intact.
- **I/O amplification:** Fetch-then-filter multiplies DB hits and network bytes; caching multiplies stale data windows.
- **Unverifiable semantics:** Without precedence/nesting, business logic gets “re-encoded” in code branches—hard to reason about, harder to test.
- **Security as an afterthought:** Permissions not pushed down force late filtering, risking leaks and wasting work.

## The architecture we actually need

1. **Policy-aware predicate pushdown**
   - Express permissions and row-level filters as **first-class predicates** that compile into storage-level constraints (views, RLS, WHERE clauses, indexable conditions).
2. **A real query model (mini-DSL or structured API)**
   - Support `AND/OR/NOT`, parentheses, precedence, comparison operators, IN/ANY, ranges, null-safe ops.
   - Validate and normalize to an internal AST → **query plan** (no ad-hoc if-else forests).
3. **Plan stages & execution**
   - **Projection & pruning** (only needed columns).
   - **Early selectivity**: push selective predicates first.
   - **Join/order/pagination** with cost-aware limits; never materialize “the world” to RAM.
4. **Right-layer caches**
   - Cache **post-policy** results with short TTL + key derivation from normalized predicates.
   - Prefer **index-friendly** caches (e.g., materialized views) over opaque app-layer blobs.
5. **Guardrails & SLOs**
   - Query cost caps (row/byte/time), circuit breakers, backpressure; structured metrics (hits, rows scanned, plan type).
   - Golden tests for correctness (truth tables for complex boolean expressions).

## Migration plan (no big bang)

- **Phase 0 — Freeze + Observe:** Freeze new “quick fixes”. Add telemetry for query shapes, result sizes, memory, and DB hits.
- **Phase 1 — Introduce the model:** Define the query schema (JSON/DSL) with full boolean semantics + parentheses. Keep old path; **shadow-execute** new path on a small cohort, compare results.
- **Phase 2 — Pushdown mapping:** Map each predicate to storage-level filters (views/indexes/RLS). Add projections and limits; prove DB hits ↓ and RAM usage ↓.
- **Phase 3 — Policy integration:** Encode permissions as predicates in the plan; remove duplicate app-side filters.
- **Phase 4 — Decommission:** Turn off legacy branches; delete cache hacks tied to pre-pushdown flow.

## What “good” looks like (measurable)

- **DB hits:** ↓ by an order of magnitude for top 20 query shapes.
- **RAM spikes / OOMs:** 0 in rolling 30 days.
- **Correctness:** 100% match to golden truth tables for complex boolean cases.
- **Latency:** P50/P95 within SLO; tail stabilized under load.
- **Hotfix rate:** ↓ significantly (bugs move from “logic mismatch” to normal edge cases).

---

## Lessons Learned / Cultural Reflection

### 1. Organizational short-termism

Teams often confuse *speed* with *progress*. Under pressure to deliver visible results, they favor “quick fixes” that create the illusion of progress while silently increasing systemic debt. Leadership tends to reward immediate closure of tickets, not the invisible effort of refactoring or system redesign.

### 2. The false comfort of “working code”

Once a system “runs,” there’s a strong bias against revisiting core logic, even if the cost of running it is skyrocketing. Teams mistake stability for health — but technical debt behaves like interest, not principal; it compounds invisibly until it consumes all agility.

### 3. Fear of complexity

A design that introduces formal models (like predicate pushdown or a DSL) is often labeled as “too complex,” even when it actually *reduces* total complexity by organizing chaos. The aversion to such change stems from a lack of shared understanding of architectural value and a fear of owning long-term consequences.

### 4. Skill asymmetry and cognitive bias

In teams with uneven technical depth, decision-making naturally drifts toward the lowest common denominator. Simpler, ad-hoc fixes feel more accessible; deep redesigns require a shared conceptual language that’s often missing. Over time, this produces a culture of patching rather than planning.

### 5. Leadership’s role in shifting incentives

Architectural renewal doesn’t happen spontaneously — it requires leaders to *protect long-term work from short-term noise*. Changing the reward structure (celebrating systemic improvements, not just ticket throughput) is the only way to escape the low-hanging-fruit trap.

### 6. Conclusion

The million-hit query is not just a performance bug — it’s a mirror reflecting how teams perceive effort, ownership, and time. True progress begins when an organization realizes that **maintainable systems grow from disciplined choices, not quick wins**.

---

## Talking point for leadership

This is not “making it more complicated.” It’s **organizing existing complexity** so the system can scale with fewer surprises:

- We stop paying the tax of fetch-then-filter.
- We move **security and selectivity** to where they belong: the data layer.
- We trade endless micro-patches for a **maintainable plan** with clear semantics, tests, and guardrails.


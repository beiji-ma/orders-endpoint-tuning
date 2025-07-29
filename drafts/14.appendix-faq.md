# Appendix: FAQ and Misconception Debunking

> **Imagine this:**  
> Dassault one day open-sources the engine of ENOVIA / 3DEXPERIENCE.  
> You wouldn’t be reading this series.  
> But until that day comes, I’m left with one choice:  
> **Build it myself.**

> *(This note also appears in the series README as an epigraph — the spirit behind this journey.)*

## 1. “Aren’t you being too radical?”

We understand this reaction. Saying “JPA and Redis are broken” sounds harsh.

But this series is not about dismissing technologies — it’s about exposing their structural blind spots.

If you’ve ever:
- Been confused by a cascade of lazy-loading queries
- Struggled with a cache that’s either stale or missing
- Had to optimize query after query with band-aids…

Then you’ve already felt the pain we’re addressing.
We’re not radical. We’re structural.

---

## 2. “JPA can be tuned. Why throw it away?”

Yes, JPA can be tuned — but often only within its assumptions:
- Static entity graphs
- Compile-time relationships
- No runtime structure observability

We’ve tried those paths. Some work, some don’t. But all require heroic effort to achieve what structure-first design gives you by default.

We’re not throwing JPA away. We’re just refusing to let it define the limits of system design.

---

## 3. “But everyone uses Redis. Are they wrong?”

Not wrong — just incomplete.

Redis is great for delivery latency.
But when it becomes a substitute for design — when it hides structural debt instead of resolving it — then it becomes a liability.

Most Redis setups lack:
- Structural awareness
- Dependency tracking
- Intent-based invalidation

So it works… until it doesn’t. We advocate Redis *after* structure, not instead of it.

---

## 4. “Isn’t this overengineering?”

Structure isn’t overhead. It’s prevention.

- You only think you’re going fast when skipping it
- You pay later — in complexity, in bugs, in degraded performance

Real structure:
- Clarifies what’s being fetched
- Predicts access patterns
- Enables observability
- Reduces test effort

Overengineering is when you build for scale you’ll never need. We’re building for clarity you’ll need every day.

---

## 5. “Will this work for small teams?”

Absolutely. In fact, structure helps *especially* small teams:

- Less tribal knowledge
- Fewer performance fire drills
- More predictable scaling
- Easier onboarding

It’s not about headcount. It’s about shared intent.
DSL + structure = team coherence.

---

## 6. “So... you’ve built your own ORM?”

Not quite.

ORMS are about **mapping**. We’re about **expressing**.

An ORM maps classes to tables.
A DSL expresses access paths, semantics, and structure — across memory, database, and services.

We’re not reinventing ORMs.
We’re proposing a shift in how we **declare, observe, and evolve** data access.

That shift is architectural.
And long overdue.

---

📚 *Further Reflections:*  
- `afterword.md — Why Metaphors Stay When Code Fades`  
- `appendix-koans.md — Rootless Roots: Architectural Koans of MMSA (中文)`  
- `appendix-koans-en.md — Rootless Roots (English)`  
- `appendix-koans-sv.md — Rotlösa Rötter (på Svenska)`


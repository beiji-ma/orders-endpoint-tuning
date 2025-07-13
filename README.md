# Orders Endpoint Tuning

A real-world performance tuning story that became an architectural inquiry.

## 📖 Introduction

This project begins with a high-volume `/orders` endpoint that exposed deep cracks in how traditional ORM tools—especially JPA—deal with data access under scale.

Through one tuning effort, we rediscovered the power of **access path design**, **two-phase hydration**, and **systemic observability**.

## 📚 Series Roadmap

This repository is structured around a set of focused articles:

1. `intro.md` — *From Hotfixes to Structure*  
   Opening story and systemic framing.

2. `cacheoperations-analysis.md` — *When Code Smells Cry for Help*

3. `jpa-hhh000104.md` — *Paging vs. Fetching: Why JPA Crumbles*

4. `entitygraph-vs-subgraph.md` — *Declarative Fetch Plans, Declarative Headaches*

5. `two-step-fetching.md` — *Rebuilding the Access Path*

6. `access-path-critique.md` — *ORM Can’t Save You*

7. `matrixone-oid-api.md` — *The OID-First Philosophy*

8. *(Optional)* `observability-design.md` — *Designing for Query Shape Visibility*

9. *(Optional)* `jpa-vs-systemic-tuning.md` — *Why We Don’t Recommend JPA Fixes*

> We don't tune harder. We tune structurally.

## 🖼️ Assets

Place illustrative diagrams in the `assets/` folder, e.g.:

- `assets/jdbc-rt-comparison.png`

## 📎 References

- [Baeldung: NamedEntityGraph](https://www.baeldung.com/spring-data-jpa-named-entity-graphs)
- [Appsloveworld: Pagination with Lazy Load](https://www.appsloveworld.com/springboot/100/123/using-spring-data-jpa-entitygraph-with-lazy-load-mode-for-namedattributenode-fiel)
- Internal: p6spy logs, profiling dashboards
- MatrixOne (legacy system) – OID-first access model (not MatrixOrigin)


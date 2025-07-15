# 📘 Orders Endpoint Tuning Series · MMSA 架构文集

> **Imagine this:**  
> Dassault one day open-sources the engine of ENOVIA / 3DEXPERIENCE.  
> You wouldn’t be reading this series.  
> But until that day comes, I’m left with one choice:  
> **Build it myself.**

This series documents a journey from firefighting performance issues in a production system, to uncovering the deeper structural limitations of modern application development — and proposing a way forward.

It begins with a real-world tuning case and expands into a critique of JPA, ORM-based access paths, and the misuse of caching systems like Redis. But we don’t stop there. We offer an alternative: **a language-first, structure-oriented approach** to query, cache, and system design.

---

## 🧭 Roadmap

*Note: All chapter files follow numeric naming convention, starting from `01.intro.md`.*

### 📂 Core Chapters

| No. | File | Title | Description |
|-----|------|-------|-------------|
| 01 | [01.intro.md](docs/01.intro.md) | *From Hotfixes to Structure* | Opening story and systemic framing |
| 02 | [02.cacheoperations-analysis.md](docs/02.cacheoperations-analysis.md) | *When Code Smells Cry for Help* | The first sign of trouble: ugly caching logic and desperation |
| 03 | [03.jpa-hhh000104.md](docs/03.jpa-hhh000104.md) | *Paging vs. Fetching: Why JPA Crumbles* | The real cost of mixing fetch joins with pagination |
| 04 | [04.entitygraph-vs-subgraph.md](docs/04.entitygraph-vs-subgraph.md) | *Declarative Fetch Plans, Declarative Headaches* | Why EntityGraph looks declarative but breaks down in practice |
| 05 | [05.two-step-fetching.md](docs/05.two-step-fetching.md) | *Rebuilding the Access Path* | Manual fetch logic vs structural control |
| 06 | [06.access-path-critique.md](docs/06.access-path-critique.md) | *ORM Can’t Save You* | Static plans, runtime blindness, and why mapping isn’t access |
| 07 | [07.index-design-matrixone.md](docs/07.index-design-matrixone.md) | *Indexing as Language* | Designing index structures as native access logic |
| 08 | [08.model-unification.md](docs/08.model-unification.md) | *One Model, Many Runtimes* | From DB to JVM to memory — unifying structure everywhere |
| 09 | [09.fetch-dsl-intro.md](docs/09.fetch-dsl-intro.md) | *From Fetch Graphs to Fetch Language* | DSLs for expressing structure, access intent, and observability |

### 🧪 Optional / Extended Chapters

| No. | File | Title | Description |
|-----|------|-------|-------------|
| 10 | [10.observability-design.md](docs/10.observability-design.md) | *Designing for Query Shape Visibility* | Structure-aware logging and runtime introspection |
| 11 | [11.jpa-vs-redis.md](docs/11.jpa-vs-redis.md) | *Caching Is Not a Query Strategy* | Redis is fast. But what are you caching, really? |
| 12 | [12.appendix-metadata-cache.md](docs/12.appendix-metadata-cache.md) | *What Oracle Knows That ORM Forgot* | Library cache, dictionary cache, and the power of structure-first memory |
| 13 | [13.the-shift.md](docs/13.the-shift.md) | *A Developer’s Journey from Fixes to Structures* | From random patches to system clarity: a personal transition |
| 14 | [14.appendix-faq.md](docs/14.appendix-faq.md) | *FAQ and Misconception Debunking* | Addressing common doubts, misunderstandings, and critiques |
| 15 | [15.appendix-koans.md](docs/15.appendix-koans.md) / `-en.md` / `-sv.md` | *Rootless Roots* | Metaphorical reflections on architectural thinking |
| 16 | [16.afterword.md](docs/16.afterword.md) | *Why Metaphors Stay When Code Fades* | Final words on enduring design language |

---

## 📝 License

© 2025 Beiji Ma · This series is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

You are free to share, adapt, and build upon the content for any purpose — even commercially — as long as you give proper credit and license your modifications under the same terms.

> Because structure deserves to be shared, too.


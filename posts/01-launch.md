# Post 1 — Launch

**Principles:** 1, 2, 8, 11 · **Graphic:**
[`assets/principles-overview.png`](../assets/principles-overview.png) ·
**Status:** published 2026-08-12

> Text as finalized before publication; the posted version may differ in
> small wording. The `@TechVera` mention has to be typed into the LinkedIn
> composer and picked from the dropdown — pasted text does not become a tag.

---

For the last weeks I've been running @TechVera — the European LLM platform
I co-founded — as a software company where no human reads the code. Not as
a stunt — as an operating model.

Agents write every line. What keeps it governable is not trust in the
models, but a set of operating principles: humans own exactly three
artifacts (the behavioural spec, the requirements, the rules of the
verification harness). Everything else — architecture, code, tests — is
agent territory, governed by gates, twin-environment verification and a
single typed register instead of code review.

I wrote the whole model down: who owns which decision, what counts as
evidence when nobody reads diffs, how changes are verified against two
freshly built environments, and what to measure so the machinery itself
stays understandable.

It runs in production. The numbers in the paper come from that
installation, including the uncomfortable ones.

Full paper (free, CC-BY): https://github.com/lubo92/agentic-engineering-principles

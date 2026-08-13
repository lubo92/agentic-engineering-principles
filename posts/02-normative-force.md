# Post 2 — Where the target state lives

**Principle:** 2 (code has no normative status) · **Graphic:**
[`assets/normative-force.png`](../assets/normative-force.png) ·
**Status:** ready

---

In traditional software development — the kind run by humans — there is
often a certain contradiction about what really defines what a solution is
supposed to do. In theory, and at the beginning of an implementation,
there are user journeys, requirements, acceptance criteria. But as soon as
a solution is alive and gets developed iteratively, the code itself
usually takes that role over: what counts is exactly what the code says.
Georg Jellinek's normative force of the factual, carried over from state
theory into the world of software development.

If the code is neither written by humans nor reviewed by humans, then it
may no longer define the target state of a solution. Normative force then
needs to rest with user journeys and requirements alone, translated into
acceptance criteria. Inside a corset of journeys and requirements,
development agents rebuild the code, and an automated verification layer
checks whether all acceptance criteria are met. Code becomes a disposable
artifact — not because quality stopped mattering, but because it only
becomes replaceable once everything worth keeping is written down
somewhere else.

Building a new feature means adding a new user journey, or extending an
existing one. Eliminating unwanted behaviour means adding an acceptance
criterion that rules that behaviour out.

On my GitHub account you'll find my document, in which I lay out a
comprehensive set of principles for how complete hands-off engineering can
work: https://github.com/lubo92/agentic-engineering-principles

---

## Notes

- The graphic is built to stand alone: its kicker states the setting
  (agents write every line, no human reads the code) before the headline
  poses the question, so a reader who never saw post 1 still lands.
- Deliberately left out, to keep one idea per post: the inversion of
  review (agents review the human's spec changes), and the split between
  deterministic tests and AI reviewers inside the verification layer —
  the latter gets its own post, and naming it here would contradict it.
- The sentence defusing "disposable artifact" is the one place where the
  text says more than the graphic. If the post is trimmed further, the
  answer to the inevitable "so quality doesn't matter?" belongs in the
  comments instead.

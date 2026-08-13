# Seventeen principles for software organizations where no human reads the code

## A governance model for agentic engineering

*Wolfgang Lubowski · TechVera ([techvera.ai](https://techvera.ai)) · Version 1.1, August 2026*

Software engineering is entering a new operating model: agents increasingly write the implementation, while humans define goals and constraints. Most discussions focus on better models, better prompts, better tools, or better harnesses. Much less attention has been paid to a more fundamental question:

**What are the operating principles of a software organization in which humans no longer review code?**

This document answers that question with **seventeen concrete principles**. They define who owns decisions, which artifacts are normative, how authority is distributed, what constitutes evidence, how changes are verified, and how an agentic software organization remains governable as it scales.

The principles are not design aspirations or best practices. Together they form a coherent operating model that can be implemented directly: each principle introduces a concrete rule, and Part II translates the principles into repositories, permissions, processes, gates, and workflows.

## Terms

Two terms carry the whole document, and only they are fixed here.

**Inner harness** — Claude Code, OpenAI Codex, Pi, Cursor, Gemini CLI, Aider,
OpenHands and their peers, commercial and open source alike: the software
around an LLM that lets it act at all. It runs the agent loop, calls the
tools, manages the context. A team adopts one rather than building it. It
is not the subject of this document, and everything here is meant to work
on top of any of them.

**Outer harness** — everything a team assembles on top of an inner harness
to govern the work: the rules, the gates, the verification runs, the
records and the procedures. The inner harness decides how an agent acts;
the outer harness decides what counts as done. Almost every principle
below constrains its shape, which is why the two are worth telling apart
from the start.

---

## Part I — The seventeen principles

### Authority — who writes what, and who judges

Four principles form the constitution of the model: what humans own,
how power is layered and protected, and why nobody — human or agent —
reviews their own work.

#### 1. Humans write the user journeys, the requirements, and the rules of the outer harness; agents make every technical decision.

Humans own exactly three classes of normative artifacts. The **journeys** describe the observable
target behavior of the platform from the perspective of users, operators,
failure cases, data, and security — with numbered, individually testable
acceptance criteria. The **requirements** state functional and
non-functional qualities and boundaries: data residency, security posture,
latency, availability and cost budgets — each with a named means of
enforcement. The **harness rules** define how the agentic work itself is
conducted: the gates, the judges, the procedures. Everything below that —
architecture, technology choices, structure, code — belongs to the agentic
layer. No human reads diffs; correctness is established by automated gates,
not by review. Where an implementation reveals that a human decision is
missing, that gap is escalated as a decision request, never
decided silently in code. The human acts that remain in operation —
answering decision requests, ruling on findings, triggering promotion —
are not exceptions to this boundary but exercises of it: each one applies
or amends a norm. The line does not separate humans from activity; it
separates deciding what shall count from deciding how to build.

#### 2. Authority is layered: harness rules above journeys and requirements, above tests and ADRs, above code — and each layer is protected in proportion to its power.

The **harness rules** (gates, judge rubrics, procedures, severity classes)
are changeable only by humans, and agents technically cannot write to them:
they live outside the product repo, the verification machinery lives in its
own version-pinned repo, and agents work under an identity without
administrative rights. The **journeys and requirements** live inside the
product repo, but in a server-side write-protected area. The **tests and
ADRs** are freely evolvable by agents, but only through a regulated
procedure. The **code** sits at the bottom and has no normative status:
existing behavior is never its own specification. The decisive property of
this architecture: the system under test cannot modify its own verification —
enforced by permissions, not by convention.

#### 3. Humans write the journeys and requirements and agents review them; agents write the code and deterministic tests review it.

Classical review does not die — it inverts. Every human change to journeys
or requirements passes a standardized agentic **consistency review**:
contradictions with the existing body, gaps in testability, affected
projections. It is deliberately non-blocking — humans keep the last word
over what they wrote — but it **requires acknowledgment**: the
report is worked through or explicitly accepted, and the acknowledgment is
recorded in the register. The mirror image holds for the artifacts agents
own, with a deliberate asymmetry. Tightening is free: new tests, stricter
assertions and new constraining ADRs merge with the change that brings
them — they can only strengthen the oracle, and a wrongly added test
surfaces on its own by blocking work. Weakening is a guarded act: removing
or relaxing a test, or superseding an ADR, is ratified by a separate judge
in a fresh session, examined against the journeys, the requirements and
the harness rules — never against the code, and never by the session that
proposed it. ADRs stay append-only: a changed decision is a new,
superseding record.
Each side reviews the other, and nobody reviews themselves.

#### 4. Separation of powers: The agent session that implements a change never judges it; implementer and verifier are always separate agent sessions.

If an analogy helps, it is the separation of powers in constitutional
government: humans are the legislature — they write the journeys, the
requirements, and the harness rules; agent sessions are
the executive — they implement; and verification is the judiciary —
deterministic tests and AI reviewers together. This
principle enforces the boundary the whole model stands on: the executive
never sits in judgment of its own case.

Concretely: the verifier works in a fresh context and sees only the
journeys and requirements and the running system — never the code, never the
implementer's session. The implementing identity has no write access to
judge rubrics, register dispositions, or gates. This separation is not
ceremony; it is the precondition for machine judgment meaning anything at
all: agents that evaluate their own output systematically overestimate it.

### Verification — two kinds of checks

What checks exist, what each of them is allowed to do, and what counts as
evidence.

#### 5. Only deterministic checks can block a merge; an AI reviewer's finding never blocks by verdict — it forces a recorded decision: fix, schedule, accept, or dismiss.

A gate needs answers that are the same on every run. Deterministic tests
give exactly that. AI reviewers do not: their verdicts can flip between runs
on identical evidence, and their agreement with human judgment falls well
short of what a blocking gate demands. So an AI verdict — pass or fail —
never decides a merge.

The opposite extreme fails too. Where AI findings are collected as mere
advice, the channel decays into a write-only log: real defects sit unread
for weeks.

The stable middle ground: AI reviewers *discover*. A deterministic gate then
enforces that every relevant finding receives exactly one recorded
decision — fix it now, schedule it, accept it deliberately, or dismiss it as
a false alarm. The decision unblocks; a rerun never does. Re-running a flaky
verdict until it passes selects an outcome — it does not verify one.
The gate that enforces this is itself deterministic: what blocks is either
a red deterministic test or a mechanical bookkeeping fact — a finding
still awaiting its decision, an expired entry, an overfull queue. The
block never attaches to an AI verdict; it attaches to something a machine
can re-check and an agent can resolve by an explicit act.

#### 6. The deterministic layers: static checks and ratchets, unit and integration tests, API and UI journey tests, and requirement fitness functions — every acceptance criterion is covered by at least one of them.

Five deterministic layers, from cheapest to most complete:

- **Static checks and CI ratchets**, on every commit: linting, type checks,
  and the ratcheted baselines — test coverage on changed lines, complexity,
  dead code, duplication, architectural layering — each on a baseline that
  may only shrink.
- **Unit and integration tests**, written and maintained by agents alongside
  the code: the fast feedback of the inner loop, per service.
- **API end-to-end journey tests**: assertions for acceptance criteria,
  executed against the deployed twin environments through the same API a
  real client uses.
- **UI journey tests**: the same, driven through the user interface.
- **Requirement fitness functions**: load tests for latency and
  availability budgets, dependency and egress checks for residency and
  supply-chain rules — the deterministic enforcement each requirement names.

The completeness rule turns this list into a safety property: **every
acceptance criterion is covered by at least one deterministic end-to-end
test.** A criterion without coverage is recorded as debt in the
register — never silently omitted. Debt is not the only honest state.
Some criteria resist deterministic form — visual quality,
comprehensibility, behavior of infrastructure outside the system's own
surface; those are exempted by explicit human decision, reason recorded,
and watched by the AI review layer alone. Forbidden is only the third
state: silent omission. This is what keeps the rule
honest: "only deterministic checks block" is safe only if the blocking layer
is complete by construction, so that the AI layer hunts the
residue — behavior nobody has asserted yet — instead of compensating for
missing tests.

#### 7. One AI reviewer per journey, one auditor per ADR and per audit-enforced requirement, one reviewer per norm change — and a verdict about behavior rests only on observed behavior, never on code.

The AI review layer is fully enumerable: one reviewer per norm, one
reviewer per change to a norm. Standing, per norm:

- **One reviewer per journey**, on every verification run. The reviewer
  is a small pipeline, not a single session: an *executor* drives the
  running platform through the user interface like a real user and through
  the API like a real client, and records what it observes; a *judge* in a
  separate session — holding only the journey's text and the recorded
  evidence, no tools — delivers the verdict. Each journey gets its own
  fresh pair, so no journey's verdict can color another's. This is the gap
  no test suite covers: behavior nobody has written an assertion for yet.
- **One auditor per ADR and per audit-enforced requirement**, on a fixed
  cadence; findings go to the register, never into a merge gate.
- Requirements with deterministic enforcement have no AI reviewer —
  machines settle them. A test's standing watcher is the test itself.

Triggered, per norm change:

- **One consistency reviewer per human edit** to journeys or requirements.
- **One ratifying judge per agentic change** to tests or ADRs.

Nothing else has a reviewer — deliberately. A concern that appears in no
norm is watched by nobody: if a cross-cutting property matters — say, the
same figure must match across two surfaces — it must be written down as a
criterion, and then it has its watcher. The enumeration forces implicit
expectations to become explicit norms.

The evidence rule cuts across all of it. A reviewer judging *behavior* sees
only the journeys and requirements and the running system — never the
implementation. Two reasons: knowledge of the implementation biases the
judgment toward what the code suggests instead of what the journeys
demand; and code and
its comments are written by the implementer — they are claims, not proof.
Reading code is legitimate in exactly two roles, where code is the *object*
of examination rather than evidence of behavior: the ratifying judge reads
test changes, because tests are norms written as code; and conformance
auditors read code and configuration read-only, because structural norms —
architectural layering, dependency and residency rules — have no runtime
behavior to observe.

Two prohibitions hold without exception. AI reviewers never re-judge what a
deterministic test already asserts: machine-checkable facts are settled by
machines, and a reviewer's opinion on them carries no weight. And they never
review output of their own session.

The asymmetry with the merge gate follows a rule: an AI verdict is given
authority only where the act is rare, a wrong refusal fails safe, and
appeal to the human exists. Merges fail all three tests; ratifying a
weakening passes them — it is rare and deliberate, the conservative error
is the cheap one when guarding the oracle, and a rejection can escalate
as a decision request.

### Flow — from twin environments to merge to production

How a change is judged and where the results travel: the twin-environment
comparison, what blocks a merge, what gets registered, what it takes to
close a finding, and what promotion demands.

#### 8. Every change is verified against two twin environments — base without it, head with it; their behavioral difference is the basis of every verdict that needs attribution.

Both environments come up fresh, identically provisioned, production-like
and disposable — driven with the same stimuli.
The behavioral difference between them is the primary evidence of
attribution — far stronger than an agent's claim or a historical baseline,
but evidence, not proof: timing, external services and residual
nondeterminism can differ between twins, and a scope label can be
challenged like any other observation. Not every layer needs the pair. The cheap
deterministic layers — static checks, unit and integration tests — run on
the change alone: a deterministic red is attributable by itself and blocks
regardless of what base would show. The twin comparison exists for the
verdicts where attribution decides everything — end-to-end behavior and
the AI reviewers' findings, where the difference between "introduced" and
"pre-existing" is the difference between blocking and registering. The AI reviewers, too, judge comparatively ("is head
worse than base anywhere?") — a form that has proved far more stable in
operation than absolute scoring. The set of journeys to verify is computed mechanically from the
change, never declared by the agent; if it cannot be computed, everything is
verified. A full sweep additionally runs nightly. The pair is the
model's largest standing cost — a deliberate trade of machine hours for
attribution; whether it still earns its keep is a standing measurement
question, not an article of faith.

#### 9. A merge is blocked by unmet declared criteria and by regressions the change introduces — never by pre-existing defects, which are registered instead.

Three things block a merge. First, failing mandatory deterministic tests.
Second, unmet declared criteria: the acceptance criteria declared for the
change must pass on head — "it already failed on base" does not apply to
them. Third, regressions introduced by the change,
relative to base: fixing those is part of the change by definition. For
non-critical regressions, every decision is available; for critical ones,
only two: fix, or do not merge.

Pre-existing defects — findings that reproduce on base — never block: they
are recorded in the register. The same applies in the other direction:
defects discovered along the way are registered, not fixed inside the same
change. The same discipline holds for scope: a change declares up front
what it will deliver, and that scope only ever narrows while the change
runs — a slice that turns out to be separable is split off, and what turns
out to be additionally needed becomes its own change with its own
declaration. A change that widens while it runs dissolves the very
attribution the twin model pays for: its verdicts speak about a scope
nobody defined, and its cost grows past the point where anyone can tell
what bought it. This attribution model is also what keeps changes small: a
large change produces many attributable regressions and becomes expensive;
a small one stays cheap. Because AI verdicts vary between runs, no single run
changes durable state: state changes require multiple concordant runs, the
base/head labels are randomized, and contradictory verdicts escalate.

#### 10. A finding is closed only by a fix plus a regression test: what an AI reviewer discovered once, deterministic tests check from then on.

When an AI reviewer finds a defect and the defect gets fixed, the fix must
include a regression test that checks exactly that point. Without the test,
the finding stays open.

The reason: AI reviewers notice things nobody wrote a test for — a total
that does not match its chart, a button visible to the wrong role. Once such
a defect is known, it is almost always trivial to check with an ordinary
test. The hard part was noticing it, not checking it.

The effect over time: every discovery permanently extends the deterministic
test suite. The same defect cannot return unnoticed, and the AI reviewer
does not have to re-find it on every run. The AI reviewers work as
generators of new test cases; the test suite keeps their results. The more
they find, the less the system depends on them.

#### 11. Findings, debts, and decision requests live in one capped register — the human's single inbox; what is reported only in chat does not exist.

Agents narrate. A single session produces thousands of lines of chat
output, and whatever matters inside it is lost the moment the session
ends — a transcript has no state, no owner, no follow-up, and nobody reads
it twice. The rule is therefore strict: information reported only in chat
has no standing. An agent that finds something relevant writes a register
entry; that act — not the mention — creates the obligation.

The register is a single, typed list: findings (with severity class), debts
(refactoring and coverage needs), **decision requests** (context, options,
recommendation — the formal escalation to the human), open acknowledgments,
and **change orders** — the human's side of the queue: what to build next,
stated as context and target state. Every entry carries an owner and an expiry date — an
expired entry blocks again. The stock is capped like a Kanban queue, grows
only by explicit decision, and shrinks through evidence. For the human, the
register is the single inbox: the one place to check, and the only channel
from which anything is owed.

#### 12. Merging is not deploying: the merge gate judges relative to base; the promotion gates judge absolutely.

The merge gate onto the development trunk asks only: no new regressions,
declared criteria met. That keeps incremental construction possible —
unfinished work may live on the trunk; a UI may exist before its security
layer does. This does not collide with the rule that the trunk never
carries a journey or requirement it does not fulfil. What may be
unfinished is *capability* — things the specification does not yet
promise. A normative commitment, once ratified, merges only together with
the code and tests that satisfy it; and where a reviewer later finds the
trunk falling short of one, that shortfall becomes a visible register
entry, never a silent breach. Unfinished capability is allowed;
unfulfilled commitments are not. The promotion gates toward staging and production are fed from
the register and are absolute: nothing is promoted to production while open
entries of defined severity classes exist — security holes, data loss, auth
bypass, crash loops — no matter how old they are. The severity classes are
enumerated, and a classification must be backed by demonstrable evidence: a
reproducible exploit, a failed request, a measured crash loop. An agent
cannot assert criticality without such evidence; unclear cases become
decision requests.

### Longevity — the codebase and the harness itself

What keeps the system healthy over the long run: mechanical brakes on
decay, and a harness that must justify its own existence.

#### 13. Everything that is not mechanically checked drifts — instructions first of all; growth is stopped by ratchets, shrink needs a named actor.

In an agentic organization, prose does not hold. Agents follow instructions
literally, including the stale ones, and nothing pushes back when a
document and reality part ways — so a rule that no machine compares against
reality must be presumed stale, and the fastest-decaying artifact is
precisely the one meant to prevent decay: the written instruction. Nor is
decay confined to product code: any part of the system that writes
artifacts can be its source, the verification machinery included; an
organization that only inspects its code will find its repository decaying
anyway. Quality baselines therefore may only shrink — freezing is cheap and
reliable. But a frozen baseline does not shrink by itself: without a named
actor with a standing mandate, the stock stays exactly where it was frozen;
and without cycles cheap enough that cleanup is worth its price, even the
actor stays idle.

#### 14. The harness is judged by its kill record: every mechanism carries a removal condition from birth, and a metric nobody would act on is not collected.

Felt diagnoses mislead in both directions: they see crises where none are,
and miss the ones there are — and every mechanism looks useful from inside,
so without measurement a harness accretes protections against imagined
weaknesses while the real cost sits elsewhere, unnoticed. Therefore each
mechanism records, at its introduction, the model weakness it assumes and
the measured condition under which it is removed; the review cadence walks
that inventory and executes what has fallen due. A measurement earns its
collection cost only if some possible result would retire a mechanism,
redirect a priority, or overturn a diagnosis — a number nobody would act on
is decoration. Without the kill record, growth is the default, and the
harness itself becomes the next legacy system: one that can only grow
eventually costs more than it saves.

#### 15. The harness's own operation is captured step by step in one durable record; operational evidence that exists only in a session transcript does not exist.

Agents report their own work as efficient, and within a single session they
usually are. The time of a change disappears between the steps: in tool
calls, in testing against real infrastructure, in the handoffs between
sessions — and none of these seams reports itself. Worse, when something
goes wrong, the diagnosis tends to live where the work happened: in the
transcript of the session that hit it — plausible, unverifiable, and gone
with the context. So the rule of the register extends to the harness's own
runtime: every step a change passes through — sessions, environment builds,
test runs, gates, merges — writes its timestamps, outcome, and failures
into one cross-cutting durable record at the moment they happen. Retries,
swallowed errors, and incidents are entries, not memories. And the record
must not become a second transcript: one structured line per step, no
prose — but each line self-contained and free of insider shorthand, so
that a reader understands it a month later without the session that wrote
it. A record only its author can read has failed its purpose. Where the
time of a change went is a question the record answers, never the agent;
an analysis that can cite only a transcript as its source is, by that
fact alone, not evidence. A human must never have to let this system run
opaquely.


#### 16. Recurring work runs as written procedure, not as a chain of fresh decisions. What a machine can check is enforced by a script that stops the work. Everything else is written down with the reason it exists. This includes the step from one task to the next, so the rules outlive the instruction that set them.

Most of what an agent does in a day is not deciding, it is executing:
standing an environment up, verifying a change against it, picking the
next item by priority, closing an entry with its evidence. Each of these
was decided once. An agent that does not find the sequence written down
derives it again, and deriving turns every step back into a decision that
can come out differently — the instability of long autonomous runs is the
sum of those small decisions, not the difficulty of the work. It is also
expensive: verification runs against freshly built environments and LLM
judges, so a step taken in the wrong order does not cost a retry, it
invalidates the evidence the run has just produced.

Written procedure takes three forms.

- **A step a machine can check** is enforced where the mistake would
  happen: the script that was about to proceed stops instead. An
  instruction that only reminds is advisory, and an agent trying to be
  helpful skips it.
- **A step it cannot check** is written down together with the failure it
  prevents. An agent that cannot see why it was stopped looks for a way
  around, and a step whose purpose is invisible is the first one dropped.
- **The step between two tasks**, where an instruction that lives only in
  the session's context has usually been summarized away, re-states the
  mandate and its limits, picks the next item by a fixed rule, and forces
  a stated outcome: continue with a named item, stop at a named obstacle,
  or finish on purpose. Saying nothing is not one of them.

Real decisions remain, but in written procedure they are the exception and
marked as such. And procedure is fallible the way code is: a wrong one is
followed exactly, every time, so it is versioned with the machinery it
governs, covered by tests, and changed by a recorded change — never edited
from memory. None of this is peculiar to agents. A human team that decides
every step on the fly is unreliable for the same reason, and answers with
processes that are written down and known to everyone.

### Contact with reality — what verification runs against

One principle stands apart: not what is checked or who judges, but what
the checks are pointed at.

#### 17. Maximize the checks that touch reality: end-to-end and integration verification runs against the real neighboring systems wherever that is possible at all, and its cost is accepted.

A test double encodes what its author believed the other system does, and
can only ever confirm that belief — including where the real system
changed last month. The defects that reach customers live in that gap: a
request field the provider began refusing, a mail path that silently
stopped delivering, a response shape that moved. Against a stub, all three
pass forever.

In an agentic organization a second reason weighs heavier. An environment
that cannot reach the real system manufactures excuses: where a red is
explained by an unreachable dependency the environment could never have
reached anyway, the explanation cannot be checked — and an agent under
pressure to finish prefers the explanation that ends the work. Faked
surroundings do not merely miss defects; they generate false attributions
the harness cannot refute. Hence a narrow rule: an external cause is a
claim until the other system's own answer is recorded verbatim; until then
the finding stays open and belongs to the system that raised it.

The cost — credentials in every environment, per-call charges, longer
runs, the intermittent failure of systems nobody controls — is not argued
away; it buys the only evidence that generalizes to production. Access to
non-production dependencies is a low-sensitivity, high-frequency good,
seeded frictionlessly; an environment missing it is a delivery defect, not
a circumstance. Intermittent failure is triaged like any other finding,
never grounds to retreat to a double. Where reality genuinely cannot be
touched — destructive operations, charges beyond a budget, credentials
that exist only in production — the substitute is a named exception with
its reason recorded, watched like any other gap; forbidden is the silent
double. And the direction runs one way: moving a check from a real
dependency to a substitute weakens the oracle and takes the same guarded
act as removing a test; moving it toward reality is free.

---

## Part II — From principle to practice

Part I states the model. Part II shows one concrete realization of it —
the shape these principles took in the production installation this
document draws on. Read it as a reference implementation, not as part of
the claim: where Part I says what must hold, this part shows one way it
can. Skip freely; return when building.

### The artifacts and where they live

| Artifact | Location | Changeable by |
|---|---|---|
| Harness rules: constitution, judge rubrics, gates, workflows | Dedicated harness repo, version-pinned; server-side protected | Humans only |
| Journeys + requirements | Product repo, write-protected area | Humans (with agentic consistency review); agents hold a formal proposal right via decision requests |
| ADRs (append-only decision records) | Product repo | Agents, under procedure: proposer ≠ ratifier, checked against journeys, requirements, harness rules |
| Tests | Product repo | Agents; tightening is free, weakening is a separate, visible act |
| Register — findings, debts, decision requests, acknowledgments, change orders; the human's single inbox and the agents' work queue | Product repo; dispositions only by non-implementers or the human | Process-bound |
| Change orders — context and target state, never implementation prescriptions | Register (typed entry) | Humans; agents pre-draft the journey or requirement change and derive units of work |
| Verdict attestations (small, checksum-bearing) | Product repo, part of the merge history | Verification runs |
| Full protocols, transcripts, raw material | Object store, digest-bound, retained permanently | Verification runs (append-only) |
| Code | Product repo | Agents, freely |

### Identities and permissions

Three classes of identity, mirroring the layers of authority — and the
agentic side is
deliberately a plurality, not one account: the separation of powers only
holds if the powers cannot share a login. The **human** holds the only
administrative account: repository settings, write-protection rules,
harness-repo merges. **Implementing identities** write code, tests, ADRs,
and register entries for their own findings — they cannot change
protection rules, judge rubrics, or gate definitions. **Verifying
identities** — executors, judges, auditors, reviewers — are separate from
the implementing ones and from each other where their interests conflict:
they write verdict attestations and object-store protocols, nothing else,
and can write no product code at all. The gate refuses dispositions
authored by the identity that implemented the change — a rule it can only
enforce because the identities are distinct. One shared agent account
would quietly collapse the judiciary into the executive; the number of
identities follows the number of separated powers, not convenience.
Enforcement is server-side permission, not convention.

### The processes at a glance

Eight processes make up the whole operation. Each is described below or in
Part I.

| Process | Trigger | Who acts | Outcome |
|---|---|---|---|
| Change lifecycle | Change order, register entry scheduled for fixing, or a ratified change to journeys or requirements | Implementing session, verifying sessions, gates | Merged change on the development trunk |
| Journey or requirement change | Human intent, or a decision request | Human + consistency reviewer | Ratified change — handed to the change lifecycle automatically |
| Test/ADR change | Proposal by an implementing session | Ratifying judge | Accepted norm change, or rejection with reasons |
| Conformance audit | Fixed schedule | One auditor per ADR and per audit-enforced requirement | Register findings |
| Nightly sweep | Schedule | Full deterministic + AI verification of the trunk | Register findings |
| Triage cadence | Human, roughly weekly | Standardized triage + janitor | Worked-off queue, one decision briefing, indicator check |
| Review cadence | Human, roughly quarterly | Deep review of the harness | Recalibrated reviewers, removal decisions, rule-change requests |
| Promotion | Human | Promotion gates fed from the register | Deploy to staging/production, or a named blocker |

### The procedures as files

Every inner harness in use today offers some form of named, reusable
procedure that is loaded on demand rather than carried in the session —
the vendors call them skills, prompt files, workflows or rules, and an
open format for exchanging them between tools has begun to converge. The
outer harness uses that mechanism but does not depend on any particular
one of them. What it requires of the
mechanism is only this: a procedure is addressable by name, it is loaded
fresh at the moment of use, it is versioned together with the machinery it
governs, and it can ship executable scripts next to its text. Where an inner harness
offers nothing of the kind, plain files with a fixed location and a
strict loading rule do the same job less comfortably.

Ten procedures cover the operation described in this document. Each pairs
a short written text — what to do, and which failure the step prevents —
with the scripts that enforce whatever is mechanical about it.

| Procedure | What it fixes in place |
|---|---|
| Start a change | One issue per change; the scope is declared before any implementation exists |
| Verify a change | Both environments up and running the intended revisions before a judge starts; the compact attestation is committed, the full protocols go to the object store |
| Record a disposition | Evidence re-checked by a session that did not implement; the closure rules per entry type |
| Adopt a new harness version | Version pin plus exactly one register disposition; the referenced tests must exist in the pinned version |
| Finish and merge | Per-change scope reset, merge only on green checks, trunk confirmed afterwards |
| Move to the next task | Preconditions, mechanical selection of the next item, and an explicit outcome |
| Triage sitting | What is read, what must be decided, what may close and on what evidence |
| Handle an incident | Record first, register second, fix as its own change with a test that pins it |
| Run the scheduled full sweep | How it is started, read, and repaired after a failure |
| Write a register entry | The row format and the evidence each closing status requires |

The last one is a reference rather than a sequence: it is loaded whenever
the register is touched, not invoked as a step. The first nine are
sequences, and the sixth is the one that keeps the other eight running
after the instruction that started them is gone.

Like every other mechanism, each procedure carries a removal condition:
one whose steps a model no longer omits has become overhead, and the
review cadence retires it.

### The lifecycle of a change

1. **Declaration.** Every change starts from one of three triggers: a
   change order, a register entry that triage scheduled for fixing, or a
   ratified change to the journeys or requirements waiting for its
   implementation (see "Changing the journeys and requirements"). The implementing agent session declares
   which acceptance criteria the change introduces or fixes. The declaration can only add obligations; it
   can never exempt a finding from blocking.
2. **Implementation.** The coding session cannot declare itself done: a
   completion hook lets it end only when the deterministic suite is green.
   Whether the harness bounds attempts and spend is an organizational
   choice with three honest options. (a) No limit inside the harness:
   subscription- or account-level spending caps bound the worst case, and
   a mechanism against runaway sessions is added only if runaway sessions
   actually occur — the leanest option, and the kill record's default (every
   mechanism must justify its existence). (b) A per-change budget: attempts
   or spend per change are capped, and hitting the cap is not a failure but
   an escalation — the change becomes a decision request in the register,
   so a human sees what kept burning money. (c) A global rate: the harness
   bounds concurrent sessions or daily spend as a whole, trading throughput
   for a hard ceiling. What is not an option is a silent limit: any cap
   that aborts work must leave a register entry, or the abort becomes an
   invisible seam.
3. **Verification.** Two twin environments (base/head) come up fresh and
   simultaneously. Static checks, unit and integration tests run in CI on
   the head commit — they need no environment. The end-to-end suites (API
   and UI), the fitness functions, and the AI journey reviewers run against
   both twins. The set of journeys to verify is computed mechanically from
   the change — via the maintained mapping between code paths and criterion
   IDs, the same mapping the coverage check uses; if the mapping cannot
   resolve the change, everything runs. That mapping is among the most
   expensive artifacts in the model — a traceability structure with its
   own drift — which is why the fallback points the safe way: when the
   mapping thins out, the run gets bigger, never smaller. Every run leaves a verdict attestation in the repo and its
   full protocols in the object store.
4. **Attribution.** Findings that reproduce on base → registered, not
   blocking. Regressions introduced by the change → a decision is owed
   before the merge (critical: fix, or do not merge); a "fix" decision
   closes only when the fix includes its regression test.
   Declared criteria must pass.
5. **Merge.** Automatic, without human approval, as soon as the mandatory
   checks are green and all owed decisions are recorded.
6. **Promotion.** Staging and production have their own, absolute gates fed
   from the register (severity classes). Merging never means deploying;
   promotion is a separate, human-triggered step.

### Changing the journeys and requirements

Journeys and requirements carry stable, numbered acceptance-criterion IDs.
Declarations, tests, findings, and the scope computation all reference these
IDs; an ID, once assigned, is never reused. Journeys and requirements change
through their own process, not through the change lifecycle:

1. The trigger is human intent, or a decision request from the register
   ("a decision is missing here").
2. Agents pre-draft the edit: the changed journey or requirement text, new
   acceptance criteria, and the list of affected projections — which tests
   and which reviewers will have to change with it.
3. The consistency reviewer checks the draft against the entire existing
   body: contradictions, testability gaps, overlaps. Its report is
   non-blocking but must be acknowledged; the acknowledgment is a register
   entry.
4. The human ratifies the edit in the write-protected area. Nobody else
   can.
5. Ratification merges nothing by itself. The ratified delta is handed to
   the change lifecycle automatically: implementing sessions pick it up,
   its acceptance criteria become their declaration — the scope is fixed
   at that moment; widening it means opening a new change — and the delta
   merges together with the tests and code that satisfy it — so the trunk never
   carries a journey or requirement it does not fulfil. A large delta is closed as a
   sequence of small changes, each taking its slice and declaring its own
   criteria. The human triggers nothing after ratification; the next human
   contact is, at most, a decision request.

A mechanical coverage check runs alongside: every criterion ID must map to
at least one deterministic end-to-end test. A criterion without one creates
a debt entry in the register automatically — coverage gaps
are visible, never silent.

### Changing tests and ADRs

1. Any implementing session may propose a norm change: a new ADR, a
   superseding ADR, new tests, or a test weakening.
2. Tightening needs no ceremony: new tests, stricter assertions, and new
   constraining ADRs merge with the change that brings them.
3. Weakening is a separate, visible act: removing or relaxing a test, or
   superseding an ADR, goes to a ratifying judge in a fresh session. The
   judge sees the diff and the norm layers above it — journeys,
   requirements, harness rules — and not the implementation that motivated
   the change. It approves or rejects, with reasons; the outcome is
   recorded.
4. ADRs are append-only: a superseded ADR stays in the record and points to
   its successor. History is never rewritten.

### How the AI reviewers work

**Journeys.** Two sessions per journey, per verification run. The
*executor* has exactly the tools a user has — the user interface and the
API — plus the journey text. It performs the journey's steps on both twin
environments with the same inputs and writes an evidence log: actions
taken, observations made, values seen verbatim. The *judge* is a second
session with no tools at all: it receives the journey text and the two
evidence logs, and answers two questions — are the acceptance criteria met
on head, and is head worse than base anywhere? Every finding must quote the
evidence log and carries a proposed severity class. Because verdicts can
flip, a failing verdict is sampled more than once, base/head labels are
randomized, and contradictory samples escalate instead of averaging.

**ADRs.** One auditor session per ADR, on schedule. It reads the ADR's
decision, investigates the repo read-only — code, configuration, dependency
manifests — and reports: honored, or drifted, with a file reference for
every claim. Findings go to the register; conformance is never a merge
gate.

**Requirements.** The requirement's named enforcement decides. Measurable
requirements — latency, availability, cost — are checked by deterministic
fitness functions; no AI is involved. Structural requirements —
data residency, permitted third-party services, security posture — get an auditor
like an ADR: it reads dependency manifests, egress and infrastructure
configuration read-only, and reports with references.

**Norm changes.** The consistency reviewer (journey and requirement changes) and the
ratifying judge (test and ADR changes) receive the diff and the norm layers
above it — never the implementation. Their reports become register entries.

**Reviewer hygiene.** Every reviewer runs with a pinned model and prompt
version; both are recorded in the verdict attestation, so a silent model
upgrade cannot silently shift verdicts. Reviewer quality is measured
against a human-labeled golden set (see Measurement); a degrading hit rate
is an agenda item of the review cadence.

### The severity classes

Every finding in the register carries a severity class, and the class
drives exactly two decisions. At the **merge gate**, it determines what may
be done with a regression the change introduced: non-critical findings
allow all four decisions — fix, schedule, accept, dismiss — while critical
ones allow only fix-or-do-not-merge. At the **promotion
gates**, it determines what blocks: each gate names the classes it refuses,
and production refuses at least everything critical, regardless of the
entry's age.

The classes are defined once, in the harness rules — human-owned and out of
the agents' write reach, so no agent can soften the scale it is judged
against. The reviewer that raises a finding proposes the class and must
attach the evidence the class requires; the classification becomes binding
with the triage decision, made by a session that did not implement the
change, or by the human. An ambiguous classification is itself a decision
request.

The classes:

- **Critical** — demonstrated unauthorized access; demonstrated data loss
  or corruption; secret leakage; measured crash loop. Consequence: newly
  introduced, the only options are fix or do not merge; open in the
  register, it blocks promotion to production always.
- **Substantial** — functional regression of an acceptance criterion;
  violation of a requirement with a named means of enforcement.
  Consequence: registered with owner and expiry; whether it blocks a
  promotion is defined per target environment in the harness rules.
- **Minor** — cosmetic deviations, wording, tolerated variance within
  budgets. Consequence: registered, counts against the register cap, never
  blocks a promotion.

### The cadences

**Triage cadence — roughly weekly, human-triggered, mostly automated.**

1. A pre-pass sorts every open register entry into decision-free (the fix
   is unambiguous, no trade-off involved) or decision-needing.
2. Decision-free entries are worked off autonomously as small changes, each
   through the normal change lifecycle. A change that fixes a finding must
   contain the pinning regression test — the gate refuses to close the
   finding without it.
3. Decision-needing entries are bundled into one briefing — context,
   options, recommendation per entry.
4. The janitor runs: dead code, unused dependencies, stale documents,
   orphaned artifacts — small changes, auto-merged when green.
5. Ratchet baselines are reviewed for tightening, and the indicator set is
   compared against the previous cadence. Loosening a baseline never
   happens in passing — it is its own isolated, justified change.

The human's part of all this: read one briefing, make the bundled
decisions.

**Review cadence — roughly quarterly, human-triggered.**

1. Reviewer calibration: verdicts are sampled against the golden set; hit
   rate and flip rate per reviewer are updated.
2. Gate probe: for a sample of past changes, the counterfactual question —
   what would the gates have admitted or blocked under different rules?
3. Removal check: every harness mechanism is examined against its removal
   condition — has a better model made it obsolete?
4. Nightly-sweep and audit findings are checked for patterns a single
   triage would miss.

Outcomes that would change harness rules become decision requests; only the
human amends the rules.

### The nightly sweep

Once a day, the full verification runs without scoping: every journey,
every reviewer, all fitness functions, against the current trunk — compared
with the previous nightly's trunk, so findings are attributable to the
day's merges. Everything found goes to the register. The sweep is the
safety net under the per-change scoping: whatever the mechanical scope
computation missed during the day surfaces here at most one day late.

### The human processes

The human has exactly six touchpoints in this system:

1. **Order and set the journeys and requirements.** Work is commissioned as a change
   order in the register: context and target state, never implementation
   prescriptions — agents pre-draft the resulting journey/requirement
   change, the human ratifies it in the protected area (see "Changing the
   journeys and requirements").
2. **Decide.** Answer decision requests from the register — each in the
   form "context, options, recommendation," written so it can be decided in
   minutes.
3. **Trigger the triage cadence** and read its one briefing (see "The
   cadences").
4. **Trigger the review cadence** and decide the rule changes it proposes
   (see "The cadences").
5. **Promote.** Trigger promotion to staging or production; the gates
   answer with a pass or a named blocker from the register.
6. **Amend the harness rules** — the rare, deliberately heavy act outside
   the agents' reach.

### Measurement

A harness like this runs mostly unattended; from the outside it is a
black box that occasionally merges. Measurement exists to open that box.
The examples below suggest what is worth capturing — they are not a
complete set, and deliberately not a scorecard. Four questions organize
them.

**Where do time and tokens go?** Per iteration: wall-clock and token
spend, split into implementing, verifying, and waiting — mean and
variance, and the variance read per change class, because a stable mean
with wild outliers means the outliers have a cause. Implementing a change
and verifying it should take roughly the same time; a change class that
persistently misses that target is itself a prioritized harness finding.
Break out, separately, what infrastructure failures cost. A twin
verification is not a trivial setup — two complete environments, freshly
provisioned and deployed for one comparison — and much can go wrong
below the actual work: image builds, provisioning races, cold caches,
stale leases, a base environment that arrives late. Every one of these
burns time and tokens without producing evidence, and a run invalidated
halfway is paid for twice. Expect the verification phase to dominate the
wall clock — and expect most of it to be waiting on environments rather
than checking. That one split decides where investment goes: into the
judges, or into the provisioning underneath them.

**What do the layers let through?** Block rate per layer and per gate.
As a rule of thumb, sustained rejection above roughly twenty percent
points to a structural weakness — and because verdicts are comparative per increment, a desolate
overall codebase cannot drive this number up: pre-existing faults are
exonerated, so a high reject rate always indicts the process, never the
backlog. Acceptance near one hundred percent deserves the same suspicion:
a gate that never blocks may be gating nothing — typical culprits are a
check condition that is vacuously satisfied and an error suppression
that turns every outcome into success. A strong asymmetry between layers
is a signal of its own, and it can only be read together with the next
question.

**What becomes of what they catch?** The disposition distribution is the
yield ledger: what share of findings the twin comparison disposes
mechanically — expect it to be the large majority — and how much of the
rest ends as a fix, rather than as a wrong observation or as a repeat
sighting of an already-registered finding. AI reviewers describe what
they see in fresh words each run, so the same defect returns under new
phrasing and is closed as a duplicate pointing at the original entry. A
high block rate with high yield is a working sensor; without yield it is
an alarm bell wired to nothing. Three companions: the sighting frequency
per finding — the length of exactly those duplicate chains — is a
priority and severity signal that otherwise stays buried in them, with
the caveat that it also measures how often something is looked at; the
discovery→test conversion rate says how reliably what a judge found
once becomes a deterministic assertion; and the recurrence rate among
findings closed as fixed checks whether that assertion holds — a defect
that returns after its closure indicts either the test that was supposed
to pin it or the closure itself. This number should sit at zero, and
every exception is a harness finding, not a statistic.

**Is the instrument itself still healthy?** Twin availability, the
success rate of the scheduled full runs, and the noise volume per run
over time — the last being the most honest progress curve, because it
falls only when things actually get fixed. A product under this regime
can become more stable than the instrument that measures it, and it
happens sooner than expected; without these numbers, a sick instrument
is indistinguishable from a sick product.

**The change record.** Every machinery entry point appends
one line to a cross-cutting event record: timestamp, actor, action,
outcome, duration, and a wait class from a small fixed taxonomy
(implementing, environment build, environment flake, active verification,
form rework, waiting on CI, waiting on merge, waiting on the human,
platform fault, result produced but not consumed). Retries and swallowed
failures are events; incidents and their diagnoses are written to the
record at the moment they occur — a session that diagnoses a failure
registers it before acting on it, so no operational evidence survives only
in a transcript. Entries are single structured lines with fixed fields and
plain names — never prose, never session shorthand; the record must stay
skimmable end to end, or it becomes the transcript it replaces. The record is rolled up on the existing cadences into
per-change flow figures; prioritization of harness work follows the
measured wait classes, not impressions. Where richer tooling is warranted,
the record maps onto emerging standards that fuse distributed tracing with
agent telemetry (e.g. OpenTelemetry's GenAI conventions) — the record is
the contract, the tooling is exchangeable.

**The seam rule.** Every artifact handoff between sessions or mechanisms
has a validation the producer runs before the expensive consumer does. A
form error must be catchable locally in under a second; and any form rule
that voids an expensive artifact — a full verification run, a nightly
sweep — is by that fact a harness defect and gets a register row, never a
shrug and a rerun.

Four instruments have proven the most informative: the **dispositioned
register** (measures AI-reviewer precision through the distribution of
fixed/scheduled/accepted/false-alarm), the **verdict corpus** in the object
store (measures verdict flip rates, noise classes, cycle costs), the
**classified git history** (measures the harness tax: the verification
machinery's share of commits and fixes), and **counterfactual analysis**
("what would a different gate design have admitted?"). The platform's DORA
metrics (lead time, deploy frequency, change failure rate, MTTR) serve as
the external anchor. Harness changes are based on measurements and
falsifiable hypotheses with isolated before/after comparison, not on
impressions.

**What the numbers do not see.** Everything above measures precision;
nothing measures what all layers missed — an escape rate exists only if
later-found defects are dated back to the merge that admitted them. Green
means conformant, not right: none of it checks the journeys themselves,
so it is worth tracking how often a finding resolves as a change to the
specification rather than to the code. The gap between disposable
verification environments and production is invisible by construction and
surfaces exactly at deployment. And the cheapest number of all is usually
missing: the human queue — open decision requests and their age. In a
system whose rule is that the human decides, throughput is bounded by
precisely that queue.

**The direction stays human.** These numbers open the box; they explain
the machine. Whether the product as a whole is moving in the right
direction is not a question to delegate to the harness, and prescribing
KPIs for it would be pretense. That verdict belongs to the human in the
loop — the cadences exist so that it is made regularly, and made with the
box open.

### Situating the principles

The principles are deliberately built to connect to established practice:
twin-environment comparison with exoneration unifies four proven lines
(clean-as-you-code and diff-aware analysis; canary analysis with a
simultaneous baseline; test exoneration through reproduction on the
mainline; the FAIL-TO-PASS / PASS-TO-PASS split of the agent benchmarks).
Merge gate vs. promotion gate is continuous-delivery standard; the cadences
and the queue cap are Kanban; the write-protection of the verification
machinery follows the privileged-config-repo pattern of large CI systems;
the refusal to let LLM judges gate matches their measured verdict
reliability. The main contribution is the composition: a closed
governance model for engineering without human code review, in which each
mechanism carries the others. Within it, two building blocks lack, as far
as the author knows, a counterpart in published practice: the **ratchet from discovery to determinism**
— every closed finding leaves behind the test that catches it again — and
the **agentic curation of architecture decisions** under append-only
discipline with dedicated conformance judges. The latter is the
consistent consequence of the fact that in a hands-off system technical
decisions were never actually made by the human, so a humanly "frozen"
architecture archive would document decisions nobody made.

---

## What this is for

Coding agents are already good enough that the bottleneck has moved. It is
no longer how much code an organization can produce, but how much it can
be accountable for — and the answer to that question is not another model
or another tool, it is a set of operating rules. Those rules barely exist
yet. Teams are adopting agents far faster than they are adopting a way to
govern them, and the gap shows up as the same recurring doubt: nobody can
say what a system is worth when no human has read what it does.

The seventeen principles are one concrete answer, running in production
rather than sketched. A snapshot from that operation's own records: fifty
changes merged in one 96-hour stretch, none reviewed by a human and none
reverted; roughly nine of ten reviewer findings dispositioned mechanically
by the twin comparison; two vacuously green gates caught by watching
acceptance rates; implementation about a sixth of cycle wall-clock, most
of the rest environment provisioning. They are offered as a contribution to that
discussion, not as a standard: argue with them, copy the parts that hold,
replace the ones that do not. What they show is that the accountability
does not have to be given up. It moves — out of the reading of code, into
the design of authority, evidence, records and procedure. Where that move
succeeds, the size of a team stops being the limit on the size of the
system it can responsibly run.

---

*Provenance: this document originated in the author's work building
TechVera, a European LLM platform, and the production installation
referred to throughout is that platform. The model described here is how
it is actually built and operated; the views and any errors are the
author's own.*

*Haag, Lower Austria — August 2026*

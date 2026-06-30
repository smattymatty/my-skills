---
name: drill-with-docs
description: >-
  The teaching and auditing counterpart to grill-with-docs. Grill interrogates
  a person to converge on decisions and writes them down. Drill takes what's
  already written down and tests whether a head actually holds it, by making
  that head solve constrained exercises derived from the canonical source. Same
  Socratic spine, opposite direction. Use to onboard a new engineer onto a
  system, to calibrate an AI agent against a system before it acts, or to
  self-check whether your own mental model has drifted from your docs (and to
  catch docs that have gone stale).
---

# drill-with-docs

The teaching/auditing counterpart to `grill-with-docs`. Grill interrogates the
human to converge on decisions and write them down. Drill takes what's already
written down and tests whether a head actually holds it, by making that head
solve constrained exercises. Same Socratic spine, opposite direction.

**The Pocock principle:** you do not learn a mental model by reading it, you
derive it by solving a tight problem and getting corrected. Every exercise is
concrete, constrained, and has an answer derivable from the canonical source.
No open-ended "discuss." No pre-explanation before the attempt - the struggle
is the point.

## Setup (do this once)

Drill writes one artifact per session - a dated record of how a session went.
These artifacts are a longitudinal journal of someone's understanding over
time, so they want to live somewhere durable, not get blown away with a build
folder.

Pick a home directory and set it once:

```
DRILL_HOME=./drills        # default: a drills/ folder at the repo root
# or somewhere outside the repo if you'd rather keep the journal separate
# from the code, e.g. DRILL_HOME=~/notes/drills
```

Everywhere below, `$DRILL_HOME` means that directory. Create it on first run if
it does not exist. Nothing else in the skill is path-coupled - point it at the
canonical sources you want to drill and it works.

## When to use

One engine, three targets (pass the target; it changes tone, not the loop):

- **onboard-human** - teach a new engineer the system. Scaffold up, encourage,
  build from zero. Divergence is a teaching trigger.
- **onboard-agent** - make an AI agent solve exercises to calibrate against the
  system before it acts (e.g. before an automated coding run touches the
  codebase). The agent's wrong answers reveal where its context is thin - fix
  the context, not the agent.
- **drift-check (self)** - point it at the author. Divergence between the
  author's answer and the canonical source is the signal, and it is AMBIGUOUS
  on purpose: either the author's mental model drifted, or the doc/code is
  stale. (Both happen. A developer once "knew" a cache TTL was 60 seconds and
  the doc corrected them - model drift. Another time the doc claimed a rate
  limit of 100 req/min, the developer said 1000, and the code had been raised
  to 1000 months earlier - doc drift.) This mode's whole value is forcing that
  fork into the open.

## The loop

1. **Ingest.** Read the canonical source(s) named: an ADR, a runbook, a
   CONTEXT.md, a code module, another SKILL. Extract the load-bearing concepts -
   the things someone MUST hold to operate safely, and the things most likely
   to be misunderstood or to have drifted. These become the exercise set.
2. **Check prior drills.** Look in `$DRILL_HOME` for an existing folder for this
   topic. If one exists, read the most recent entry to see what was already
   covered and where the target did well or stumbled last time. Use that to skip
   what they're solid on and probe what they weren't.
3. **Pose.** One exercise at a time. Problem-first, constrained, concrete. Good
   shapes: "here is a broken nginx.conf, what's wrong and why," "this auth check
   passes - should it? what failure does it miss," "ADR-0007 says Postgres not
   SQLite - given a crash mid-write, what actually protects the data?" Bad shape:
   "explain the database decision."
4. **Attempt.** The target answers. No hints first.
5. **Reveal.** Show the correct mental model and CITE the source (ADR id,
   runbook step, file:line). Mark exactly where the attempt diverged.
6. **Flag divergence** - first-class output, not a failure:
   - *onboard modes:* a knowledge gap. Teach it, then re-pose a variant later.
   - *drift-check:* ask the fork explicitly - "the doc says X, you said Y. Is the
     doc right and your model drifted, or are you right and the doc/code is
     stale?" Route to a re-learn note OR a doc-fix issue. Never silently assume
     the human is wrong; the author's wrong answer is often how you find a stale
     doc.
7. **Progress.** Track concepts covered vs remaining, one branch at a time.
8. **Write the artifact** (see below).

## Interactive format

Header on every turn (parallels grill's):

```
🎯 Covered: N | Gaps/Drift: N | Est. Remain: ~N | Progress: xxx........
```

Per exercise:

```
EXERCISE n - <concept>
<the constrained problem>
```

After the attempt:

```
✅ or ❌  <one-line verdict>
Reveal: <the mental model, in your own words>
Source: <ADR id / runbook step / file:line>
[drift-check only] Doc-or-model? <the fork question, if diverged>
```

## Artifact: where it goes, how it's named

All session artifacts land in `$DRILL_HOME`. Treat this as the journey of the
target's understanding over time - it lives with your reference material, not in
whichever project you happen to be working in. Create the folder on first run if
it does not exist.

Layout, one folder per topic:

```
$DRILL_HOME/
├── README.md                      ← index, one line per topic, latest score
├── auth-flow/
│   ├── 2026-05-29-baseline.md
│   ├── 2026-06-12-after-adr-009.md
│   └── 2026-07-01-drift-check.md
├── billing-model/
│   └── 2026-05-29-baseline.md
└── runbook-philosophy/
    └── ...
```

Topic folder name is the canonical source's slug (`auth-flow`, `billing-model`,
`engineer-onboarding`). One topic = one folder. Don't split the same concept
across folders.

Filename is `YYYY-MM-DD-<label>.md`, where `<label>` is a smart, short tag for
what this session was: `baseline`, `after-adr-009`, `drift-check`, `pre-demo`,
`post-grill`. The date orders chronologically, the label makes the journey
readable at a glance.

## When to update an existing doc vs write a new one

Before posing the first exercise, check the topic folder. Decide:

- **Update / extend an existing doc** when: the canonical source hasn't
  materially changed since the last drill, the concept set is the same, and this
  session is essentially a re-test or a quick top-up. Append a dated "Re-drill"
  section to the most recent file rather than spawning a near-clone.
- **Write a new doc** when: the source has changed (new ADR, runbook rewrite,
  contract update), the concept set is different, the target persona has changed
  (now drilling an agent instead of yourself), or the diff against the last entry
  would be larger than the file itself. Make a fresh dated file with a label that
  explains the why (`after-adr-009`, `after-auth-rewrite`).
- **Show the diff before deciding when ambiguous:** briefly tell the target "last
  drill was 2026-06-12 covering A/B/C, source has since added D and E - new doc?"
  and let them call it. Default to a new doc on tie.

## Artifact format (one file per session)

```
---
topic: <slug>
date: <YYYY-MM-DD>
label: <baseline | drift-check | after-<event> | ...>
target: <onboard-human | onboard-agent | drift-check>
source(s): <ADR 0006, docs/specs/auth-flow.md, ...>
score: <N right / M total>     ← honest, not generous
drift_findings: <count of MODEL + DOC tagged items>
---

# <Topic> drill - <date> (<label>)

One-paragraph read of how this session went. What's solid, what's shaky, what
surprised. Plain language, no euphemism. "Nailed the dispatch chain, fumbled the
token scope" beats "good progress on protocol concepts."

## Concepts covered

- ✅ <concept> - <one-line note on what they got right>
- ❌ <concept> - <one-line note on the divergence, with source cite>
- ⚠️  <concept> - <partial: the framing was right, the specifics were off>

## Drift findings (drift-check mode only)

- `MODEL` <concept>: mental-model drift. They thought X, doc says Y. Re-learn note.
- `DOC` <concept>: doc/code drift. They said X, doc says Y, but the code now does
  X. The doc is stale - open a fix issue against <source>.

## Re-drill list

Concepts to come back to next session, ordered by leverage. Each is a one-liner
naming the concept and the variant exercise that should test it.

## Re-drill — <next date> (<label>)   ← only when extending an existing doc
<append-only block, same shape as above>
```

## The README index

`$DRILL_HOME/README.md` is a flat index, one line per topic, kept in sync after
every session:

```
- [auth-flow](auth-flow/) — last: 2026-07-01 drift-check, 6/8, 1 DOC drift open
- [billing-model](billing-model/) — last: 2026-05-29 baseline, 4/7, re-drill due
- [runbook-philosophy](runbook-philosophy/) — last: 2026-04-12 baseline, 7/7, solid
```

Update this index after writing the session artifact. Don't let it rot.

## Authoring rules for exercises (non-negotiable)

- **Derive the answer from the source. Never invent it.** If the source is silent
  or ambiguous on a point, that is itself a finding - the doc has a gap, surface
  it instead of guessing an answer.
- **One concept per exercise.** Pocock density. No compound questions.
- **Problem before explanation, always.** If you catch yourself explaining before
  the attempt, you have written a tutorial, not a drill.
- **Constrained, with a checkable answer.** If two reasonable people would answer
  differently and both be right, it is a grill question, not a drill one.
- **Cite on reveal.** Every reveal points at the canonical source so it is
  auditable and so a stale source gets caught.
- **Cover the load-bearing set, then stop.** Do not drill trivia. The target is
  the concepts that, misunderstood, would break production or mislead a customer.
- **Score honestly in the artifact.** The whole point of the journey-of-
  understanding file is that it is real. A flattering record is worse than no
  record. If they bombed it, the file says they bombed it.

## Relationship to the rest of the toolkit

- **Pairs with grill-with-docs.** Grill writes the canonical docs; drill tests
  that a head holds them. Run grill to decide, drill to verify the decision
  landed.
- **Generates from canonical sources, so it stays in sync.** Unlike a
  hand-authored onboarding walkthrough, the exercises are derived from the current
  ADRs/runbooks - when the source changes, the drill changes. This is the fix for
  the drift between the walkthrough and the live system.
- **Feeds the next handoff.** A drift-check artifact tagged `DOC` becomes a fix in
  the next handoff (fix the stale doc). Tagged `MODEL` becomes a re-drill scheduled
  next session.
- **Stop condition:** the load-bearing concept set is covered, or (drift-check)
  every divergence has been routed to MODEL or DOC, AND the artifact is written
  and the index updated.

---

*This is a generic, shareable version of a personal skill. It is shaped for one
person's brain and one person's workflow - which means the highest-value thing
you can do with it is read it, disagree with parts, and bend it to your own. The
inverse skill (drill) only exists because its author refused to use the original
(grill) unmodified. Do the same to this one.*

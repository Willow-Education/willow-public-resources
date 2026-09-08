---
name: create-plan
description: Author a Planned or Orchestrated plan another agent can execute without asking a single question. Load before writing any file under plans/.
---

# Create Plan

TRIGGER: `workflow-router` returned **Planned** or **Orchestrated**. Direct work never creates a
plan, and a plan is never written to invoke a skill.

The plan is not a description of the work. It is the **entire** context the executing agent
gets — a different agent, in a fresh context, with no memory of the conversation that produced
the plan, that stops the moment the work *looks* done. Everything it cannot check, it asserts
instead.

So a plan is **self-contained or it is broken**. It answers, in this order:

| | Where | What it has to carry |
|---|---|---|
| **Why** | Objective, Non-goals | the outcome being bought and what is deliberately excluded — intent an agent would need to make a judgment call correctly, never the case for the decision |
| **What** | Acceptance criteria | the finish line, in checkable claims with real values |
| **How** | Phases | ordered work, exact files, expected failures, exact commands |
| **With what** | Resources | every link the work needs: the GitHub issue number, the Sentry issue ID, the dashboard, the doc, the skill that owns a procedure |

Nothing in it may depend on something said in chat. "As discussed", "the approach we agreed",
"the file you mentioned" name things the executing agent cannot see, and the lint fails them.
Write every referent out: the number, the path, the URL — and when the work traces to a GitHub
issue or a Sentry issue, record that issue number or Sentry issue ID in `## Resources`.

## Research first, then write

You cannot write a bounded criterion for code you have not read. Before the first heading, find
the feature's entry points, the service and hooks it goes through, the tests covering it now, and
the nearest thing in the repo that already does what you are about to build.

Two questions decide most of a plan's quality, and reading answers both: **does this already
exist here** — a second way to do what the repo already does is a defect the tests will pass, and
`golden-path-architecture` and `requirements` own those answers — and **what currently depends on
the code I am about to change**, which becomes the regression surface below.

**The plan records conclusions, not the search** — no transcript of what you grepped, no list of
files ruled out. What the reading leaves behind: dated facts in Evidence, the file to imitate in
Existing patterns, the dependents in Regression surface, exact paths in File ownership, real ids
and routes in the criteria, and every link the work needs in Resources.

**Unverified assumptions are not research.** A fact you could not confirm that would change the
plan's shape is a `Human-only` question or the first phase's work — never a sentence written as
though it were checked.

## The one test

**If two competent engineers could read a criterion and ship different behaviour, it is not
ready.** Apply it to every acceptance criterion before writing a single phase.

## Write the finish line first

Acceptance criteria come before phases, always. Phases are how you reach the finish line; you
cannot lay them out before you know where it is. Each criterion is:

- **Observable** — software can check it. A person's judgment is not a check.
- **Atomic** — one claim. "Renders source, date, author, and distance" is four criteria.
- **Bounded** — exact route, file, input, and expected output, with real values.
- **A result, not a recipe** — what must be true, never which algorithm gets you there.

| Defect | Rejected | Rewritten |
|---|---|---|
| Not observable | "the list loads quickly" | `GET /api/schools?districtId=…` returns in under 400 ms against a 10,000-row district |
| Not atomic | "submitting a query renders hits with source, date, author, and distance" | four criteria, one field each |
| Recipe, not result | "use a binary search over the index" | "lookup stays under 5 ms at 100k rows" |
| Unbounded | "handle bad input" | `POST /api/x` without `studentId` returns 400 and writes no row |
| Unfalsifiable | "a real query returns real hits, visible and labelled" | name the query, the fixture, and the two fields that must appear |
| Happy path only | "the import creates students" | plus: "a row whose email already exists is skipped and counted in `skipped`" |

**Never invent the fixture at execution time.** A criterion without concrete values leaves the
executing agent to make up its own input, and "correct" becomes whatever it made up. Name the
fixture, the row, the id, or the file the criterion is asserted against.

## Say what it logs

Nothing in a green suite tells you what the feature does at 3am. Every plan carries
**`## Observability`**: what is logged, at what level, and which failures reach Sentry — with the
fields that make a line useful, not just that a line exists. Where a log or event is load-bearing,
assert it in RED like any other behaviour; an unasserted log line disappears in the next
refactor. "Changes neither" is a valid answer and the lint accepts it.

## Name what you could break

Criteria describe what becomes true. They say nothing about what must stay true, and that is
where regressions live: the caller two files over, the second consumer of the type you widened,
the query that shares the index. `pnpm test:affected` covers the tests that reach your diff — it
cannot cover behaviour nobody wrote a test for.

So every plan carries **`## Existing patterns`** — for each thing being built, the file that
already does it and gets copied rather than reinvented — and **`## Regression surface`**: the
existing behaviour closest to the diff that must keep working, and the test pinning each item.
Where nothing pins it, that missing test is a phase task, not a footnote. A field added to a
shared type means every existing reader of it, named; a query that gains a filter means the
callers that relied on the unfiltered result. "None" is a legitimate answer and a deliberate one;
an empty section is neither, and the lint fails it.

## Two halves of the file

Everything above the first `## Phase` is read once for context: Objective, Evidence, Non-goals,
Acceptance criteria, Existing patterns, Regression surface, Dependencies, Resources, File
ownership.
Everything from the first `## Phase` down is worked through. Keep them apart — an executing agent
that has to re-read strategy prose to find its next action burns context twice.

Pin every fact in **Evidence** to an exact environment and date — "0 rows in `x` on prod,
2026-08-20", not "the table looks empty". Undated evidence is unverifiable a week later.

**Non-goals are load-bearing.** They are the only thing that stops an agent from expanding the
diff into work you already decided against.

## Phases

One phase is **one result that can go red, go green, and be checked by one command**. Not a file,
not a step. "Create the module, add the import, export the type" is a diff, not three phases;
over-decomposition costs more in plan drag than it buys. Split a phase when its contract needs a
paragraph per file, or when half of it could be verified while the other half is still being
written; merge two when the second adds no new RED and runs the same command as the first.

Each `## Phase N — <the result, in one line>` carries:

- `### Depends on` — required for Orchestrated; name the phase and the output it produced.
- `### Contract` — the behaviour, in Given/When/Then with concrete values. This is where the
  acceptance criteria for this phase get their exact inputs and outputs.
- `### Files` — every path the phase may touch, as a table. Mark a row `[P]` when it shares no
  file with another phase runnable at the same time; anything not marked `[P]` is written
  single-threaded. Parallel reads are always fine.
- `### RED` — one failing test per criterion, each naming the failure you expect to see. "Fails"
  is not the entry; "`recordConnection is not a function`" is.
- `### Verification` — the exact command and the result that counts as pass.

Every phase opens with `Closes: 1, 3` — the acceptance criteria it delivers. The lint fails a
phase that claims none and a criterion no phase claims, so nothing reaches the end unowned; a
plan-wide criterion is claimed by `## Verification` instead. And **every phase names a failure
case**, because an agent implements exactly the error paths written and no others.

A phase boundary is a checkpoint: the plan can be stopped there and resumed by a fresh agent
reading `status:` and the RED ledger. Nothing about a phase may depend on conversation the
executing agent never saw.

## Fence what an agent cannot do

Investigation needing a human — a judgment call, an external account, a decision about product
intent — goes in a block an agent is told to leave alone, opening
`> **Human-only — do not attempt.**`. That is the most common way a plan gets silently corrupted:
the one section only a person can complete is the one an agent will confidently "finish".
Anything open that does not block execution goes under `## Retirement` with its owner.

## Only agents read this file

That is the whole rule, and most length problems dissolve into it. A plan carries what makes its
phases executable — objective, dated evidence, non-goals, existing patterns, regression surface,
dependencies, ownership, criteria — and nothing else. Rationale earns a line only where an agent
hitting a judgment call mid-execution would decide differently without it. The case for the
decision, the alternatives weighed, the cost model, the scheduling debate: not moved elsewhere,
**not written**. Anything durable that surfaced in research goes to `docs/` at retirement.

**The context half is capped at 250 lines and the lint fails it**, with a note past 150; the
longest plan here spent 900 lines there. Phases are never capped — a blunt cap buys a deleted
fixture value. Past 400 lines total the lint prints a note: cut the prose, or, if the length is
genuinely executable work, it is Orchestrated and each phase moves to its own file so a worker
loads one phase instead of eight (`references/plan-template.md`).

Why those numbers, and the measurements behind them: `references/why-the-limits.md`.

**Never compress to fit** — concrete values, error paths, and file paths are the plan.

## Check the plan, then stop

```bash
python3 scripts/agent-config/plan_schema.py --authoring plans/<date>-<slug>.md
```

`--authoring` runs the execution schema plus the checks that only matter before approval: a
numbered acceptance list, no unfalsifiable words in it, at least one failure-path criterion, a
non-empty `## Existing patterns` and `## Regression surface`, real paths in every `### Files`, a runnable command in every
`### Verification` and in the plan's own `## Verification`, a `## Resources` section, no
dependency on the conversation, and `status: planning`. Fix what it names. A plan that does not pass is not ready to hand over.

Then **stop**. `status: planning` means awaiting approval; the owner approves before anything is
executed, and `execute-plan` flips it to `executing`. Writing the plan does not authorize
starting it.

## What never goes in a plan

**A review gate** — an independent reviewer is not a required link in any lane (owner,
2026-08-17), so never add one to the chain or make completion wait on one. **Restated skill
procedure** — name `tdd`, `verify`, `supabase-migrations` and let them own their commands; a copy
in a plan is a copy that rots. The plan also ends at verified code: no commit, PR, merge, deploy,
branch switch, or production write, all of which the hooks enforce anyway.

## References

- `references/plan-template.md` — the skeleton to copy, with every required heading in order.
- `references/why-the-limits.md` — the evidence behind the caps and the split rule. Read before
  changing a number, not before writing a plan.

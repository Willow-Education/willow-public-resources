# Plan template

Copy this into `plans/<date>-<slug>.md` and replace every bracket. Headings are matched exactly by
`scripts/agent-config/plan_schema.py`; do not rename them. Delete `## Evidence` and `## Non-goals`
only if there is genuinely nothing to put in them.

---

````markdown
# [The result, as a sentence a person would say]

status: planning
lane: Planned

## Objective

[Two or three sentences: what must become true, and why it is not true now. No implementation.]

## Evidence

[Every load-bearing fact, pinned to an exact environment and date. A table when there are
numbers. Delete this section only when the change rests on no external facts.]

| Fact | Value |
|---|---|
| [`table_x` rows on production, 2026-08-20] | [0] |

## Non-goals

- [What a reader might reasonably expect here and is deliberately excluded, with the reason.]

## Acceptance criteria

1. [Observable, atomic, bounded, with concrete values. One claim.]
2. [The failure path: exact input, exact status or message, and what must NOT have changed.]
3. `pnpm test:affected` is green and `pnpm build` is clean.

## Existing patterns

[For each thing being built, the file that already does it. Copy it rather than inventing a
second way. "Nothing in the repo does this yet" is a valid answer and a claim you have checked.]

| What this change needs | Already done in |
|---|---|
| [a paginated staff table] | [`packages/frontend/src/features/[…]/StaffTable.tsx`] |

## Regression surface

[The existing behaviour closest to this diff that must keep working, and the test that pins each
one. Where nothing pins it, adding that test is a phase task. "None; nothing imports this yet" is
a valid answer — an empty section is not.]

| Existing behaviour | Pinned by |
|---|---|
| [`x` still returns […] for […]] | [`packages/[…]/x.test.ts`] |

## Observability

[What this logs and where a failure shows up. "Changes neither" is a valid answer.]

| Event | Level | Where |
|---|---|---|
| [what is logged, with the fields that make it useful] | [info / warn / error] | [logger / Sentry] |

## Resources

[Every link the work needs, so nothing sends the executing agent back to a person. "None" is a
valid answer — but if the work traces to a GitHub issue or a Sentry issue, its issue number or
Sentry issue ID is required here, not optional.]

| Resource | Link |
|---|---|
| [GitHub issue #N] | [https://github.com/Willow-Education/willow-app/issues/N] |
| [Sentry issue ID, e.g. WILLOW-APP-123] | [url] |
| [The skill that owns a procedure this touches] | [`supabase-migrations`] |

## Dependencies

- [What must already exist: credentials, a migration, an earlier phase, an external subscription.]
- [Which phases depend on which — Phase 3 needs Phase 1's output.]

## File ownership

| File | Change |
|---|---|
| `packages/[…]` | [what changes there] |
| matching `*.test.ts` files | RED for each phase |

## Phase 1 — [the result of this phase, in one line]

Closes: 1, 2

### Depends on

[Orchestrated only: the phase and the output it produced. Delete for a Planned plan.]

### Contract

Given [state, with real values], when [action], then [observable result].
[One paragraph maximum. Name the fixture, row, id, or file each assertion runs against.]

### Files

| Task | File | Parallel |
|---|---|---|
| T1.1 | `packages/[…]/x.ts` | |
| T1.2 | `packages/[…]/y.ts` | [P] |

### RED

- [Test name] → [the exact failure expected, e.g. `recordConnection is not a function`]
- [Test name for the failure path] → [expected failure]

### Verification

```bash
[exact command]
```

Pass when [the readable result: N tests passing, exit 0, this row present].

## RED ledger

| Phase | RED written | RED observed failing | GREEN |
|---|---|---|---|
| 1 | [file (n tests)] | [filled in during execution] | [filled in during execution] |

## Verification

Closes: 3

- Each phase's own command above.
- `pnpm test:affected` across the diff.
- `pnpm build` clean.
- `verify` before handoff.
- [The one check that settles it in the real system, and where to look.]

## Deviations

[Filled in during execution. Every change to files, behaviour, or verification that this plan did
not say, with the reason.]

## Retirement

[Where the durable facts get promoted on completion — a README, a reference doc, a requirement.]

Open items, with owners:

1. [Anything unresolved that does not block execution.]
````

---

## Splitting phases into files

Only for Orchestrated work whose phases separate workers execute — one agent reading one file
gains nothing from the indirection. The parent keeps the phase heading and its dependencies, and
points at the child:

````markdown
## Phase 3 — [the result of this phase]

Closes: 4

### Depends on

- Phase 1: [the output it produced].

Phase file: `plans/[date]-[slug]/phase-3.md`
````

The child file holds the rest of the phase, and the schema validates it there:

````markdown
### Contract

Given [state], when [action], then [observable result].

### Files

| Task | File | Parallel |
|---|---|---|
| T3.1 | `packages/[…]/x.ts` | |

### RED

- [Test name] → [the exact failure expected]

### Verification

```bash
[exact command]
```
````

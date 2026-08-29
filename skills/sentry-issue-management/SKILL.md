---
name: sentry-issue-management
description: Pull every production Sentry issue from a time window, group them into worktree-sized units of work, and create a worktree for each. Owner-invoked.
disable-model-invocation: true
allowed-tools: Bash, Read, Grep, Glob
---

# Sentry Issue Management

TRIGGER: the owner selects this skill with a time window — `2 hours`, `3 days`, `6.5 hours`,
`30 minutes`. Reading what is currently erroring is `sentry-access`. Diagnosing issues the owner
names by id, changing nothing, is `investigate-sentry-issues`.

**This skill ends with worktrees on disk.** It reads production Sentry, groups what it finds into
units of work, prints one table, and then creates one worktree per row. It never writes a fix, a
test, a branch beyond the worktree's own, or a plan file. Selecting this skill **is** the owner's
current-turn permission to create those branches and worktrees.

## Step 1 — Turn the argument into a window

One argument, free text, always a duration. Convert it to hours, then pick the smallest Sentry
period that contains it and compute the exact cutoff:

| Window | Period to query |
|---|---|
| up to 1 hour | `1h` |
| up to 24 hours | `24h` |
| up to 7 days | `7d` |
| up to 14 days | `14d` |
| up to 30 days | `30d` |

```bash
HOURS=6.5   # the argument in hours
python3 -c "import datetime,sys; print((datetime.datetime.now(datetime.timezone.utc)-datetime.timedelta(hours=float(sys.argv[1]))).isoformat().replace('+00:00','Z'))" "$HOURS"
```

Over 30 days is not readable: query `30d`, and say in the report that the window was cut to 30
days. No argument at all is not a reason to guess — ask for the window and stop.

## Step 2 — Pull every production issue in the window

Production only, every level, through `sentry-access`. The period is coarser than the window, so
filter the rows by `lastSeen` against the cutoff from Step 1:

```bash
python3 scripts/agent-access/sentry_access.py list prod --period 24h --level all --limit 100 \
  | python3 -c "import json,sys; from datetime import datetime as d; c=d.fromisoformat(sys.argv[1].replace('Z','+00:00')); print(json.dumps([i for i in json.load(sys.stdin) if d.fromisoformat(i['lastSeen'].replace('Z','+00:00')) >= c], indent=2))" "$CUTOFF"
```

Two honest limits, both stated in the report when they bite:

- The helper reads **unresolved** issues only. An issue someone already resolved will not appear,
  and that is correct — it is not work.
- 100 is the ceiling. If the filtered list holds 100 rows, say the window was busier than the
  ceiling rather than implying it was complete.

If nothing comes back, say so, create nothing, and stop. An empty list is a real answer.

## Step 3 — Read each issue properly before grouping

Grouping guessed from titles is the failure mode of this skill: two issues with the same words are
routinely two different bugs, and two with nothing in common are routinely one. Follow
`investigate-sentry-issues` Steps 2 and 3 for every issue in the list — the latest event, a batch
of recent events, and the repository files the stack frames actually name.

Record for each issue: the files its frames touch, the release it started on, its event count, its
affected-user count, and the failing condition in one sentence.

## Step 4 — Group issues into units of work

Two issues belong in the same group when a single person, fixing one, would have the other open in
front of them. In practice that is one of:

- **Same root cause** — one break wearing several stack traces.
- **Same files** — different causes, but the fix lands in overlapping code.

Everything else is its own group. A group of one is normal and correct; padding groups to look
tidy hands one person two unrelated bugs, and splitting a shared cause across worktrees produces
two branches that conflict.

Never group on level, on error class, or on "these both look like timeouts".

## Step 5 — Name the worktrees

One name per group: kebab-case, at most four words, naming **the fix**, not the exception —
`class-roster-null-guard`, not `typeerror-undefined-roster`. Check the name is free before you
put it in the table:

```bash
git worktree list
git branch -a --format='%(refname:short)'
```

If a name is taken, adjust it; never plan to reuse an existing worktree.

## Step 6 — Print the table, once

Exactly these five columns, one row per group, highest priority first:

| Worktree | Sentry issues | What is going wrong | Who it hits | Priority |
|---|---|---|---|---|
| `class-roster-null-guard` | 6789, 6801 | Opening a class with no students crashes the page. | ~40 teachers, 300 times today, still happening. | P1 |

- **What is going wrong** — plain English, what a person tried to do and what happened instead. No
  file names, no exception types, no stack traces.
- **Who it hits** — people, then occurrences, then whether it is still happening. Keep affected
  users and event counts separate; they answer different questions. Say when the number is small.
- **Priority** — `P1` blocks a person from finishing something, or touches money, grades, or
  access. `P2` is visible and has a way around it. `P3` is noise, a retry that already recovered,
  or something no person ever saw.

Under the table put one line only: how many worktrees are about to be created, and that each one
is a cold build. Nothing else. This is not a plan and it is not asking for approval.

## Step 7 — Create the worktrees, in parallel

Select `new-worktree` once, with every name from the table, and follow its **Creating several at
once** section: the git steps run one after another, then every build starts together and the
call waits for all of them.

Do not create them one at a time. Builds are niced and never queue, so a row costs a share of
what is already running, and running six in sequence turns minutes into an hour for nothing.

If one worktree fails, keep the rest and name the failure in the report.

## Step 8 — Offer the Warp tabs

Write one Warp **tab config** per worktree — a TOML file that opens the dev box in that worktree
with the agent already running **on that row's Sentry ids**, its first prompt naming the
`investigate-sentry-issues` skill and those ids. Launch configurations are legacy and newer Warp
silently ignores them. Steps, the exact template and the PATH check the agent needs are in
`references/warp-tabs.md`;
follow it rather than writing the files from memory. Print the single command the owner runs on
their Mac and stop — opening the tabs is theirs, not yours.

## Step 9 — Report

The table from Step 6, then three lines at most: which worktrees exist, which failed if any, and
the Warp command. No summary of what you read, no per-issue detail, no fix suggestions. If the
owner wants one of these fixed, that is the next instruction and it routes through
`workflow-router` like any other change.

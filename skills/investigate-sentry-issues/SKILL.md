---
name: investigate-sentry-issues
description: Investigate one or more Sentry issues end to end and report what is failing, who it hurts, why it broke, why it started, and how confident a safe fix is. Read-only; changes no code.
allowed-tools: Bash, Read, Grep, Glob
---

# Investigate Sentry Issues

TRIGGER: a request names Sentry issues and asks what is going on — "investigate these Sentry
issues", "what is 6789 and 6801", a pasted Sentry link. Reading a list of what is currently
erroring is `sentry-access` and needs nothing from here.

**This skill ends at an answer.** No branch, no test, no fix, no plan file — even when the fix
looks obvious. If the owner wants it fixed, that is a separate instruction and it routes through
`workflow-router` like any other change.

## The argument

One argument, free-form. All of these are the same request:

| The owner types | Read as |
|---|---|
| `6789` | one production issue |
| `6789, 6801, 6842` | three production issues |
| a Sentry permalink | the numeric id inside it |
| `WILLOW-PRODUCTION-4KZ` | a short id — resolve it in Step 1 |
| `the two timeout errors from this morning in dev` | free text — resolve it in Step 1 |

Default to production. Only use `dev` when the argument says dev or staging. If the argument
names no issue you can resolve, say so and stop — never investigate a different issue than the
one asked about.

## Step 1 — Resolve the argument to numeric issue ids

Numeric ids need no resolution. Everything else does, and `sentry-access` owns the commands:
list the target's unresolved issues over a period wide enough to contain the described issue,
then match on short id, title, or the described symptom.

An issue that is resolved or ignored will not appear in a list of unresolved issues. Its numeric
id still reads fine, so an unmatched short id or description is a reason to widen the period and
then to say plainly that it is not currently unresolved — not a reason to guess.

**Never investigate an issue you resolved by guessing.** If free text matches two issues,
investigate both and say you did.

## Step 2 — Read each issue in full

Three reads, all through `sentry-access`, all required for every issue:

1. **The issue row**, from the list — title, culprit, level, event count, affected-user count,
   first seen, last seen, permalink. This is the only place impact and age come from.
2. **The latest event** — exception type and message, the repository stack frames, the release,
   the transaction, and any failure context the payload carries.
3. **A batch of recent events** — one event is an anecdote. Read a batch and ask what every
   occurrence shares and what varies: one release or several, one call path or many, one
   account or a spread. That contrast is what separates a cause from a coincidence, and reading
   only the latest event is the difference between this skill and a transcription.

Widen the list period (`1h` → `24h` → `7d` → `14d` → `30d`) until the issue appears. **First
seen is the whole of Step 5**, so an issue that never appears inside `30d` is one whose age you
cannot state — say that rather than implying it is new.

The helper deliberately omits user, request, breadcrumb, and third-party frames. That is a
boundary, not a gap to work around: never reach past it with a raw API call.

Record for each issue the project, pinned environment, period, and the time you queried, and keep
event count and affected-user count separate. They answer different questions and conflating them
is the most common error in this report.

## Step 3 — Read the code the stack trace names

An investigation that stops at the Sentry payload is a transcription, not an investigation. Open
every repository file the stack frames name, at the line they name, and read outward until you
can state the failing condition as a sentence: *what was true about the input or the state that
made this line throw*.

Then check the obvious neighbours — the caller, the value that was null, the request that timed
out, the migration behind the column that is missing. `debugging-patterns` holds the failure
shapes this codebase repeats; read it before concluding that a cause is novel.

**Prove the cause, do not narrate a plausible one.** If the evidence supports two causes, hold
both and say which evidence would separate them. A confident wrong cause is worse output than an
honest fork.

## Step 4 — Group issues that are one failure

Several issue ids are often one break wearing several stack traces — the same release, the same
call path, the same minute. When they are, say so once and report them as one failure with its
ids listed. Padding the report to one section per id hides the shape of the problem.

## Step 5 — Date it: why did this start?

First seen is the anchor. Compare it against what changed:

```bash
git log --since="<first-seen-date>" --until="<first-seen-date + 1 day>" --oneline
git show <release-from-the-event>          # when the release tag is a commit sha
```

Then name the trigger as one of:

- **a deploy** — a change landed at that time on that path; name what it changed in plain English
- **data or scale** — the code is unchanged and the input or volume is new
- **an external dependency** — a provider, quota, credential, or expiry moved
- **nothing; it did not just start** — it has been failing since first seen, and what changed is
  that someone looked. Say this outright when it is true. It is a common and useful answer.
- **undetermined** — the evidence does not date it, and here is what would

Never assert a culprit deploy you have not opened. A commit landing near the right time is a
candidate; a commit touching the failing path is a cause.

## Step 6 — Rate confidence in fixing it safely

Rate confidence in **fixing it safely and completely**, not in understanding it. They differ, and
the second one is what the owner is buying. One rating per failure:

| Rating | Means |
|---|---|
| **High** | The cause is proven, the change is contained, and existing tests cover the path. A fix now would be routine. |
| **Medium** | The cause is proven, but the fix touches shared behaviour, lacks test coverage, or has a plausible way to break something adjacent. Nameable, not yet safe. |
| **Low** | The cause is unproven, or the fix depends on a product decision, a data migration, or an external system. |

Every Medium or Low states **the one thing that would raise it** — a test to write, a payload to
read, an answer only the owner has. That sentence is the value of the rating.

Anything the fix would touch that is genuinely risky — production data, a migration, ui-kit, an
external write — is a rating input, and it is named in the report.

## Step 7 — Report

Plain English, written to a non-technical reader, following the answer contract in `AGENTS.md`.
No file paths, function names, or stack traces in the report — you read them so the owner does
not have to. Lead with the worst failure by user impact.

Per failure, five short answers under a one-line heading, and nothing else:

- **What is failing** — what a person tried to do and what happened instead. One sentence.
- **Who it hits** — how many people, how many times, and whether it is still happening. Say
  when the count is small; a loud issue affecting two people is not an emergency.
- **Why** — the cause, in a sentence a product lead can repeat.
- **Why now** — the trigger from Step 5, or that it did not just start.
- **Confidence** — the rating, and for Medium or Low the one thing that would raise it.

Close with a single line the owner can act on: which of these to fix first, or that none need
fixing. Do not attach a fix plan, a diff, or a list of options he could pick from — if the answer
is that one should be fixed, the next instruction will say so.

**No FYIs.** A finding that does not belong to one of the five answers is deleted, not appended.

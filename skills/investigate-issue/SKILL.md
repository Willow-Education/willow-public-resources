---
name: investigate-issue
description: Investigate a prefixed list of GitHub and Sentry issues as one read-only unit and report their shared or separate failures. Owner-invoked.
disable-model-invocation: true
allowed-tools: Bash, Read, Grep, Glob
---

# Investigate Issues

TRIGGER: the owner selects this skill with comma-separated `gh:<number>` and
`sentry:<numeric-id>` values, for example `gh:3101,sentry:6789`. Use `github-access` for a simple
GitHub read and `investigate-sentry-issues` for Sentry-only investigations that do not need a
combined report.

**This skill is read-only and ends at one report.** It never moves a card, writes a plan, changes
Sentry status, creates a branch or worktree, or fixes code. Do not invoke `triage-issue`: that
skill's valid endings can change state.

## Tool discipline

Never delegate repository exploration to an Explore agent. Use the host's `Glob`, `Grep`, and
`Read` tools directly with repository-rooted paths. Never chain `cd` with a repository search, and
never use `grep`, `find`, `rg`, or another shell search command for code discovery. Reserve shell
commands for the named GitHub and Sentry helpers, and batch independent reads and searches.

## 1. Parse the identifiers

Accept a comma-separated list containing only:

- `gh:<number>` for a numeric GitHub issue in `Willow-Education/willow-app`.
- `sentry:<numeric-id>` for a numeric production Sentry issue id.

Trim whitespace, de-duplicate, and keep the supplied order. Reject the whole request on an empty
item, non-numeric id, or unknown prefix such as `github:`; name the invalid value and investigate
nothing. Never guess a replacement issue.

## 2. Read every GitHub issue without triaging it

For each GitHub id, follow `github-access` and read the title, body, state, labels, assignees,
comments, URL, and project items. A missing or closed issue is still reported as supplied evidence,
but never silently replaced.

Images in the body or comments are evidence. Extract every GitHub attachment URL, download it to
`.agent-state/`, and inspect the image. Summarize for yourself what is broken, how to reproduce it,
and the expected behavior. Do not move the card and do not write a plan.

Read the repository path implicated by the report and its nearest callers and tests until the
failing condition is established. If the issue is a feature request rather than a failure, identify
the behavior it asks to change and the existing behavior that would be affected.

## 3. Read every Sentry issue fully

For each Sentry id, follow `investigate-sentry-issues` Steps 2, 3, 5, and 6. That requires the issue
row, latest event, recent-event batch, implicated repository code, start trigger, impact, and
confidence. Use `sentry-access` only; default to production and preserve its sanitization boundary.

Record the project, environment, period, query time, event count, and affected-user count. Never
conflate occurrences with people.

## 4. Decide whether the supplied issues are one failure

Group two ids only when the evidence proves the same root cause or the fix would touch the same
files and must be designed together. Shared wording, error level, exception class, timing, or the
word “timeout” is not proof. Read across GitHub reproduction details, Sentry event variation,
release timing, stack frames, and repository code.

If the evidence supports separate failures, preserve separate groups in the same report. If it
cannot distinguish two plausible relationships, state what evidence would settle it rather than
forcing a group.

## 5. Report

Lead with the failure that hurts a person most. For each proven group, list its exact prefixed ids
and give five short answers:

- **What is failing** — what a person tried and what happened instead.
- **Who it hits** — cost to one person, affected people, occurrences, and whether it continues.
- **Why** — the proven cause or the unresolved alternatives.
- **Why now** — deploy, data or scale, external dependency, longstanding failure, or undetermined.
- **Confidence** — High, Medium, or Low confidence in a safe complete fix; for Medium or Low, the
  one thing that would raise it.

Close with one line naming which group to fix first. Do not add a fix plan, implementation options,
file paths, stack traces, or state changes.

---
name: audit-agent-work
description: Fresh-context audit of work another agent completed on the current branch. Read-only and evidence-first by default.
---

# Audit Agent Work

Audit the current branch as an independent reviewer. **Read-only is the default.** The audit's
value is a fresh pair of eyes on the code and the plan — not a second execution of gates that
already ran.

**The auditor is never the agent that wrote the code, and never the agent that fixed it**
(owner, 2026-09-06). A cleared context is not independence — a blind spot shared by the plan and
the audit survives it. If you fixed anything on this branch, say so and hand the audit over.

## Why this skill is shaped this way (2026-08-23)

The previous version opened with "re-run every applicable gate", which cost a full
`pnpm test:affected` per audit and once put two of them on one machine at load average 92. It
also bought nothing: re-running a suite once does not detect flakiness, and the failure this
audit exists to catch is that the suite never ran or never reached the diff — both checkable
from a receipt in seconds. The measurements are in `references/why-evidence-first.md`.

So: consume the evidence, verify that it *binds* to the tree in front of you, and spend the saved
time reading code.

## 0. Preflight — is this worktree fit to be audited from?

Two checks, both cheap, both before anything else:

```bash
test -f scripts/agent-hooks/guard-test-concurrency.py \
  && grep -q guard-test-concurrency .claude/settings.json && echo GUARD-OK
grep -q deferred_global_inputs scripts/ci/affected_tests.py && echo SELECTION-OK
```

A worktree branched before a guard landed does not have the guard *or* its registration — the
hooks live in tracked files, so an older base silently opts out of every protection added since.
That is exactly how the 2026-08-23 collision happened.

- **`GUARD-OK` absent:** you have no machine-wide test lease here. **Do not launch a suite from
  this worktree at all**, in any mode. Report it as a finding and audit read-only.
- **`SELECTION-OK` absent:** local test selection still widens on a lockfile or package-manifest
  change, so any run you start may be the whole package. Same rule: do not launch one.

In both cases the remedy is for the owner to merge `main` into the worktree. Say so; do not do it
yourself — it is not your branch.

## 1. Reconstruct the intended work

- Find the trigger from the branch name, root `plans/` frontmatter, commit messages, or a plan's
  `source:` field. Use `github-access` or `sentry-access` to read any linked issue live; treat
  issue counts and status read earlier in the session as stale.
- Read the approved plan in `plans/`. If it was retired, inspect git history for the plan first.
  Only if history is insufficient, recover execution context from the host session logs:
  - Claude Code: `~/.claude/projects/<flattened-cwd>/*.jsonl`
  - Codex: `~/.codex/sessions/<YYYY>/<MM>/<DD>/*.jsonl`; match the worktree by the first-record
    `cwd`, then inspect message and patch payloads.
- Note claimed verifications, plan deviations, and the RED ledger. Treat each as a claim to check
  against evidence — not as a claim to re-execute.

## 2. Read the actual code

**This is where audits find things, and it costs nothing but attention. Spend the time here.**

- Inspect every changed, staged, and untracked file with `git status --short`, `git diff`, and
  `git diff --cached`.
- Read each touched file whole, including its unchanged interactions, error handling, and other
  callers. Find all callers of changed signatures, triggers, and re-export barrels.
- Check the wire contract: confirm each caller still sends what its consumer expects.
- Load the governing `.agents/rules/`, the closest package `AGENTS.md`, and relevant architecture,
  identity, tenancy, security, or requirements skills.

## 3. Use the feature, do not only read it

**Anything with a screen is audited by operating it** (owner, 2026-09-06). Start the app, sign in,
and do what the requirements describe: click the buttons, sort the table, follow the links.

Take each user-visible acceptance criterion and **observe it**, then record what you saw. "The
code calls the endpoint" is not an observation; "I pressed Analyze and themes appeared" is. A
criterion you could not exercise is a finding, not a pass. Judge the experience too — a sort that
redraws the page, a header that scrolls away, a link that goes nowhere.

This audit is one the owner asked for, not a link in the delivery chain: it never blocks a
handoff. The two audits that made this a rule are in
`references/independence-and-observation.md`.

## 4. Verify the evidence, do not rebuild it

### What a verification receipt must bind

A receipt is trustworthy only if it pins the tree it describes. `verify` link 1 writes
`.agent-state/verify-receipt.json`; **it is void** if its `head` or `worktreeDigest` differs from
the tree in front of you, if any file it covers has an mtime later than `finishedAt`, or if its
`versions` no longer match. A void receipt is treated exactly as a missing one. Never repair a
receipt; regenerate or work without it. Every field and why each is load-bearing:
`references/receipt-binding.md`.

### Reading the receipt

```bash
python3 scripts/ci/verify_receipt.py check
```

This runs nothing. It prints `RECEIPT VALID` or `RECEIPT VOID` with the reason, the number of
test files actually collected, and the coverage gap. Read its output as:

1. **`NO RECEIPT` / `RECEIPT VOID`** — treat both identically: there is no usable evidence. Go to
   *When evidence is missing* below.
2. **`gap: N`** — the plan selected something the run never collected. That gap is **the only
   thing you may run.**
3. **`plan widened`** — a finding in its own right: selection could not be scoped, so the green
   covers more than the diff and tells you less than it appears to.

### When evidence is missing, stale, or void

Compute what the diff needs, then run **only that** — never the lane's whole chain:

```bash
python3 scripts/ci/affected_tests.py --list    # prints the plan, runs nothing
pnpm exec vitest run <specific files>          # from the package directory
```

Widening past the gap is the failure this skill was rewritten to stop. If the gap is genuinely the
whole package, that is a finding to report — not a suite to launch. Hand it to the owner.

### Concurrency

Every suite, build, lint, or type-check you launch competes with every other agent on this
machine, and `scripts/agent-hooks/cpu_budget.mjs` is what keeps that survivable: each Vitest run
claims its workers from a machine-wide ledger of what other live runs hold. So launch when
you need to and **never wait for another agent's run to finish** — nothing queues, and the owner
runs twenty to thirty agents at once by design. Never background a run: a detached pool is
workers nobody reaps.

### verify-red

**Do not run `scripts/agent-hooks/verify-red.sh` as part of an audit.** It executes the changed test files twice — once
against a reverted implementation, once restored — and it is explicitly advisory: it never gates a
handoff. Consume the RED ledger the executing agent already produced. If no ledger exists, that
absence is the finding.

### What re-running never establishes

Re-running a green gate once tells you nothing about flakiness, environmental coupling, or
external state. If you suspect one of those, name the file and run **that file** repeatedly —
`pnpm exec vitest run <file> --repeat=5` — a targeted diagnostic with a real answer, and orders of
magnitude cheaper than the chain it replaces.

## 5. Judge and report

Assess the plan's acceptance criteria and these standing questions:

- **Completeness:** Does every create/edit, API/trigger, and caller path receive the fix?
- **Correctness:** Is the demonstrated root cause fixed without a workaround?
- **Regressions:** Which previously working behavior changes, especially new rejections caused by
  required fields or fail-closed permissions?
- **Security:** Does identity come from authentication rather than request data, and do logs avoid
  credentials and student data?
- **Technical debt:** Flag weakened types, `any`, optional parameters that should be required, dead
  code, or missing validation. Label touched pre-existing debt as residual, not introduced.
- **Tests:** Would the tests catch the original incident, spoofing/failure paths, and prove RED?

Work the plan's `Closes:` map rather than judging the criteria loosely: every acceptance criterion
names the phase that claimed it, and that phase names the command that proved it. A criterion no
phase claimed, or one whose evidence you could not bind to this tree, is a finding.

For a criterion a person can see, the evidence is what you saw in §3 — not the code that ought to
produce it.

Report the verdict first, then the gates whose evidence you verified, then residuals, then what
remains in delivery. **Name every gate you consumed rather than re-ran** — an audit that quietly
trusts a receipt reads identically to one that checked its binding, and only one of those is worth
anything. A Sentry-backed issue is not fixed until the release ships and a live re-read shows
events stopped. Never resolve a Sentry issue yourself.

## 6. Retire the plan, but only on a pass

`execute-plan` deliberately leaves the plan standing at `status: verifying` so this audit has its
primary input. Retirement is this skill's last step and only on a PASS: promote the durable facts
to the README, reference, or requirement that outlives the work, set `status: done`, then delete
the transient plan.

On a FAIL the plan stays exactly where it is, at `verifying`, with the findings reported. A plan
deleted over unfinished work leaves the next agent reconstructing intent from git history.

**Three failed audits on one branch is the ceiling** (owner, 2026-09-06). Report the findings and
hand the branch back rather than starting a fourth. A fifth round that finally passes is churn,
not convergence.

## `rerun` mode — owner-invoked only

Selecting this skill with `rerun` re-executes the lane's chain from scratch instead of consuming
evidence. It costs a full `test:affected` and is justified only when the owner has a concrete
reason to distrust the toolchain itself — a corrupted receipt, a suspected bad merge, a runner
change. **An agent never selects it on its own**, and it still respects the preflight in §0 and
the machine-wide lease.

The word `full` is deliberately not used here: it was retired from `commit` because it meant
the workspace suite, and reusing it would re-import that confusion.

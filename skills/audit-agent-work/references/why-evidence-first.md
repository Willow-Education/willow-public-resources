# Why this skill consumes evidence instead of re-running it (2026-08-23)

The previous version opened with "re-run every applicable gate". Following that instruction cost
a full `pnpm test:affected` per audit, and audits cluster — they are the last step of the plan
lifecycle, so several branches finishing together fire several suites together. On 2026-08-23 two
of them ran side by side on one laptop: 16 Vitest worker threads on 10 cores, load average 92,
and every other agent on the machine slowed to a crawl. One of the two had widened to the whole
frontend package — 1,444 files, 27.5 minutes — for a diff that touched a handful of them.

The re-run also did not buy what it was supposed to. Re-running a suite once does **not** detect
flakiness: a flaky suite passes or fails on the second run for the same reasons it did on the
first. Non-determinism argues for repeating one suspect file several times, never for repeating
everything once. And the failure this audit genuinely exists to catch is not "the suite lied" —
it is **"the suite never ran"** or **"the suite did not reach the diff"**, both of which are
checkable from a receipt in seconds.

So: consume the evidence, verify that it *binds* to the tree in front of you, and spend the saved
time reading code.

## What re-running never establishes

Re-running a green gate once tells you nothing about flakiness, environmental coupling, or
external state. If you have a specific reason to suspect one of those, name the file and run
**that file** repeatedly:

```bash
pnpm exec vitest run <file> --repeat=5
```

That is a targeted diagnostic with a real answer, and it is orders of magnitude cheaper than the
chain it replaces.

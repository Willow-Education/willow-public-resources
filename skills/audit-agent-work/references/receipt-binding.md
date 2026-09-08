# What a verification receipt must bind

A receipt is trustworthy only if it pins the tree it describes. `verify` link 1 writes `.agent-state/verify-receipt.json` through
`scripts/ci/verify_receipt.py`; a receipt missing any of these fields is not a receipt:

| Field | Why it is load-bearing |
|---|---|
| `head` | The commit the run covered |
| `base` | The merge base selection was computed against |
| `worktreeDigest` | Hash of tracked-but-dirty **and** untracked content — a green run on a tree that has since been edited proves nothing |
| `plan` | The `affected_tests.py` selection: packages, mode, and whether it widened |
| `collected` | The exact test files Vitest actually collected — the plan says what was asked for, this says what ran |
| `versions` | node, pnpm, vitest — a toolchain change invalidates the result |
| `startedAt` / `finishedAt` | Ordering against file mtimes |
| `command` and `exit` | Verbatim invocation and status per link |

**The receipt is void** if `head` or `worktreeDigest` differs from the tree you are looking at, if
any file it covers has an mtime later than `finishedAt`, or if `versions` no longer match. A void
receipt is treated exactly as a missing one. Never repair a receipt; regenerate or work without it.

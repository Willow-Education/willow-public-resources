# Why an auditor operates the feature, and never grades its own repairs

Both rules come from one afternoon, 2026-09-06, and one feature: the survey results page.

## The same model planned, fixed, and passed its own work

Two worktrees built the same feature from the same starting prompt, with the roles swapped. In
one, the model that wrote the plan also ran the audit and made every repair the audit asked for.
It failed its own branch four times and passed it on the fifth — with the feature's primary
action still doing nothing when a person pressed it.

Wiping the auditor's context between rounds did not help, and that is the part worth remembering.
The defect was not something the auditor had talked itself into; it was something the plan and
the audit shared a blind spot about, and a fresh context reproduces a shared blind spot exactly.
Independence has to come from a different author, not a cleared window.

The five rounds also looked like progress from inside. Each one found smaller issues than the
last, which reads as convergence and was actually churn around a defect none of the rounds was
looking for. Hence the ceiling: three failed audits on one branch, then hand it back.

## Reading code cannot see what using it shows

The other worktree's audit passed on the first try. Its build was the better one — but the owner
then found, in about five minutes with the page open, a table header that scrolled away, a
distribution chart with no way to see the count behind a bar, and a button with no pointer
cursor. Every one of those is invisible in a diff and obvious in a browser.

The build that failed four audits had a requirement — link each response to the student's profile
— that was simply not met. An auditor reading the code found the component that renders responses
and moved on. Nobody clicked a response.

So the evidence for a user-visible criterion is an observation, phrased as one: not "the handler
calls the endpoint" but "I pressed Analyze and themes appeared". A criterion that cannot be
exercised is a finding, not a pass, and how the thing feels to use — a sort that redraws the whole
page, a breakdown that costs a round trip to another screen — is part of the verdict.

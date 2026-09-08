---
name: execute-plan
description: Execute or resume an approved plan from root plans/. Planned and Orchestrated work only.
---

# Execute Plan

Accept a root `plans/<date>-<slug>.md`, authored through `create-plan`. Validate it first, at
the same bar it was written to:

```bash
python3 scripts/agent-config/plan_schema.py --authoring plans/<date>-<slug>.md
```

`--authoring` is the strict mode: numbered criteria with no unfalsifiable wording, a failure case
in every phase, real file paths, a runnable command behind every verification, the resources
linked, observability stated, and every criterion claimed by a phase through its `Closes:` line.
Run it while the plan is still `status: planning`, before the flip below.

**A plan that cannot pass it is not ready to execute.** Report what it names and stop — an hour
of implementation against criteria nobody can check is worse than the minute this takes. A plan
that came from `create-plan` passes it by construction; the ones that fail came from somewhere
else.

Do not execute a plan still awaiting approval. Once approved, change `status: planning` to
`executing` and run to completion without phase-boundary permission checks.

## Planned

The main agent owns implementation. For each ordered phase: confirm dependencies, load `tdd`,
produce legitimate RED, implement minimum GREEN, run phase verification, and update the RED ledger
and checklist. Record any deviation that changes files, behavior, or verification. After all phases,
run the plan's own verification chain against the whole acceptance contract.

Do NOT dispatch a reviewer agent and do not treat one as a required step — the owner audits when they
choose to.

## Orchestrated

**Before dispatching anything, name the independent threads.** Orchestration buys wall-clock time
and nothing else; if you cannot name which phases genuinely run at the same time, this is Planned
work and one agent is faster than the round trips.

A phase may carry `Phase file: `plans/<slug>/phase-N.md`` instead of its contract inline; read
that file for the phase and leave the rest unread. The orchestrator integrates; `phase-worker`
agents implement only assigned phases. Each dispatch must include the plan path, phase, completed
dependency outputs, exact file ownership, acceptance criteria, expected RED, and verification
command.

**Never accept a worker's report as evidence.** A subagent marking its own output as passing
without having tested it is the most common way this fails. Run the phase's own verification
command yourself before accepting the phase — one command, and it is the difference between
catching a false green now and at integration.

**Verify each phase as it lands, not all of them at the end.** A barrier makes every fast phase
wait for the slowest; a phase that has landed and passed is done.

**Parallel writers are allowed for disjoint files, and only when the plan pins the conventions
they would each otherwise invent** — the error shape, the naming, the file each imitates, which
`## Existing patterns` exists to fix. Two workers touching no shared file still ship two different
error conventions into one feature otherwise.

**A worker that reports the map was wrong stops the siblings it affects.** When a phase discovers
that a file, dependency, or regression it was handed does not match reality, re-brief every
in-flight phase whose contract that changes before letting it finish, and record the deviation.

Integrate results single-threaded, then re-run cross-phase verification.

## Direct

Direct work has no plan and must return to `workflow-router`: RED → GREEN → proportional verify.
Never create a plan merely to invoke this skill.

## Finish, and leave the plan standing

Once verification passes, fill the RED ledger, record every deviation, close out against the
`Closes:` map — each acceptance criterion, the phase that claimed it, and the command that proved
it — and set `status: verifying`.

**Do not set `done`, and do not delete the plan.** The plan is the audit's primary input, and
`audit-agent-work` runs after this skill: an executed plan that deletes itself forces the next
reader to reconstruct intent from git history or session logs, which is exactly the evidence an
independent audit must not have to guess at. Retirement — promoting durable facts, setting
`done`, deleting the file — belongs to the audit that passed it.

Never merge, deploy, write production data, or change production configuration unless the
separate owning workflow and current-turn authorization explicitly permit it.

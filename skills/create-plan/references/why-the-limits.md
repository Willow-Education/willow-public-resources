# Why the limits are where they are

Read this before changing a number in `create-plan` or `plan_schema.py`. Do not read it to write
a plan — the operative rules are in SKILL.md and this adds nothing to them.

## The mechanism is distraction, not length

Chroma evaluated 18 frontier models on increasing input length and found degradation **well below
the context limit** on tasks as simple as retrieval and replication. What drives it is not raw
size: it is **distractors** — topically related content that is not the answer — and each
additional distractor compounds the effect. Two findings matter most here:

- Performance falls as the semantic distance between the question and the answer narrows against
  the surrounding text. On-topic prose is the worst neighbour an instruction can have.
- Models scored **better on shuffled haystacks than on logically coherent ones**. Structural
  coherence hurt. A well-written page of argument about the same feature is therefore not a
  neutral cost — it is the strongest form of the interference being measured.

A plan's argument section is exactly that: same nouns, same feature, none of it the instruction.
That is why the cap lands on the context half and never on the phases.

## Why 250, and why a note at 150

Measured across the 58 phased plans in `plans/` on 2026-08-21: median context half 129 lines,
p75 249, p90 356, max 1,056. A cap at 250 fails only the longest quarter, every one of which was
argument-inflated, and still leaves room for a real evidence table. 150 is where a look is worth
it, so it prints a note instead of blocking.

Weak local signal, stated honestly: plans with a context half over 150 lines average 1.99
deviations per phase against 1.37 for the rest, but the correlation is 0.20-0.25 with n=58 and is
confounded by complexity. It shows the direction is not contradicted. It does not carry the rule.

## Why phases are never capped

A cap an agent can only satisfy by deleting something will get satisfied by deleting something,
and in a phase the deletable material is the concrete values, error paths, and file paths that
make the work checkable. A shorter document and a worse build.

## Why splitting is conditional

Progressive disclosure only pays when **separate agents each load one part**. At single-document
scale the same study found routing overhead with no benefit, and deeper hierarchies matched or
underperformed flat ones — "depth does not pay, and can hurt". So phase files are for Orchestrated
work whose phases separate workers execute, and never a default.

## Sources

- Chroma, *Context Rot: How Increasing Input Tokens Impacts LLM Performance* —
  https://www.trychroma.com/research/context-rot
- Anthropic, *Effective context engineering for AI agents* —
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- *Is Progressive Disclosure All You Need for Long-Context Agents?* —
  https://arxiv.org/html/2607.17598v1
- O'Reilly via Stack Overflow, *The right amount of spec for agentic development* (2026-08-21) —
  https://stackoverflow.blog/2026/08/21/dispatches-from-o-reilly-the-right-amount-of-spec-for-agentic-development/

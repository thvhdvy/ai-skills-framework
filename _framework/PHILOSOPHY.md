# Skill Framework Philosophy

## What a Skill Is

A skill is a **context module** — not a library, not a plugin, not a prompt template.

Its medium is Claude's attention. Its compiler is token probability. Its runtime is a single
conversation turn. Every design decision in this framework flows from that constraint.

A skill has three jobs, in priority order:

1. **Fire when it should.** A skill that doesn't trigger is worthless regardless of its
   internal quality.
2. **Cost as little context as possible.** Every token the skill consumes displaces user
   content, conversation history, and tool output. Lean by default.
3. **Behave consistently.** When a skill fires, Claude's output should be predictable across
   invocations with similar inputs.

Everything else — documentation, examples, templates, checklists — serves these three jobs
or it doesn't belong in the skill.

---

## Four Governing Principles

Listed in priority order. When principles conflict, the higher one wins.

### P1 — Trigger Fidelity

The description field is the sole mechanism by which Claude decides to invoke a skill.
A skill with a weak, vague, or missing description cannot be used. No other quality dimension
compensates for failure here.

Trigger fidelity has three axes:
- **Appropriate activation** — fires on the core use cases
- **No false positives** — stays silent on near-miss and adjacent-domain queries
- **No false negatives** — fires on implicit phrasings where the user clearly needs the skill
  but does not name it directly

Trigger fidelity must be measured empirically, not assumed. A description that seems
precise to its author may over- or under-trigger in ways that only become visible through
testing with realistic prompts.

### P2 — Context Economy

The context window is finite and shared. Skills that bloat the context displace signal.

Every instruction added to a skill creates permanent per-invocation token debt. Instructions
that are rarely needed accumulate debt against the common case. The correct response to a
rarely-needed instruction is not to include it anyway — it is to move it to a reference file
loaded only when the condition is met, or to trust Claude's base reasoning.

Lean by default. Every token must justify its presence on every invocation.

### P3 — Behavioral Predictability

When a skill fires, Claude's behavior should be consistent across invocations with similar
inputs. This requires:
- Unambiguous instructions (no room for conflicting interpretations)
- Specified output formats when output is structured
- Explicit handling of edge cases rather than delegation to inference
- No contradictions between modules

Unpredictable behavior is worse than no skill. A skill that produces inconsistent output
trains users to distrust it. A skill that fails silently (due to a broken reference or
contradictory instruction) is harder to debug than a missing skill.

### P4 — Maintainability

Skills are living artifacts. They will be updated as Claude's capabilities evolve, as
domains change, and as production use reveals failure modes.

Good architecture makes individual skill updates cheap and safe:
- Small blast radius — changing one file should not require changing others
- Single source of truth — each piece of knowledge lives in exactly one place
- Explicit dependencies — nothing is assumed; everything referenced exists

---

## What a Skill Is Not

**Not a system prompt.** Skills do not configure Claude's global behavior. They scope
Claude's behavior within a specific task context. Once the task is complete, the skill's
influence ends.

**Not a prompt library.** Prompt libraries store reusable text fragments. Skills store
structured workflows, decision logic, and domain knowledge — with explicit conditions for
when each piece of that knowledge should be loaded.

**Not a software package.** There are no imports, no type system, no runtime. The
"execution environment" is Claude's attention. Design accordingly: prefer clarity over
cleverness, explicit over implicit, and explanation over enumeration.

**Not a static document.** A skill that never changes is a skill that has never been used
in production. Maintainability is a first-class property, not an afterthought.

---

## The Token Debt Model

Every instruction in a skill's body costs tokens on every invocation, regardless of whether
that instruction is relevant to the current task. This is **token debt** — a permanent,
recurring cost paid for every instruction that exists in Tier 2.

Token debt is acceptable when the instruction earns its cost:
- It prevents a common failure mode
- It shapes a high-variance output into a consistent one
- It encodes non-obvious domain knowledge Claude cannot reliably infer

Token debt is not acceptable when:
- The instruction covers a rare edge case (move to references/)
- The instruction restates something Claude can reason about correctly without it
- The instruction exists because of one observed failure that has not recurred
- The instruction duplicates content available in a shared module

Periodically audit skills for accumulated token debt. An instruction that was justified at
authoring time may no longer be justified after Claude's base capabilities improve.

---

## The Presence-Necessity Principle

**Presence is never scored higher than necessity.**

A skill with no examples scores full marks on the Examples criterion if no examples are
warranted. A skill with no templates scores full marks on the Templates criterion if the
output is not structured enough to benefit from one. A self-review checklist that contains
five targeted items scores higher than one that contains twenty generic items.

This principle exists to prevent authors from padding skills to maximize evaluation scores.
Padding increases token cost without increasing value. The rubric penalizes padding
explicitly through the Context Economy criterion.

When in doubt about whether to include something: ask whether its absence would cause a
plausible invocation to fail or produce meaningfully worse output. If the answer is no,
omit it.

---

## Design Tensions

These tensions are real and will recur. The framework resolves them as follows:

**Trigger coverage vs. trigger precision:** A description broad enough to catch all true
positives will catch some false positives. A description narrow enough to eliminate false
positives will miss some true positives. Optimize for no false negatives first (missing a
true invocation is the worse failure), then reduce false positives with boundary statements.

**Documentation depth vs. context economy:** Explanation costs tokens. The framework
resolves this in favor of *why* over *what* when forced to choose — a single sentence
explaining the reason enables Claude to generalize correctly, while a rule without reasoning
only covers the stated case.

**Completeness vs. maintainability:** A skill that tries to cover every case becomes a
monolith that is expensive to update. Prefer coverage of the 80% case with explicit
documentation of known gaps over comprehensive coverage that makes the skill brittle.

**Examples as value vs. examples as padding:** Examples are useful when the skill's
behavior cannot be adequately described by instruction alone. They are padding when they
illustrate cases already covered unambiguously by the instructions. The distinction requires
judgment — the Presence-Necessity Principle is the deciding criterion.

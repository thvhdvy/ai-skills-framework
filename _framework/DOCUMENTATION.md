# Documentation Style Guide

## Scope

This document owns: writing voice, structural conventions, the why-over-what principle,
formatting rules, and documentation anti-patterns. For what each file is responsible for
documenting, see `ARCHITECTURE.md`. For how token cost affects documentation decisions,
see `KNOWLEDGE.md`.

---

## Governing Principle

Documentation in a skill is not written for humans reading a repository. It is written
for Claude processing an instruction set under token constraints. These are different
audiences with different needs.

A human reader benefits from context, narrative, and examples that build understanding
progressively. Claude benefits from density, specificity, and structure that minimizes
orientation cost and maximizes instruction clarity on first pass.

Every style decision in this document serves the second audience without completely
abandoning the first.

---

## Voice

### Person and Register

Skills are written in the **second person imperative** directed at Claude.

```
# Correct
Extract the table, then validate row count against the source.

# Wrong — third person
Claude should extract the table and then validate row count.

# Wrong — first person
I will extract the table and validate row count.

# Wrong — user-directed
Ask the user to provide the source document.
```

The imperative register signals that instructions are unambiguous directives, not
suggestions. Claude's response to imperative instructions is more consistent than its
response to descriptive or conditional framing.

### Formality

Maintain a technical register throughout. No conversational hedging ("you might want
to"), no apologetic qualifications ("this is a bit complex"), no narrative preamble
("first, let's understand the context").

State what to do. If the why matters, state it in one clause. Move on.

---

## The Why-Over-What Principle

When forced to choose between stating a rule and explaining the reasoning behind it,
prefer the reasoning.

A rule covers the case it was written for. The reasoning covers that case plus every
analogous case Claude can generalize from it. This is not a token-efficiency argument —
an explanation often costs more tokens than the rule it replaces. It is an
instruction-quality argument: a rule without reasoning produces brittle behavior at
the edges.

```
# Rule only — covers one case
Always include a summary section at the top of every report.

# Reasoning — covers all cases the author intended and more
Open with a summary. Stakeholders frequently read only the first section;
front-loading the conclusion ensures value even when they stop reading early.
```

The second version handles "make this a quick informal summary," "the client only
wants the recommendation," and "can you skip the usual structure" correctly. The first
version fails all three by triggering its ALWAYS constraint.

### When Rules Are Appropriate

Reserve unqualified directives (`always`, `never`, `must`) for true invariants — cases
where no context justifies deviation and where the cost of deviation is high enough that
Claude should not exercise judgment.

Before writing `always` or `never`, ask: is there genuinely no context in which this
rule should bend? If the answer is uncertain, replace the directive with reasoning.

---

## Structure Conventions

### Heading Levels

| Level | Use | Example |
|---|---|---|
| `#` | Document title only | `# Skill Architecture` |
| `##` | Major sections | `## Extension Patterns` |
| `###` | Sub-sections with 3+ distinct items | `### Pattern 1 — Wrapping` |
| `####` | Prohibited in SKILL.md | Use decomposition instead |

Four-level heading depth in SKILL.md is a signal that the skill is doing too much.
Apply decomposition criteria from `KNOWLEDGE.md` before adding a fourth level.

Reference files may use `####` sparingly when documenting deeply structured domains
where the hierarchy genuinely aids navigation.

### Formatting by Content Type

| Content type | Format | Rationale |
|---|---|---|
| Option comparison | Table | Relationships visible at a glance; lower token cost than prose |
| Sequential steps | Numbered list | Order is semantically significant |
| Unordered enumeration | Bullet list | Only for items with no prose dependency between them |
| Technical operations | Code block | Unambiguous; prevents variation across invocations |
| Output structure | Code block with placeholders | Constrains output space; more efficient than examples |
| Single key term | **Bold** | One instance per sentence; never decorative |
| Cross-reference | Backtick path | `` `ARCHITECTURE.md` ``, `` `references/aws.md` `` |

Bullet lists are not a default format. Prose with clear sentence structure is often
more token-efficient and conveys dependency between ideas that bullets obscure.

### Section Order in SKILL.md

Sections should appear in the order Claude needs them during a typical invocation:

1. **Orientation** — what this skill does and any immediate constraints
2. **Input handling** — how to interpret what the user provided
3. **Workflow** — the steps to execute
4. **Output format** — the structure of the result
5. **Conditional loading** — which reference files to load and when
6. **Edge cases** — only cases Claude cannot handle correctly by inference
7. **Self-review checklist** — final verification before responding

Sections that appear after Claude needs them create backtracking. An output format
section at the bottom of SKILL.md means Claude may have already started generating
output before reading the constraints on that output.

---

## Cross-Referencing

Reference earlier framework documents and sibling files rather than restating their
content. A cross-reference costs one line. A restatement costs that plus the risk
of diverging when the source is updated.

### Cross-Reference Syntax

```markdown
# Reference to another framework document
See `ARCHITECTURE.md` for extension patterns.

# Reference to a file within the same skill
If deploying to AWS, read `references/aws.md` before proceeding.

# Reference to a shared module
Output schema is defined in `shared/report-schema.md`.
```

Cross-references must be specific. "See the framework documentation" is not a
cross-reference — it is a delegation without a destination.

---

## Anti-Patterns

These patterns are prohibited. Each degrades either instruction clarity or token
efficiency in ways that compound across invocations.

| Anti-pattern | Problem | Correction |
|---|---|---|
| Preamble before instructions | Delays useful content; wastes Tier 2 budget | Start with the first directive |
| Restating what the user said | Claude already has the user message in context | Remove entirely |
| "As mentioned above" | Creates positional dependency; fragile under edits | State the thing again in one clause, or cross-reference by section name |
| Hedging language | Introduces ambiguity where there should be none | Remove: "you might", "consider", "it may be worth" |
| Passive voice throughout | Reduces instruction clarity; under-specifies the actor | Rewrite in active imperative |
| Numbered list for non-sequential items | Implies order where none exists | Use bullet list or table |
| Bullet list for items with prose dependency | Obscures logical flow between items | Rewrite as prose |
| Bold used decoratively | Dilutes signal value of bold | Reserve for the single most important term in a sentence |
| ALWAYS / NEVER without justification | Brittle at edges; breaks on valid exceptions | Replace with explained reasoning |
| Inline examples when a template suffices | Higher token cost for lower precision | Replace with parameterized template |
| Section headers for single-item sections | Structural overhead with no navigation value | Fold content into the parent section |

---

## KNOWN_ISSUES.md Style

`KNOWN_ISSUES.md` documents failure modes, not aspirational improvements. Each entry
must include three things:

```markdown
## [Short failure description]

**Trigger condition:** The specific input or context that causes this failure.
**Observed behavior:** What Claude actually does.
**Workaround:** What the user or caller can do to avoid or recover from the failure.
```

Do not document issues without workarounds unless the failure is severe enough to
warrant a lifecycle regression. An undocumented workaround is not a workaround.

---

## CHANGELOG.md Style

See `LIFECYCLE.md` for the CHANGELOG format. Style conventions that apply here:

- Entries are written in the past tense: "Added", "Changed", "Removed", "Fixed"
- Breaking changes are listed first within a version entry
- Each line describes one change, not a category of changes
- Do not document patch changes unless they fix a known issue

---

## Documentation Completeness by Lifecycle State

The documentation standard rises with lifecycle state.

| State | Minimum documentation standard |
|---|---|
| DRAFT | SKILL.md body exists and is readable |
| REVIEW | Cold-read test passes; no undefined terms; no broken cross-references |
| TESTED | `KNOWN_ISSUES.md` written; all cross-references verified to exist |
| VALIDATED | All sections follow this style guide; `CHANGELOG.md` exists if applicable |

Documentation that meets VALIDATED standard at DRAFT state is not penalized. These are
floors, not ceilings.

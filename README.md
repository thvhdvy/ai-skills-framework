# Claude Skills Framework

**Version 1.0**

A production framework for designing, building, reviewing, and maintaining Claude Skills —
modular instruction sets that extend Claude's behavior within specific task domains.

---

## What This Repository Is

This repository defines and enforces the standards every Claude Skill in this collection
must meet. It answers three questions:

- **What is a skill?** A context module — not a prompt template, not a chatbot persona —
  that fires when Claude recognizes it is relevant and costs tokens only when loaded.
- **How should a skill be built?** With a clear activation description, a three-tier
  knowledge structure, a documented lifecycle, and an explicit extension path.
- **How do we know a skill is ready?** Through a 100-point rubric, eight hard-fail
  conditions, and a lifecycle gate that distinguishes tested from validated.

The framework itself lives in `_framework/`. Every skill in this repository is governed
by it.

---

## Repository Structure

```
claude-skills-framework/
│
├── _framework/          # The framework. Read this before writing any skill.
│   ├── INDEX.md         # Entry point and navigation guide
│   ├── PHILOSOPHY.md    # Core principles and governing constraints
│   ├── LIFECYCLE.md     # Skill states, promotion gates, versioning, frontmatter schema
│   ├── CONVENTIONS.md   # Repository structure, naming, dependency syntax
│   ├── ACTIVATION.md    # Trigger description engineering and eval design
│   ├── KNOWLEDGE.md     # Three-tier loading model and token optimization
│   ├── ARCHITECTURE.md  # Extension patterns, conflict resolution, dependency governance
│   ├── DOCUMENTATION.md # Writing voice, structure conventions, anti-patterns
│   ├── QUALITY.md       # Evaluation rubric, hard fails, review workflow
│   ├── TEMPLATES.md     # Canonical file templates for every skill file type
│   └── EXTENSION.md     # Wrapper authoring, contribution, fork governance, migration
│
├── public/              # General-purpose utility skills. Triggerable by Claude.
├── shared/              # Dependency modules. Not triggerable. No description field.
├── plugins/             # Workflow and slash-command oriented skills.
├── org/                 # Organization-specific skills. Never submitted upstream.
├── examples/            # Reference implementations. Read-only for skill authors.
└── _deprecated/         # Retired skills. Preserved, never deleted.
```

**Key distinction:** `public/` contains skills Claude can invoke directly. `shared/`
contains knowledge modules that other skills reference as dependencies — they have no
trigger description and never appear in Claude's `available_skills` list.

---

## Reading Order

Read the framework documents in this order the first time. After that, consult
individual documents by responsibility.

1. `_framework/PHILOSOPHY.md` — understand the constraints before designing
2. `_framework/LIFECYCLE.md` — understand how skills mature and what each state means
3. `_framework/CONVENTIONS.md` — structure, naming, and dependency rules
4. `_framework/ACTIVATION.md` — how Claude decides to invoke a skill
5. `_framework/KNOWLEDGE.md` — how to organize content across three loading tiers
6. `_framework/ARCHITECTURE.md` — how skills relate to each other
7. `_framework/DOCUMENTATION.md` — how to write skill content
8. `_framework/QUALITY.md` — how skills are evaluated
9. `_framework/TEMPLATES.md` — starting templates for every file type
10. `_framework/EXTENSION.md` — how to extend existing skills

---

## Creating a New Skill

### 1. Choose a location and name

| Skill type | Location | Naming pattern |
|---|---|---|
| General-purpose utility | `public/` | `kebab-case` |
| Workflow / slash-command | `plugins/` | `domain--skill-name` |
| Organization-specific | `org/` | `org--topic--skill` |

See `_framework/CONVENTIONS.md` for the full naming specification.

### 2. Scaffold the directory

```
mkdir public/your-skill-name
```

Copy the SKILL.md template from `_framework/TEMPLATES.md` into your new directory.
Add only the subdirectories your skill actually needs — `references/`, `scripts/`,
`templates/`, `checklists/`, `examples/`. Do not create empty directories.

### 3. Write the trigger description

The `description` frontmatter field is the sole mechanism by which Claude decides to
invoke your skill. Write it following the four-component anatomy in
`_framework/ACTIVATION.md`:

1. Primary action statement (verb-first, one sentence)
2. Trigger inventory (concrete phrases, file types, contexts)
3. Coverage expansion (implicit cases the user won't name directly)
4. Boundary statement (what this skill does not cover — only when overlap exists)

### 4. Assign content to tiers

Every piece of content belongs to exactly one loading tier:

- **Tier 1** (description only): ≤ 150 tokens. Activation only.
- **Tier 2** (SKILL.md body): needed for the majority of invocations.
- **Tier 3** (references/, scripts/, etc.): everything else, loaded conditionally.

See `_framework/KNOWLEDGE.md` for placement rules and the loading condition syntax.

### 5. Set lifecycle state

Start at `lifecycle: draft`. Advance through `review → tested → validated` by
meeting the promotion gates in `_framework/LIFECYCLE.md`. Do not mark a skill
`validated` until real invocations have been observed and trigger accuracy confirmed.

### 6. Run the quality checklist

Before requesting peer review, self-assess against `_framework/QUALITY.md`. Check
for hard fails first — any hard fail scores the skill at 0 regardless of other quality.

---

## Reviewing a Skill

### Hard fails first

Check all eight hard-fail conditions before scoring. A hard fail ends the review.

| ID | Condition |
|---|---|
| HF-01 | Missing SKILL.md |
| HF-02 | Missing or ambiguous activation criteria |
| HF-03 | References to non-existent files or modules |
| HF-04 | Contradictory instructions between modules |
| HF-05 | Duplicate knowledge that should be shared |
| HF-06 | Monolithic design without extension path |
| HF-07 | Circular dependency |
| HF-08 | Stale lifecycle status |

Full detection and fix guidance for each is in `_framework/QUALITY.md`.

### Apply the rubric

| Criterion | Points |
|---|---|
| Trigger Fidelity | 20 |
| Context Economy | 20 |
| Architecture and Modularity | 15 |
| Documentation Quality | 10 |
| Examples and Templates | 10 |
| Maintainability | 10 |
| Naming and Consistency | 5 |
| Workflow Design | 5 |
| Self-Review Capability | 5 |
| **Total** | **100** |

Score interpretation: **90–100** → VALIDATED / **75–89** → TESTED /
**60–74** → REVIEW / **< 60** → DRAFT

### Cold-read test

Before scoring, read SKILL.md without context. Can you state the skill's purpose,
primary use case, and output format within 60 seconds? If not, the Documentation
Quality criterion is failing regardless of other scores.

---

## Extending an Existing Skill

Three patterns are available, in order of preference:

**Wrapper (preferred):** Create a new skill with a strictly more specific trigger
description. The wrapper adds organization-specific or domain-specific behavior and
defers to the base skill's pattern for the core task. The base skill is not modified.

**Contribution:** Propose a general-purpose improvement to a shared or public skill.
Appropriate for bug fixes, coverage gaps, and new reference variants. Not appropriate
for org-specific content.

**Fork (last resort):** Copy a validated skill and modify freely. Accept full
maintenance responsibility. Forks do not inherit improvements from the original.

For detailed authoring guidance on each pattern, including migration procedures when
a base skill changes, see `_framework/EXTENSION.md`.

---

## Framework Version

This repository uses framework version **1.0**. Skills record which framework version
they were written against in the `framework-version` frontmatter field. When the
framework is updated, `_framework/INDEX.md` documents what changed and which skills
require review.

---

## Contributing

Contributions to the framework itself follow the same review process as skill
contributions — peer review, quality checklist, lifecycle gates. Framework changes
that affect naming conventions, file structure, or the evaluation rubric are breaking
changes and require a major version increment.

Contributions of new skills to `public/` or `plugins/` must reach at least TESTED
state before merging. VALIDATED is required for skills intended as base skills for
wrapping by others.

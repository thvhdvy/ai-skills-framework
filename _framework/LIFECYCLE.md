# Skill Lifecycle

## Overview

Every skill exists in exactly one lifecycle state at any point in time. State is the
authoritative signal for whether a skill is safe to use, how much trust to place in its
output, and what kind of review is required before changes are accepted.

State is declared in the skill's frontmatter and is the skill owner's responsibility to
keep accurate. A stale lifecycle state is a hard fail.

---

## States

```
DRAFT ──► REVIEW ──► TESTED ──► VALIDATED
                        ▲            │
                        └────────────┘
                       (regression on breaking change)

Any state ──► DEPRECATED
```

### DRAFT

The skill is being written. No eval suite exists. The trigger description may not be final.

- **Safe to use:** No. Do not invoke in production.
- **Peer review required:** No.
- **Eval required:** No.

A skill should not remain in DRAFT for more than one working session. If work is
interrupted, the state is still DRAFT — do not promote prematurely.

---

### REVIEW

The skill has a complete SKILL.md, a plausible trigger description, and has been manually
tested against 2–5 realistic prompts with qualitative human review of outputs.

- **Safe to use:** No. For evaluation and feedback only.
- **Peer review required:** Yes — at minimum one engineer who did not write the skill must
  be able to understand its purpose and usage within 60 seconds of reading SKILL.md cold.
- **Eval required:** Yes — 2–5 prompts, qualitative review only.

The cold-read criterion is not optional. A skill that requires its author to explain it
before a peer can understand it has failed a basic comprehension test.

---

### TESTED

The skill has a formal eval suite of ≥ 10 prompts covering:
- Happy path (core use cases)
- Edge cases (boundary inputs, unusual but valid requests)
- Near-miss non-triggers (queries that share vocabulary but should not invoke this skill)

Trigger accuracy has been measured. Known failure modes are documented in `KNOWN_ISSUES.md`.

- **Safe to use:** Yes, with awareness of documented known issues.
- **Peer review required:** Yes, for any change that modifies trigger description or output
  format.
- **Eval required:** Yes — full suite must pass before promotion.

A skill can be promoted from TESTED to VALIDATED only after real invocations (not eval
runs alone) have been observed. Evals test what you anticipated; production reveals what
you did not.

---

### VALIDATED

The skill has been used in real invocations. Trigger accuracy is confirmed. Output quality
is consistent across varied inputs. The trigger description has been through at least one
optimization pass.

- **Safe to use:** Yes. This is the only state appropriate for shared or public skills.
- **Peer review required:** Yes, for any breaking change.
- **Eval required:** Yes, on any change — full suite must continue to pass.

VALIDATED is the steady state. A skill should remain here indefinitely with periodic
maintenance. It does not expire.

---

### DEPRECATED

The skill is superseded, its domain is covered by a better skill, or its underlying
capability has been absorbed into Claude's base behavior.

- **Safe to use:** No. Deprecated skills must not be invoked intentionally.
- **Deletion:** Skills are never deleted from DEPRECATED state. They are preserved in the
  `_deprecated/` directory with a `SUPERSEDED_BY.md` file.
- **Grace period:** A minimum 30-day grace period applies before archiving, to allow
  dependent wrapper skills to migrate.

The reason for preservation: production logs and wrapper skills may reference a deprecated
skill by name. The archive provides a reference point for understanding past behavior and
diagnosing failures in systems that depended on it.

---

## Transitions

### Promotion (forward)

| From | To | Gate |
|---|---|---|
| DRAFT | REVIEW | SKILL.md complete; 2–5 prompts manually tested; cold-read passed |
| REVIEW | TESTED | Eval suite ≥ 10 prompts; trigger accuracy measured; `KNOWN_ISSUES.md` written |
| TESTED | VALIDATED | Real invocations observed; trigger description optimized; no open critical issues |
| Any | DEPRECATED | Owner decision; `SUPERSEDED_BY.md` written; 30-day grace period started |

### Regression (backward)

A breaking change to a VALIDATED or TESTED skill triggers regression to REVIEW, not to
DRAFT. The distinction matters: the skill's history and eval suite are preserved. Only the
changes introduced by the breaking revision need re-review.

**What counts as a breaking change:**
- Any modification to the trigger description
- Any change to the output format or schema
- Removal of a capability or supported input type
- Restructuring that changes which reference files are loaded and when

**What does not count as a breaking change:**
- Wording clarifications that do not alter meaning
- Addition of a new reference file for a new variant
- Typo fixes, formatting normalization
- Adding an entry to `KNOWN_ISSUES.md`
- Adding examples without changing instructions

---

## Versioning

Versioning is encoded in the skill's frontmatter and `CHANGELOG.md`. Version numbers must
never appear in the skill's `name` field, directory name, or trigger description —
they are invisible to Claude's trigger mechanism and must not pollute it.

### Semver Semantics for Skills

| Type | Definition | Requires |
|---|---|---|
| **Patch** `0.0.x` | Wording, typo, formatting, new example | No re-review |
| **Minor** `0.x.0` | New capability, new reference variant, coverage expansion | Tier 1 checklist |
| **Major** `x.0.0` | Breaking change to output, trigger rewrite, workflow restructure | Full review + regression |

### CHANGELOG Format

Every skill with ≥ 1 breaking change must have a `CHANGELOG.md`. Format:

```markdown
# Changelog

## [2.0.0] — YYYY-MM
### Breaking
- Output format changed: X now required at top
- Previous format documented in examples/bad/

### Changed
- Trigger description rewritten for improved accuracy

## [1.0.0] — YYYY-MM
### Initial validated release
```

---

## Ownership

Every skill must have a declared owner. The owner is responsible for:

- Keeping lifecycle state accurate in frontmatter
- Responding to reported failure modes within a reasonable time
- Reviewing and approving breaking changes from contributors
- Making the deprecation decision when the skill is superseded

Ownerless skills in VALIDATED state are a governance risk. If a skill's owner is
unavailable, a new owner must be assigned before the skill can receive breaking changes.

---

## Frontmatter Fields

These fields are required in every skill's SKILL.md frontmatter:

```yaml
---
name: skill-name
description: <trigger copy>
version: 1.0.0
framework-version: "1.0"
lifecycle: draft | review | tested | validated | deprecated
owner: <name or team>
last-reviewed: YYYY-MM
---
```

`framework-version` records which version of this framework the skill was written
against. As framework conventions evolve, this field allows linters and reviewers to
identify skills using deprecated patterns without inspecting every file.

Optional fields for skills with relevant history:

```yaml
breaking-change: "2.0.0"        # semver of last breaking change
superseded-by: other-skill-name # only when lifecycle: deprecated
```

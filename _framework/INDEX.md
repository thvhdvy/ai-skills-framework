# Skill Framework

Version: 1.0
Status: Active

This is the authoritative framework for designing, building, reviewing, and maintaining
Claude Skills in this repository. Every skill produced here is evaluated against the
standards defined in these documents.

---

## Document Index

Each document has a single responsibility. Read the document that owns the concern
you are working on. Do not expect a document to address concerns owned by another.

| Document | Owns | Read when |
|---|---|---|
| `PHILOSOPHY.md` | Core principles, governing constraints, design tensions | Starting a new skill; resolving a design conflict; understanding why a rule exists |
| `LIFECYCLE.md` | Skill states, promotion gates, versioning, frontmatter schema | Checking if a skill is ready to promote; adding a new frontmatter field; writing a CHANGELOG |
| `CONVENTIONS.md` | Repository structure, file naming, dependency declaration syntax | Creating a new skill directory; naming a file; declaring a dependency |
| `ACTIVATION.md` | Trigger description engineering, conflict detection, eval design | Writing or revising a description; measuring trigger accuracy; resolving a trigger conflict |
| `KNOWLEDGE.md` | Three-tier loading model, content placement, token optimization | Deciding where content belongs; reducing body size; writing loading conditions |
| `ARCHITECTURE.md` | Separation of concerns, extension patterns, conflict resolution, dependency governance | Extending a skill; resolving a domain ownership conflict; governing shared modules |
| `DOCUMENTATION.md` | Writing voice, structure conventions, formatting rules, anti-patterns | Writing any skill file; reviewing documentation quality; applying the style guide |
| `QUALITY.md` | Evaluation rubric, hard fail conditions, review workflow | Scoring a skill; conducting a review; understanding why a skill failed |
| `TEMPLATES.md` | Canonical file templates for every skill file type | Starting any new file; ensuring structural completeness |
| `EXTENSION.md` | Wrapper authoring, contribution process, fork governance, migration | Extending an existing skill; contributing to a shared skill; migrating after a base skill change |

---

## Quick Reference

### Starting a New Skill

1. Read `PHILOSOPHY.md` — understand the constraints before designing
2. Choose a directory and name per `CONVENTIONS.md`
3. Copy the SKILL.md template from `TEMPLATES.md`
4. Fill in frontmatter — schema in `LIFECYCLE.md`
5. Write the trigger description per `ACTIVATION.md`
6. Assign content to tiers per `KNOWLEDGE.md`
7. Set `lifecycle: draft` until peer review passes

### Reviewing a Skill

1. Run the hard fail checks in `QUALITY.md` first — any hard fail stops the review
2. Apply the full rubric in `QUALITY.md`
3. Check documentation against `DOCUMENTATION.md`
4. Verify trigger description against `ACTIVATION.md`
5. Record the score and any blocking issues before communicating results

### Extending an Existing Skill

1. Read `ARCHITECTURE.md` — Extension Patterns section
2. Follow the pattern selection guide in `EXTENSION.md`
3. If wrapping: trigger description must be strictly more specific than the base skill
4. If contributing: scope must be general-purpose, not org-specific
5. If forking: consult the base skill owner first; document the divergence

### Resolving a Trigger Conflict

1. `ACTIVATION.md` — Conflict Resolution section detects and describes the conflict
2. `ARCHITECTURE.md` — Conflict Resolution section provides the architectural resolution
3. If unresolved after both: hard fail per `QUALITY.md` HF — escalate to skill owners

---

## Hard Fail Summary

A skill scoring any hard fail scores 0 regardless of other quality. Full condition
definitions, detection methods, and fixes are in `QUALITY.md`.

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

---

## Evaluation Rubric Summary

Full scoring definitions and deductions are in `QUALITY.md`.

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

Score interpretation: 90–100 → VALIDATED / 75–89 → TESTED / 60–74 → REVIEW / <60 → DRAFT

---

## Repository Structure

```
skills/
├── _framework/     # This framework. Not a skill.
├── public/         # General-purpose utility skills. Triggerable.
├── shared/         # Dependency modules. Not triggerable. No description field.
├── plugins/        # Workflow and slash-command skills. Triggerable.
├── org/            # Organization-specific skills. Never submitted upstream.
├── examples/       # Reference implementations. Read-only for authors.
└── _deprecated/    # Retired skills. Preserved, never deleted.
```

---

## Framework Versioning

This framework is versioned independently of individual skills. The current version is
recorded at the top of this document. Skills record which framework version they were
written against in the `framework-version` frontmatter field.

When the framework is updated:
- Patch updates (clarifications, typo fixes): no skill updates required
- Minor updates (new guidance, new optional conventions): skills updated opportunistically
- Major updates (breaking convention changes): skills must be reviewed against new
  conventions before their next promotion; `framework-version` updated in frontmatter

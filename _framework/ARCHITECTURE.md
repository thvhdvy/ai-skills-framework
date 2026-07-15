# Skill Architecture

## Scope

This document owns: separation of concerns within and between skills, extension patterns,
architectural resolution of skill conflicts, and dependency governance. For folder and file
structure, see `CONVENTIONS.md`. For how conflicts are detected during activation, see
`ACTIVATION.md`. For decomposition criteria, see `KNOWLEDGE.md`.

---

## Separation of Concerns

Every file in a skill has exactly one responsibility. When a file's responsibility cannot
be stated in one sentence, it is doing too much.

### Responsibility Map

| File | Owns | Does not own |
|---|---|---|
| `SKILL.md` | Entry point, core workflow, loading conditions | Deep domain knowledge, org-specific content, shared schemas |
| `references/<variant>.md` | Domain-specific knowledge for one variant | Instructions that apply across variants |
| `scripts/<task>` | One deterministic, repeatable operation | Workflow logic, conditional branching |
| `templates/<type>.md` | One output structure | Instructions for how to fill the template |
| `checklists/<phase>.md` | Verification steps for one phase | Remediation guidance for failures |
| `KNOWN_ISSUES.md` | Documented failure modes and workarounds | Aspirational improvements, roadmap items |
| `CHANGELOG.md` | History of breaking changes | Rationale for non-breaking changes |

Mixing responsibilities within a file is a design smell. The most common violation is
embedding deep domain knowledge in `SKILL.md` rather than extracting it to `references/`.
The second most common is embedding workflow logic in a script rather than keeping scripts
purely operational.

---

## Extension Patterns

Three patterns are permitted. They are ordered by preference — use the earliest applicable
pattern before considering later ones.

### Pattern 1 — Wrapping (preferred)

Create a new skill with a more specific trigger description. The wrapper handles the
narrow, specialized layer and defers to the base skill's pattern for the core task.
The base skill is not modified.

```
Base skill:    engineering--code-review
               Trigger: broad code review requests

Wrapper skill: acme--go-code-review
               Trigger: Go code review with Acme's standards
               Behavior: applies Acme style guide, then follows base review pattern
```

**When to use:** Adding org-specific constraints, integrating with specific tools,
narrowing the scope of an existing skill.

**Dependency declaration:** The wrapper must declare its dependency on the base skill
in frontmatter. See `CONVENTIONS.md` for dependency syntax.

**Risk:** If the base skill introduces a breaking change, the wrapper may behave
unexpectedly. Wrapper owners are responsible for monitoring base skill changelogs.

**Trigger design:** The wrapper's description must be more specific than the base skill's.
See `ACTIVATION.md` for how specificity resolves trigger conflicts.

### Pattern 2 — Contribution (upstream change)

Propose a change to an existing shared or public skill through the review workflow.
The change must pass the full quality checklist for the skill's current lifecycle tier.

**When to use:** The gap is general-purpose — it would benefit all users of the skill,
not just one team or domain. Bug fixes, coverage gaps, newly discovered edge cases.

**Not permitted:** Adding org-specific content to a public skill. Organization-specific
knowledge belongs in `org/` skills, not in `public/` skills.

### Pattern 3 — Forking (last resort)

Copy a validated skill, rename it, modify freely. The fork has no relationship to the
original and will not receive upstream improvements.

**When to use:** The base skill's core behavior must change in a way incompatible with
its existing users. No wrapper or contribution can accommodate the required change.

**Requirements:**
- Document in `CHANGELOG.md` that this is a fork, what diverged, and why
- Do not reuse the base skill's `name` field — the fork is a new skill
- Accept full maintenance responsibility for the fork from the point of creation

**Cost:** Forks double the maintenance burden. Every improvement to the base skill must
be manually evaluated and potentially ported to the fork. Prefer wrapping or contribution
until forking is clearly necessary.

---

## Conflict Resolution

This section handles conflicts that `ACTIVATION.md` has identified but cannot resolve
through description engineering alone. Activation owns detection; architecture owns
resolution.

### When Conflicts Reach This Document

A conflict reaches architectural resolution when two skills:
1. Have overlapping trigger domains that boundary statements cannot cleanly separate, and
2. Are at equivalent specificity levels, meaning neither description is demonstrably
   more narrow than the other

At this point, the conflict is not a description problem — it is a domain ownership
problem. One of three resolutions applies.

### Resolution 1 — Merge

Merge the two skills into one when:
- Their trigger conditions overlap significantly (> 50% of queries could invoke either)
- Their workflows are compatible and can be unified without bloating the body beyond budget
- A single owner can reasonably maintain the merged skill

The merged skill takes the more general trigger description. The more specific behaviors
are handled conditionally within the body or routed to variant reference files.

### Resolution 2 — Refactor Boundaries

Refactor domain boundaries when:
- The skills are genuinely distinct in purpose but have been described imprecisely
- Each skill can be given a non-overlapping trigger domain with minimal workflow changes

The process:
1. Map the actual query space each skill should own
2. Rewrite both descriptions to reflect the redefined boundaries
3. Add explicit boundary statements to each skill naming the other
4. Re-test both skills' trigger accuracy per `ACTIVATION.md` before promoting

### Resolution 3 — Redefine Ownership

Redefine ownership when:
- One skill is a superset of the other's domain
- The overlap exists because a general skill has grown too broad

The narrower skill's domain is explicitly assigned to it. The broader skill adds a
boundary statement ceding that territory. If the broader skill's remaining domain is
still coherent, it continues to exist. If the broader skill's domain has been mostly
carved away by more specific skills, it is a deprecation candidate.

### Unresolved Conflicts

A conflict that cannot be resolved by any of the three patterns above is a hard fail.
Two skills competing non-deterministically for the same query space is worse than either
skill not existing. Escalate to architectural review before either skill reaches VALIDATED.

---

## Dependency Governance

### Permitted Dependency Graph

```
org/ skill  ──►  public/ skill   ✓
org/ skill  ──►  shared/ module  ✓
plugins/ skill ──►  public/ skill  ✓
plugins/ skill ──►  shared/ module ✓
public/ skill  ──►  shared/ module ✓

public/ skill  ──►  org/ skill    ✗  (org content in public skill)
public/ skill  ──►  plugins/ skill ✗  (workflow content in utility skill)
Any skill      ──►  Any skill (circular) ✗
```

### Shared Modules

Content that would otherwise be duplicated across two or more skills belongs in
`shared/`. Shared modules are not skills — they have no `description` frontmatter
field, do not appear in Claude's `available_skills` list, and are never invoked
directly. They are dependency artifacts only. For the structural rationale behind
the `shared/` vs `public/` separation, see `CONVENTIONS.md`.

A shared module is warranted when:
- Identical or near-identical content appears in two or more skills
- The content is stable enough to be versioned independently
- Multiple teams or skill domains need to reference it

A shared module is not warranted for content that is merely similar across skills.
Similar content that serves different purposes should remain in each skill independently.
Forced sharing of merely-similar content creates coupling without reducing duplication.

### Breaking Change Propagation

When a shared module or base skill introduces a breaking change:
1. The owner must increment the major version per `LIFECYCLE.md`
2. All dependent skills must be identified from dependency declarations in frontmatter
3. Dependent skill owners must be notified before the change is merged
4. A migration window of at least 30 days applies before the old version is removed

Dependent skills that are not updated within the migration window regress to REVIEW state
automatically, because their declared dependency is now stale.

---

## Architectural Anti-Patterns

| Anti-pattern | Why it fails | Resolution |
|---|---|---|
| Monolithic SKILL.md | Cannot be extended without modifying the only file | Extract to references/; apply decomposition criteria from `KNOWLEDGE.md` |
| Knowledge duplicated across skills | Two sources of truth diverge silently | Extract to `shared/`; replace duplicates with references |
| Wrapper that reimplements base logic | Defeats the purpose of wrapping; doubles maintenance | Remove reimplemented sections; defer to base pattern |
| Circular dependency | Context explosion; infinite reference chain | Restructure ownership; extract shared content to `shared/` |
| Org content in public skill | Pollutes general-purpose resource; breaks other users | Move to `org/` skill; contribute only general improvements upstream |
| Script with workflow logic | Scripts must be deterministic operations, not decision trees | Move branching logic to SKILL.md; keep scripts purely operational |

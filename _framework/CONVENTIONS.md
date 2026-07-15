# Skill Conventions

## Scope

This document owns: folder structure, file naming, frontmatter schema, and dependency
rules. For lifecycle states referenced below, see `LIFECYCLE.md`. For why these conventions
exist, see `PHILOSOPHY.md`.

---

## Repository Structure

```
skills/
├── _framework/          # Framework documents. Not a skill. Not indexed for triggering.
├── public/              # General-purpose utility skills. No org-specific content.
├── shared/              # Dependency modules. Not skills. Never triggered directly.
├── plugins/             # Workflow and slash-command oriented skills.
├── org/                 # Organization-specific skills. Never submitted upstream.
├── examples/            # Reference implementations. Read-only for skill authors.
└── _deprecated/         # Retired skills. Preserved, never deleted.
```

Leading underscores on `_framework/` and `_deprecated/` follow the Unix convention for
non-user-content directories and prevent them from appearing in skill discovery listings.

`shared/` contains dependency artifacts — schemas, common scripts, and reusable knowledge
modules — referenced by skills but never invoked directly. Files in `shared/` have no
`description` frontmatter field and do not appear in Claude's `available_skills` list.
The structural separation from `public/` makes this distinction unambiguous and prevents
shared modules from accidentally becoming trigger targets. See `ARCHITECTURE.md` for
shared module governance.

---

## Skill Directory Structure

```
skill-name/
├── SKILL.md             # Required. Entry point. Frontmatter + core instructions.
├── CHANGELOG.md         # Required when ≥ 1 breaking change has occurred.
├── KNOWN_ISSUES.md      # Required at lifecycle state TESTED and above.
│
├── references/          # Deep knowledge loaded on demand.
│   ├── README.md        # Required if references/ exists. Index with load conditions.
│   └── <variant>.md     # One file per independent domain variant.
│
├── scripts/             # Executable helpers for deterministic, repetitive operations.
│   └── <verb>-<noun>.<ext>  # Language determined by execution environment.
│
├── templates/           # Output templates for structured, fixed-format outputs.
│   └── <output-type>.md
│
├── checklists/          # Verification gates for workflows with mandatory steps.
│   └── <phase>.md
│
└── examples/            # Input/output pairs illustrating behavior.
    ├── good/
    └── bad/             # Anti-patterns Claude should recognize and avoid.
```

Every directory except `SKILL.md` is conditional. See the Presence-Necessity Principle
in `PHILOSOPHY.md` before adding any directory.

**Depth limit:** No nesting beyond two levels inside a skill directory. If a skill requires
deeper structure, it should be decomposed into multiple skills.

---

## Naming Conventions

### Directories

| Category | Pattern | Example |
|---|---|---|
| Public utility | `kebab-case` | `pdf-reader` |
| Plugin / slash-command | `domain--skill-name` | `engineering--code-review` |
| Org-specific | `org--topic--skill` | `acme--onboarding--checklist` |
| Deprecated | Original name preserved | `pdf-reader` (in `_deprecated/`) |

Double dash separates domain from skill name. Colons are avoided: they are illegal in
Windows filenames and require escaping in most shells.

### Files

| File | Convention | Example |
|---|---|---|
| Landmark files | `SCREAMING_SNAKE.md` | `SKILL.md`, `CHANGELOG.md` |
| Reference files | `kebab-case.md` | `aws-deployment.md` |
| Scripts | `verb-noun.<ext>` | `recalc.py`, `extract-tables.sh` |
| Templates | `kebab-case.md` | `adr-template.md` |
| Checklists | `kebab-case.md` | `pre-deploy.md` |

Screaming snake for landmark files makes them visually distinct in directory listings.
Reference files use descriptive names, never numbers — numbered files break when items
are inserted or removed.

### Frontmatter `name` Field

The `name` field is what appears in Claude's `available_skills` list. It must be:
- Human-readable, not technical
- Free of version numbers, domain prefixes, and org identifiers
- Consistent with how a user would naturally refer to the skill in a slash command

The directory name and the frontmatter `name` may differ. The directory name is for
humans navigating the repository. The frontmatter name is for Claude's trigger mechanism.

---

## Frontmatter Schema

Defined in full in `LIFECYCLE.md`. Conventions that apply here:

- `name` must not contain version numbers, dates, or category prefixes
- `description` must not contain output format instructions or workflow steps — those
  belong in the SKILL.md body
- `lifecycle` must be accurate at all times — a stale value is a hard fail (see `QUALITY.md`)

---

## Dependency Rules

### Permitted

- A skill may reference a module in `shared/` by path
- A wrapper skill may explicitly defer to a base skill's pattern
- A reference file may point to another reference file within the same skill

### Prohibited

- Circular dependencies between skills or between files within a skill
- Implicit dependencies — every referenced file must be explicitly named with a load
  condition in SKILL.md
- Cross-org dependencies — an `org/` skill must not reference another org's skill
- A `public/` skill must not reference a `plugins/` or `org/` skill

### Declaring Dependencies

When a skill depends on a shared module, declare it explicitly in frontmatter:

```yaml
depends-on:
  - shared/output-schemas
  - public/pdf-reader      # base skill dependency
```

This makes dependencies auditable and prevents silent breakage when a base skill or
shared module changes.

### Shared Knowledge

If the same knowledge would appear in two or more skills, it does not belong in either —
it belongs in `shared/`. Duplication of knowledge that should be shared is a hard fail.
See `QUALITY.md` for detection and resolution guidance.

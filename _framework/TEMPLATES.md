# Skill Templates

## Scope

This document owns: canonical templates for every file type in a skill directory.
Templates are starting points, not constraints — remove sections that do not apply.
For when each file is required versus conditional, see `CONVENTIONS.md`. For style
rules that apply when filling these templates, see `DOCUMENTATION.md`.

---

## SKILL.md Template

```markdown
---
name: <human-readable name; no version numbers or prefixes>
description: <trigger copy; see ACTIVATION.md for anatomy>
version: 0.1.0
framework-version: "1.0"
lifecycle: draft
owner: <name or team>
last-reviewed: YYYY-MM
depends-on:
  - shared/<module>        # remove if no shared dependencies
  - public/<base-skill>    # remove if no base skill dependency
---

# <Skill Name>

<One sentence: what this skill does and for whom. No preamble.>

## Orientation

<Immediate constraints, prerequisites, or context Claude needs before starting.
Omit this section if there are none.>

## Input Handling

<How to interpret the user's request. What to do when input is ambiguous, incomplete,
or malformed. Omit if input handling is straightforward.>

## Workflow

<The steps to execute, in order. Use numbered list only when order is semantically
significant. Use prose or table when steps are conditional or parallel.>

## Output Format

<The required structure of the result. Use a template with placeholders rather than
an example output where possible.>

```
[Output template here]
```

## Conditional Loading

<Reference files and the exact condition under which each should be loaded.
Every reference must have an explicit condition. Omit section if no references exist.>

If <condition>, read `references/<variant>.md` before proceeding.

## Edge Cases

<Only cases Claude cannot handle correctly by inference from the workflow above.
Each entry: the trigger condition and the correct response. Omit if none are known.>

## Self-Review Checklist

Before responding, verify:
- [ ] <Skill-specific correctness check>
- [ ] <Output format compliance check>
- [ ] <Quality or completeness check>
```

---

## references/README.md Template

Required when three or more reference files exist in `references/`.

```markdown
# References Index

Load exactly one file per invocation unless instructions explicitly permit multiple.

| File | Load when | Contents |
|---|---|---|
| `<variant>.md` | <condition> | <one-line description> |
| `<variant>.md` | <condition> | <one-line description> |

If no condition is met, proceed with the SKILL.md body only.
```

---

## Reference File Template

```markdown
# <Variant Name>

> Loaded when: <restate the condition from SKILL.md for self-documentation>

## <First Section>

<Content specific to this variant. Self-contained — do not reference other variant files.>

## <Second Section>

<Continue as needed. Use the same section structure as sibling variant files.>
```

---

## Checklist File Template

```markdown
# <Phase Name> Checklist

> Run this checklist at: <the point in the workflow where this checklist applies>

## Required Checks

- [ ] <Check — state what to verify, not what to do>
      If failed: <specific corrective action>

- [ ] <Check>
      If failed: <specific corrective action>

## Advisory Checks

<Checks that represent best practice but do not block progression.
Omit this section if all checks are required.>

- [ ] <Advisory check>
```

---

## KNOWN_ISSUES.md Template

Required at TESTED state and above. See `DOCUMENTATION.md` for style conventions.

```markdown
# Known Issues

## [Short failure description — verb phrase]

**Trigger condition:** <The specific input or context that causes this failure.>
**Observed behavior:** <What Claude actually does when this failure occurs.>
**Workaround:** <What the user or caller can do to avoid or recover.>
**Status:** Open | Mitigated | Resolved in <version>

---

## [Next issue]

...
```

---

## CHANGELOG.md Template

Required when one or more breaking changes have occurred. See `LIFECYCLE.md` for
versioning semantics.

```markdown
# Changelog

## [<version>] — YYYY-MM

### Breaking
- <Past-tense description of breaking change>
- Previous behavior documented in `examples/bad/<example>.md`

### Added
- <New capability or reference file>

### Changed
- <Non-breaking modification>

### Fixed
- <Bug fix referencing KNOWN_ISSUES.md entry if applicable>

---

## [<earlier version>] — YYYY-MM

...
```

---

## examples/ Convention

The `examples/` directory uses a flat structure with descriptive file names.
Subdirectories `good/` and `bad/` are required when both types exist.

```
examples/
├── good/
│   └── <descriptive-scenario-name>.md
└── bad/
    └── <descriptive-anti-pattern-name>.md
```

Each example file follows this structure:

```markdown
# <Scenario Description>

## Input

<The user request or input that triggered this example.>

## Output

<The correct (good/) or incorrect (bad/) output.>

## Why

<For good/: what makes this output correct and what principles it demonstrates.>
<For bad/: what is wrong with this output and what mistake it illustrates.>
```

The `Why` section is what separates a useful example from a reference output.
Without it, Claude cannot generalize from the example to adjacent cases.

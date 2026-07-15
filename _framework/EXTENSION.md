# Extension Guide

## Scope

This document owns: practical guidance for authors extending existing skills — how to
choose an extension pattern, how to author each pattern correctly, and how to migrate
when a base skill changes. For the three extension patterns and their architectural
rationale, see `ARCHITECTURE.md`. This document is the implementation companion to
that specification.

---

## Choosing a Pattern

Answer these questions in order. Stop at the first match.

```
1. Does the base skill's core behavior stay unchanged?
   └─ Yes → Wrapper (Pattern 1)
   └─ No  → continue

2. Is the change general-purpose — beneficial to all users of the base skill?
   └─ Yes → Contribution (Pattern 2)
   └─ No  → continue

3. Is the base skill's core behavior fundamentally incompatible with your requirements?
   └─ Yes → Fork (Pattern 3)
   └─ No  → reconsider — you may be solving the wrong problem
```

If you reach Pattern 3 and are not certain the answer to question 3 is yes, stop.
Consult the base skill's owner before forking. Forks are difficult to undo and create
permanent maintenance burden.

---

## Pattern 1 — Authoring a Wrapper

### What a Wrapper Looks Like

A wrapper skill has three components its base skill does not:

1. A trigger description that is strictly more specific than the base skill's
2. Additional behavior that applies before or after the base skill's workflow
3. An explicit declaration of the base skill dependency in frontmatter

```yaml
# Wrapper frontmatter
depends-on:
  - public/engineering--code-review    # base skill
```

### Trigger Specificity Requirement

The wrapper's description must be narrow enough that it does not compete with the base
skill for the base skill's intended queries. The base skill owns the general case; the
wrapper owns one specific slice of it.

```
# Base skill description (general)
"Review code changes for security, performance, and correctness."

# Wrapper description (specific — adds constraint and context)
"Review Go code changes against Acme Engineering's style guide, including
naming conventions and error handling patterns specific to Acme's codebase.
Use for Go files only — for other languages, the general code-review skill applies."
```

The boundary statement at the end of the wrapper description ("for other languages,
the general code-review skill applies") prevents the wrapper from consuming queries
the base skill should handle.

### Deferring to the Base Pattern

A wrapper should not reimplement the base skill's workflow. Reference the base skill's
pattern and add only what is specific to the wrapper's context.

```markdown
## Workflow

1. Apply Acme Go style guide checks (see `references/acme-go-style.md`).
2. Check error handling patterns against `references/acme-error-patterns.md`.
3. Follow the standard code review workflow for the remaining review dimensions.
```

Step 3 defers to the base skill without reimplementing it. This keeps the wrapper
lean and ensures it inherits improvements to the base skill automatically.

### Monitoring Base Skill Changes

Wrapper owners are responsible for monitoring the base skill's `CHANGELOG.md`. When
the base skill releases a major version:

1. Read the breaking change description
2. Assess whether the wrapper's deferral to the base pattern is still valid
3. Update the wrapper if the base skill's workflow has changed in ways that affect
   the wrapper's additional steps
4. Update `framework-version` and `last-reviewed` in the wrapper's frontmatter

A wrapper that depends on a base skill that has undergone a breaking change without
a corresponding wrapper review is a candidate for lifecycle regression.

---

## Pattern 2 — Contributing to an Existing Skill

### Contribution Scope

Contributions are appropriate for:
- Bug fixes — behavior that is wrong for documented use cases
- Coverage gaps — use cases the skill should handle but does not
- New reference variants — adding support for a new domain without changing existing variants
- Documentation improvements — clarity, completeness, cross-reference accuracy

Contributions are not appropriate for:
- Org-specific content in a public skill
- Behavior changes that benefit one team but may break others
- Workflow restructuring that constitutes a breaking change without the base skill
  owner's explicit approval

### Contribution Process

1. Identify the change scope — patch, minor, or major per `LIFECYCLE.md`
2. For major changes, consult the skill owner before writing anything
3. Apply the change following `DOCUMENTATION.md` style conventions
4. Run the full quality checklist in `QUALITY.md` for the skill's current lifecycle tier
5. If the change affects trigger behavior, re-run trigger accuracy measurement per
   `ACTIVATION.md`
6. Update `CHANGELOG.md` and increment `version` in frontmatter
7. If the change is breaking, notify all dependent skill owners per the migration
   window process in `ARCHITECTURE.md`

### What Not to Contribute

Do not contribute content that you would not want applied to all users of the skill.
If the content is correct only in your context — your tools, your organization, your
domain — it belongs in a wrapper or an org skill, not in a public or shared skill.

---

## Pattern 3 — Forking

### Before Forking

Contact the base skill's owner. Describe the required change. In many cases, a
breaking contribution with owner approval is preferable to a fork — the change is
upstreamed, all users benefit, and the maintenance burden remains shared.

Fork only when:
- The owner has declined the change or is unavailable
- The required behavior is incompatible with the base skill's existing users
- The change is org-specific and would never be appropriate upstream

### Fork Authoring Requirements

A fork is a new skill, not a version of the old one. It must:

1. Use a new `name` in frontmatter — not the base skill's name
2. Use a new directory name that does not shadow the base skill
3. Document the fork in `CHANGELOG.md`:

```markdown
## [1.0.0] — YYYY-MM

### Fork
- Forked from `public/engineering--code-review` at version 2.1.0
- Divergence: replaced output format with Acme's internal review schema
- Base skill improvements will not be automatically inherited
```

4. Remove the `depends-on` reference to the base skill — a fork has no dependency
   relationship with its origin

### Fork Maintenance Responsibility

After forking, the fork owner receives no improvements from the base skill. Periodically
review the base skill's `CHANGELOG.md` and manually port improvements that apply.
The fork's `last-reviewed` frontmatter field tracks when this review last occurred.

---

## Migration Patterns

### When a Base Skill Releases a Breaking Change

Base skill breaking changes propagate a 30-day migration window to all dependent skills.
During this window:

1. **Read the breaking change entry** in the base skill's `CHANGELOG.md`
2. **Assess impact** — does the breaking change affect the code paths your wrapper invokes?
3. **Update if affected:**
   - Revise deferral language in the wrapper to match the new base skill workflow
   - Update any reference files that describe base skill behavior
   - Increment wrapper version (patch if documentation only; minor if behavior changes)
4. **Verify** — re-run the wrapper's eval suite against the updated base skill
5. **Update frontmatter** — `last-reviewed`, `version`, `breaking-change` if applicable

If no update is needed, still update `last-reviewed` to confirm the review occurred.
A wrapper whose `last-reviewed` date predates a base skill breaking change is ambiguous —
it may have been reviewed and found unaffected, or it may have been missed.

### When a Shared Module Changes

Shared module changes follow the same 30-day window. The process is identical to base
skill migration with one addition: shared modules may be referenced by many more skills
than a single base skill. If you own a shared module, use the `depends-on` declarations
across the repository to identify all dependents before releasing a breaking change.

### When a Skill Is Deprecated

When a skill you wrap or depend on is deprecated:

1. Identify the replacement skill named in `SUPERSEDED_BY.md`
2. Evaluate whether the replacement covers your wrapper's use case
3. Either update the wrapper to depend on the replacement, or absorb the deprecated
   skill's functionality directly if the replacement is not suitable
4. The deprecated skill remains available during the 30-day grace period — do not
   rush migration at the cost of correctness

---

## Extension Anti-Patterns

| Anti-pattern | Why it fails | Correct approach |
|---|---|---|
| Wrapper reimplements base workflow | Doubles maintenance; loses base skill improvements | Defer explicitly; add only the wrapper-specific steps |
| Contribution adds org-specific content | Pollutes shared resource; breaks other users | Move to org/ wrapper instead |
| Fork with same name as base skill | Trigger conflict; users cannot distinguish them | Choose a distinct name that reflects the fork's specialization |
| Wrapper with description as broad as base skill | Competes non-deterministically with base skill | Narrow the description; add boundary statement |
| Dependency declared but not used | Misleading frontmatter; creates false audit trail | Remove unused dependencies |
| Migration skipped after base skill breaking change | Wrapper behavior undefined against new base | Review within the 30-day window; update `last-reviewed` regardless of outcome |

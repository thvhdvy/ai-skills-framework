# Skill Activation

## Scope

This document owns: trigger description engineering, conflict resolution between competing
skills, boundary statement patterns, and trigger testing discipline. For how activation
failures affect lifecycle state, see `LIFECYCLE.md`. For scoring, see `QUALITY.md`.

---

## How Activation Works

Claude receives a list of `(name, description)` pairs from the active skill profile.
For each user message, Claude decides which skills — if any — to invoke based solely on
that list. The skill body, references, and scripts are not visible at decision time.

This means the `description` field is the complete activation surface. Nothing inside
the skill can compensate for a weak description. Nothing outside the description
influences whether the skill fires.

Two failure modes follow directly:

- **False negative:** The skill should have fired and did not. The user's task is handled
  without the skill's domain knowledge, producing lower-quality output.
- **False positive:** The skill fired when it should not have. The skill's instructions
  shape Claude's behavior on an unrelated task, potentially degrading output.

Both failures are silent. The user rarely knows a skill was or wasn't invoked.

---

## Description Anatomy

A well-formed description has four components. Not every skill needs all four — apply
the Presence-Necessity Principle from `PHILOSOPHY.md`.

### 1. Primary Action Statement

One sentence. Verb-first. States what the skill does at the highest level of abstraction
that is still specific enough to be useful.

```
# Weak — too vague to trigger accurately
"Helps with documents."

# Strong — specific action, specific artifact
"Create, edit, and extract content from Word documents (.docx)."
```

### 2. Trigger Inventory

Concrete phrases, file types, and contexts that should trigger the skill. This is the
primary mechanism for catching explicit invocations.

```
# Trigger inventory example
"Use when the user mentions .docx, .dotx, asks for a Word document,
or needs to produce a report, memo, or letter as a file."
```

Include: file extensions, product names, user-facing terminology, common synonyms.
Avoid: abstract category names, internal jargon the user would never say.

### 3. Coverage Expansion

Implicit cases — queries where the user clearly needs this skill but does not name
the skill, the file type, or any obvious trigger word. This is the primary mechanism
for reducing false negatives.

```
# Coverage expansion example
"Also trigger when the user wants to produce a professional deliverable
to share with a client or send to a colleague, even if they do not
specify the format."
```

Coverage expansion requires judgment. Expand too broadly and false positives increase.
The test: would a reasonable person, seeing this query and this skill, agree the skill
should fire?

### 4. Boundary Statement

Explicit statement of what this skill does not cover. Required only when meaningful
overlap exists with another active skill. Omit when there is no plausible conflict.

```
# Boundary statement example
"Do not trigger for spreadsheet files (.xlsx, .csv) or presentations
(.pptx) — use the appropriate skill for those formats."
```

Boundary statements are written for Claude, not for the user. They should name the
competing domain specifically, not gesture at it vaguely.

---

## Anti-Patterns

These patterns reliably degrade trigger accuracy. All are prohibited.

| Anti-pattern | Problem | Fix |
|---|---|---|
| Starts with "This skill..." | Wastes tokens on what Claude already knows | Start with the action verb |
| "Helps with", "assists in", "can be used for" | Vague — matches almost anything | Name the specific action |
| Lists every conceivable edge case | Noisy — hurts precision | Name the category, not the cases |
| Contains version numbers or dates | Goes stale silently | Use "current" or remove |
| Contains output format instructions | Belongs in the body, not the description | Move to SKILL.md body |
| Contains workflow steps | Belongs in the body, not the description | Move to SKILL.md body |
| Passive voice throughout | Under-triggers — passive descriptions feel less actionable | Rewrite in active, imperative voice |

---

## Conflict Resolution

When two skills could plausibly trigger on the same query, Claude's behavior is
non-deterministic. This is a design smell, not a Claude limitation.

**Resolution priority:**

1. **Specificity wins.** Claude consistently prefers more specific descriptions over
   general ones. A skill with a narrow, concrete description will usually beat a skill
   with a broad one on overlapping queries. Design specific skills to use narrow language;
   design general skills to use broad language.

2. **Boundary statements.** When specificity alone does not resolve the conflict, the
   more specific skill should carry a boundary statement directing Claude away from the
   general skill's territory, and the general skill should carry a boundary statement
   ceding that territory.

3. **Decompose or merge.** If two skills at the same specificity level both want to
   trigger on the same query family, one of three things is true: they should be merged
   into one skill, one of them has a trigger description that is too broad, or there is
   a genuine domain overlap that requires an architectural decision about which skill
   owns that territory. When description engineering alone cannot resolve the conflict,
   escalate to `ARCHITECTURE.md` — specifically the Conflict Resolution section, which
   defines the merge, refactor, and ownership-redefinition patterns.

Unresolved conflicts between skills at the same specificity level are a hard fail.
See `QUALITY.md`.

---

## Trigger Testing Discipline

Trigger accuracy must be measured empirically before a skill can reach TESTED state.
Assumed triggering is not sufficient.

### Eval Query Design

Trigger evals require two query types:

**Should-trigger queries (8–10):**
- Vary phrasing: formal, casual, abbreviated, with typos
- Include cases where the user does not name the skill or file type explicitly
- Include cases where this skill competes with another but should win
- Use realistic context: file paths, column names, colleague names, backstory

```
# Weak — too obvious, tests nothing
"Extract text from PDF"

# Strong — realistic, tests implicit triggering
"my manager sent over last quarter's board deck as a pdf and i need
to pull out the revenue figures for each region — can you help"
```

**Should-not-trigger queries (8–10):**
- Near-misses: share vocabulary with the skill but need something different
- Adjacent domain: plausibly related but handled by a different skill
- Ambiguous phrasing: a naive keyword match would trigger but shouldn't

```
# Weak negative — obviously irrelevant, tests nothing
"Write a fibonacci function"

# Strong negative — genuine near-miss
"I need to fill out a PDF form my landlord sent me"
# (Should trigger pdf-forms skill, not pdf-reader skill)
```

### Accuracy Threshold

The framework default is ≥ 80% accuracy across both query types before promotion to
TESTED. This is a starting point based on practical experience, not an empirically
validated constant. Individual repositories or domains may declare a stricter threshold
when appropriate — skills in high-stakes domains where a false positive could actively
degrade a critical task should adopt 90%+.

Accuracy is measured as:

```
(correct triggers + correct non-triggers) / total queries
```

A skill that triggers correctly on 100% of should-trigger queries but fires on 50% of
should-not-trigger queries scores 75% — below the default threshold. Both axes matter
equally.

### Optimization Loop

When accuracy is below threshold:

1. Identify which query type is failing (false negatives or false positives)
2. False negatives → expand Coverage Expansion section of the description
3. False positives → add or sharpen Boundary Statement
4. Re-test after each change — do not batch multiple description changes before testing,
   as this makes it impossible to attribute accuracy changes to specific edits
5. Document final accuracy in `KNOWN_ISSUES.md` with the eval set used

---

## Activation Criteria Completeness

A skill with missing or ambiguous activation criteria is a hard fail. The standard for
completeness:

- A reviewer who has never seen the skill must be able to state, without ambiguity,
  which of the following three things will happen for any given user query:
  1. This skill should trigger
  2. This skill should not trigger
  3. This is a genuine edge case (and the edge case is documented)

If the reviewer cannot make that determination from the description alone, the activation
criteria are ambiguous. Rewrite before promotion.

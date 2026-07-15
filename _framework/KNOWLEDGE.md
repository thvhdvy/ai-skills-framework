# Knowledge Organization

## Scope

This document owns: the three-tier loading model, rules for placing content within tiers,
loading condition syntax, module decomposition criteria, and token optimization techniques.
For why context economy is a first-class constraint, see `PHILOSOPHY.md`. For how token
budget violations affect scoring, see `QUALITY.md`.

---

## The Three-Tier Model

Every piece of content in a skill belongs to exactly one tier. Tier assignment determines
when the content is loaded into Claude's context and what it costs on every invocation.

```
TIER 1 — Always present
  name + description (frontmatter only)
  Cost: paid on every conversation where this skill is in the profile
  Budget: ≤ 150 tokens

TIER 2 — On trigger
  SKILL.md body
  Cost: paid on every invocation
  Budget: ≤ 800 tokens (focused skill) / ≤ 1,500 tokens (complex workflow skill)

TIER 3 — On demand
  references/, scripts/, templates/, checklists/, examples/
  Cost: paid only when explicitly loaded
  Budget: no hard limit per file; total volume > 5,000 tokens signals decomposition
```

These budget figures are guidelines derived from practical experience with existing
skills, not empirically validated constants. Treat them as calibrated starting points.
A skill that exceeds a budget with genuinely essential content is preferable to one
that meets the budget by omitting necessary instruction.

Tier assignment is not a judgment call — it follows from a single question applied to
each piece of content:

> Is this content needed for the majority of invocations?

- **Yes → Tier 2.** It belongs in the SKILL.md body.
- **No → Tier 3.** It belongs in a reference file, loaded conditionally.
- **Never → delete it.** Content that no invocation needs has no place in the skill.

"Majority" means more than half of realistic invocations, weighted by frequency, not by
edge-case coverage. A case that affects 5% of invocations does not belong in Tier 2
even if it seems important.

---

## Loading Conditions

Every Tier 3 file must be referenced from SKILL.md with an explicit loading condition.
An unconditional reference — "see `references/advanced.md`" — is prohibited. It forces
Claude to decide whether to read the file on every invocation, consuming attention without
guidance, and may result in the file being read when it is not needed.

### Condition Syntax

Loading conditions follow a consistent pattern in SKILL.md:

```markdown
If <condition>, read `references/<variant>.md` before proceeding.
```

The condition must be specific enough that Claude can evaluate it without reading the
file. If evaluating the condition requires reading the file, the condition is wrong.

### Condition Types

| Type | Example |
|---|---|
| User-stated format | If the user is working with AWS, read `references/aws.md`. |
| Input characteristic | If the input file is scanned (no selectable text), read `references/ocr.md`. |
| Task complexity | If the request involves more than three output sections, read `references/structure.md`. |
| Output type | If the output must be a formal deliverable, read `templates/formal-report.md`. |
| Failure signal | If extraction returns empty results, read `references/fallback-strategies.md`. |

### References Index

When a skill has three or more reference files, `references/README.md` is required.
It must list every file, its loading condition, and a one-line description. This index
is the first file Claude reads when entering the `references/` directory.

```markdown
# References Index

| File | Load when | Contents |
|---|---|---|
| `aws.md` | User is deploying to AWS | AWS-specific deployment patterns |
| `gcp.md` | User is deploying to GCP | GCP-specific deployment patterns |
| `fallback.md` | Primary approach fails | Alternative strategies and diagnostics |
```

---

## Decomposition Criteria

### Decompose SKILL.md into body + references when:

- A coherent section is needed for fewer than approximately 40% of invocations
  (this figure is a practical guideline, not a measured threshold — use judgment)
- A section's content is stable and independently versioned from the rest of the skill
- The body would exceed its token budget guidelines without that section

### Decompose a skill into multiple skills when:

- It has two or more trigger conditions that rarely co-occur in the same session
- Its body exceeds 1,500 tokens after optimization
- It produces two or more output types that require substantially different workflows
- Different user roles invoke it for different purposes with different success criteria

### Do not decompose when:

- The sections are always read together — decomposition adds file I/O overhead without
  reducing context cost
- The skill is already within budget
- The proposed sub-skills would have overlapping trigger conditions, creating the conflict
  described in `ACTIVATION.md`

### Variant Organization

When a skill operates differently across clearly distinct domains, organize references
by variant. Variant files are mutually exclusive — Claude reads at most one per invocation.

```
references/
├── README.md      # Index with selection criteria
├── aws.md         # AWS-specific patterns
├── gcp.md         # GCP-specific patterns
└── azure.md       # Azure-specific patterns
```

Variant files must:
- Be self-contained for their domain. No cross-references between variant files.
- Use identical internal structure. Structural consistency reduces Claude's orientation cost.
- Have explicit selection criteria in SKILL.md that allow Claude to choose without reading
  any variant file first.

---

## Token Optimization Techniques

These techniques reduce token cost without reducing instruction quality. Apply in order
of impact.

### 1. Tables over prose for comparisons

A comparison that would require three paragraphs in prose requires one table. Tables
are also more scannable for Claude's attention mechanism — the structure signals
relationships that prose must state explicitly.

```markdown
# Before — prose (≈ 60 tokens)
When the input is a scanned PDF, use the OCR pipeline. When the input is a native PDF
with selectable text, use direct extraction. When the input format is unknown, probe
for selectable text first and fall back to OCR if extraction returns empty.

# After — table (≈ 30 tokens)
| Input type | Approach |
|---|---|
| Scanned PDF | OCR pipeline |
| Native PDF | Direct extraction |
| Unknown | Probe → fall back to OCR |
```

### 2. Prefer deterministic artifacts over prose

When an operation can be expressed as a code block, schema, template, or structured
format, prefer that form over prose description. Deterministic artifacts are
unambiguous — they constrain Claude's output space precisely and prevent variation
across invocations. Prose descriptions of the same content invite interpretation.

This principle applies beyond programming skills: a form template is more precise
than prose describing what the form should contain; a decision table is more reliable
than prose describing a decision process; a schema is more reliable than prose
describing a data structure.

### 3. Output templates as structure, not examples

A template with placeholders constrains the output space more efficiently than one or
more example outputs. Show the template once; do not show the template and then an
example of a filled template — that doubles the token cost for marginal benefit.

### 4. Explained reasoning over enumerated rules

A single sentence explaining *why* covers the cases the rule was written for plus the
cases it was not. An enumerated rule only covers what was anticipated.

```markdown
# Before — enumerated rule (covers 3 cases, costs 20 tokens)
Always include a summary. Always put it at the top. Always keep it under 3 sentences.

# After — explained reasoning (covers all cases, costs 15 tokens)
Open with a 1–3 sentence summary. Stakeholders often read only the first section;
front-loading the conclusion ensures value even if they stop reading.
```

### 5. Conditional loading over defensive inclusion

Do not include content in Tier 2 "just in case." If content is needed for fewer than
~40% of invocations, it belongs in Tier 3 with a loading condition. Defensive inclusion
is the primary source of token debt as defined in `PHILOSOPHY.md`.

### 6. Reference pointers must be conditional

"See `references/` for more details" is prohibited. It transfers the loading decision
to Claude on every invocation, consuming attention. Every reference pointer must specify
the condition under which the file should be read.

---

## Token Budget Enforcement

Token budgets are targets, not hard limits. The quality scoring in `QUALITY.md` penalizes
budget overruns, but a skill that exceeds its budget with genuinely essential content is
preferable to a skill that meets its budget by omitting necessary instruction.

When a skill body consistently exceeds budget after optimization:

1. Audit for token debt — instructions that exist because of past failures but are no
   longer needed given Claude's current capabilities
2. Move minority-case content to Tier 3 with explicit loading conditions
3. If the body remains over budget after both steps, the skill is a decomposition candidate

Do not reduce token cost by removing genuinely necessary instruction. A lean skill that
fails its task is worse than an over-budget skill that succeeds.

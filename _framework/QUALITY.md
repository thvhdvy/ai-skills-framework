# Skill Quality

## Scope

This document owns: the evaluation rubric, hard fail conditions, scoring guidance, and
the review workflow. It is the authoritative reference for every quality gate in the
framework. For lifecycle promotion requirements, see `LIFECYCLE.md`. For the style
standards that feed into Documentation Quality scoring, see `DOCUMENTATION.md`.

---

## Evaluation Rubric

Every skill is scored 0–100 before reaching VALIDATED state. The rubric is applied by
the skill owner and at least one independent reviewer.

**Governing principle:** Presence is never scored higher than necessity. A criterion is
satisfied by correct judgment about what to include, not by maximizing content volume.
Absence of an element scores full marks when that element is genuinely not warranted.

---

### 1. Trigger Fidelity — 20 points

The description field is the complete activation surface. Score against all three axes.

| Axis | Points | Evaluation criteria |
|---|---|---|
| Appropriate activation | 8 | Fires on core use cases when tested with realistic prompts |
| No false positives | 6 | Stays silent on near-miss and adjacent-domain queries |
| No false negatives | 6 | Fires on implicit phrasings where the need is clear but the skill is not named |

**Deductions:**
- Vague trigger language ("helps with", "assists in"): −3
- Missing boundary statement when overlap with another skill exists: −3
- Version numbers or dates in description: −2
- Description starts with "This skill" or passive voice: −1
- Output format instructions or workflow steps in description: −3

**Threshold:** Skills must achieve ≥ 80% trigger accuracy across a balanced eval set
before reaching TESTED state. This threshold is a framework default. Individual
repositories or domains may declare a stricter threshold in their configuration.
See `ACTIVATION.md` for eval set design and accuracy measurement.

---

### 2. Context Economy — 20 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Minimal loading cost | 8 | Tier 1 ≤ 150 tokens; Tier 2 within budget guidelines for skill complexity |
| Modular structure | 7 | Content correctly placed across tiers; reference files loaded conditionally |
| Efficient knowledge organization | 5 | Deterministic artifacts preferred over prose; no redundant restatement |

**Deductions:**
- Tier 2 content that belongs in Tier 3: −3 per instance
- Unconditional reference file loading: −2 per instance
- Information duplicated across files: −2 per instance
- Prose where a table, code block, or template would suffice: −1 per instance

Token budgets are guidelines derived from practical experience, not empirically
validated constants. See `KNOWLEDGE.md` for budget values and the rationale for
treating them as guidelines rather than hard limits.

---

### 3. Architecture and Modularity — 15 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Separation of concerns | 6 | Each file has exactly one responsibility per the map in `ARCHITECTURE.md` |
| Reusable modules | 5 | Reference files, scripts, and templates are usable by wrapper skills without modification |
| Extensibility | 4 | Clear extension path exists; dependencies are explicit and one-directional |

**Deductions:**
- Monolithic SKILL.md with no extension path: hard fail (see below)
- Implicit dependencies: −3
- Org-specific content in a public skill: hard fail (see below)
- Script containing workflow logic or branching: −3

---

### 4. Documentation Quality — 10 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Clear | 4 | Cold-read test passes: peer understands purpose and usage within 60 seconds |
| Concise | 3 | No preamble, no hedging, no restatement of user context |
| Consistent | 3 | Voice, heading levels, and formatting match `DOCUMENTATION.md` throughout |

**Deductions:**
- ALWAYS / NEVER without justification: −2 per instance
- Passive voice throughout: −1
- What stated without why for non-obvious instructions: −1 per instance
- Anti-patterns from `DOCUMENTATION.md`: −1 per instance

---

### 5. Examples and Templates — 10 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Practical | 4 | Reflect real invocation patterns; no toy scenarios or placeholder content |
| Reusable | 3 | Templates are parameterized; examples cover a range of input types |
| Production-oriented | 3 | `examples/bad/` present when anti-patterns need illustration |

Scored on necessity. A skill with unstructured output that correctly has no template
scores 10/10. A skill that adds examples purely to increase this score is penalized
under Context Economy.

---

### 6. Maintainability — 10 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Easy to update | 5 | Small blast radius; lifecycle state accurate; sections independently updatable |
| Low duplication | 5 | Information stated once; `CHANGELOG.md` exists when required |

**Deductions:**
- Information duplicated across files: −2 per instance
- Missing `CHANGELOG.md` when a breaking change has occurred: −3
- Stale lifecycle status: hard fail (see below)

---

### 7. Naming and Consistency — 5 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Predictable naming | 3 | Directory, frontmatter name, and all files follow `CONVENTIONS.md` |
| Consistent conventions | 2 | Heading levels, code block languages, frontmatter fields match framework spec |

Largely pass/fail. Naming either follows conventions or it does not.

---

### 8. Workflow Design — 5 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Logical | 2 | Steps in correct order; no step depends on information not yet gathered |
| Repeatable | 2 | Similar inputs produce consistent outputs across invocations |
| Human-friendly | 1 | Handoff points to human-in-the-loop are explicit and unambiguous |

---

### 9. Self-Review Capability — 5 points

| Axis | Points | Evaluation criteria |
|---|---|---|
| Includes quality checklist | 3 | Checklist exists covering the skill's own correctness criteria |
| Encourages critical evaluation | 2 | Checklist is diagnostic: specifies why a check matters and how to fix failure |

Scored on quality, not length. A five-item targeted checklist scores higher than a
twenty-item generic one. The checklist must be skill-specific, not a copy of this rubric.

---

## Score Interpretation

| Score | Lifecycle state |
|---|---|
| 90–100 | VALIDATED — production ready |
| 75–89 | TESTED — real invocations permitted with documented known issues |
| 60–74 | REVIEW — functional but requires improvement before production |
| < 60 | DRAFT — not ready for use |

A skill that scores below its declared lifecycle state must be regressed immediately.

---

## Hard Fail Conditions

A hard fail scores the skill at 0 regardless of total points accumulated elsewhere.
Hard fails represent failure modes severe enough that partial quality is meaningless —
the skill cannot be trusted in its current state.

Each condition below includes: why it is a hard fail, how to detect it, and how to fix it.

---

### HF-01 — Missing SKILL.md

**Why a hard fail:** SKILL.md is the entry point for the entire three-tier loading
system. Without it, there is no skill — only a directory of files with no activation
mechanism and no instruction set. No other file in the skill directory can substitute.

**How to detect:** `ls skill-name/SKILL.md` returns no result. Or the directory exists
in the repository without a SKILL.md at its root.

**How to fix:** Create SKILL.md with valid frontmatter and a minimal body before any
other work on the skill proceeds. A skeleton SKILL.md in DRAFT state is always
preferable to no SKILL.md.

---

### HF-02 — Missing or Ambiguous Activation Criteria

**Why a hard fail:** A skill with no coherent activation logic cannot be invoked
reliably. It may fire on unrelated queries, fail to fire on intended ones, or behave
non-deterministically. An ambiguous description is worse than no description — it
creates false confidence that activation is working while delivering inconsistent results.

**How to detect:** Apply the completeness standard from `ACTIVATION.md`: a reviewer
who has never seen the skill must be able to determine, from the description alone,
whether any given query should or should not trigger it. If they cannot, the criteria
are ambiguous. Additional signals: description contains no specific trigger phrases,
description is under 20 words, or description contains only category-level language
with no concrete examples.

**How to fix:** Rewrite the description following the four-component anatomy in
`ACTIVATION.md`. Test against a balanced eval set. Do not promote past DRAFT until
trigger accuracy meets the configured threshold.

---

### HF-03 — References to Non-Existent Files or Modules

**Why a hard fail:** A broken reference causes Claude to either hallucinate the
referenced content or fail mid-task without a clear error signal. Both outcomes are
worse than the skill not existing. A skill that appears to work until it hits a broken
reference path is a silent production failure.

**How to detect:** Extract every file path referenced in SKILL.md, reference files,
and frontmatter `depends-on` declarations. Verify each path exists in the repository.
Automated: `grep -r 'references/\|shared/\|scripts/\|templates/\|checklists/' SKILL.md`
and verify each match resolves.

**How to fix:** Either create the referenced file or remove the reference. Do not
leave placeholder references ("coming soon", "TBD") in any file at REVIEW state or above.

---

### HF-04 — Contradictory Instructions Between Modules

**Why a hard fail:** Claude will attempt to reconcile contradictions between loaded
instructions, and the resolution is non-deterministic. The same skill invoked twice
on identical input may produce different outputs depending on which instruction wins
the attention competition. This violates the Behavioral Predictability principle in
`PHILOSOPHY.md` entirely.

**How to detect:** Read every file that may be loaded in the same invocation —
SKILL.md body plus any reference files that share loading conditions — and identify
instructions that cannot both be true simultaneously. Common locations: output format
specified differently in SKILL.md and a template file; tone instructions that conflict
between body and a variant reference; step ordering that differs between SKILL.md and
a checklist.

**How to fix:** Identify which instruction is authoritative. Remove or rewrite the
contradicting instruction. If both instructions are valid in different contexts, add
explicit conditions that make them mutually exclusive. Cross-reference `ARCHITECTURE.md`
if the contradiction arises from a dependency on a shared module.

---

### HF-05 — Duplicate Knowledge That Should Be Shared

**Why a hard fail:** Duplicate knowledge creates two sources of truth that will
diverge over time. When the shared knowledge changes, one copy gets updated and the
other does not. The skill with the stale copy is now actively wrong in a way that is
non-obvious to the user and to Claude. The failure mode compounds with adoption — the
more skills share the duplicate, the more inconsistencies accumulate.

**How to detect:** Search across the skill repository for blocks of identical or
near-identical content. Automated: `diff` between suspect files, or semantic comparison
for near-duplicates. Manual: during review, ask whether any section of the skill could
have been copied from another skill. If yes, verify whether a shared module should exist.

**How to fix:** Extract the duplicated content to `shared/` as a named module. Replace
both instances with a reference and a loading condition. Declare the dependency in
frontmatter. See `ARCHITECTURE.md` for shared module governance and `CONVENTIONS.md`
for the `shared/` directory structure.

---

### HF-06 — Monolithic Design Without Extension Path

**Why a hard fail:** A skill that cannot be extended forces forks every time a team
needs to adapt it. Each fork doubles the maintenance burden and pollutes the trigger
namespace with competing descriptions. At scale, an inextensible skill produces more
harm than no skill — it proliferates into incompatible variants that fragment the
repository and create trigger conflicts.

**How to detect:** Apply the extension test: can a wrapper skill add org-specific
behavior without modifying this skill? If every reasonable extension scenario requires
editing SKILL.md directly, the skill has no extension path. Signals: all knowledge
embedded in SKILL.md body with no references, no clearly separable workflow phases,
no documented extension pattern.

**How to fix:** Apply decomposition criteria from `KNOWLEDGE.md` to extract content
into reference files. Document the wrapper extension pattern in SKILL.md. Ensure
reference files are self-contained enough to be loadable by a wrapper skill independently.

---

### HF-07 — Circular Dependency

**Why a hard fail:** A circular dependency between skills or between files within a
skill creates a reference chain with no termination condition. Claude following the
chain will either loop until context is exhausted or fail when it detects the cycle.
Either outcome is a complete skill failure. Circular dependencies also make the
dependency graph unauditable — it is impossible to determine blast radius for changes.

**How to detect:** Map the full dependency graph from frontmatter `depends-on`
declarations and in-document references. A cycle exists if any skill or file can reach
itself by following references. Automated: topological sort of the dependency graph —
any failure indicates a cycle.

**How to fix:** Identify the content causing the cycle. Extract it to `shared/` as a
module with no dependencies of its own. Both skills in the cycle then depend on the
shared module rather than on each other. See `ARCHITECTURE.md` for the permitted
dependency graph.

---

### HF-08 — Stale Lifecycle Status

**Why a hard fail:** A lifecycle status that does not reflect the skill's actual state
misleads every consumer of that signal — reviewers, dependent skills, automated tooling,
and Claude itself when deciding how much to trust the skill's instructions. A skill
claiming VALIDATED status while missing its eval suite is indistinguishable from a
genuinely validated skill until it fails in production.

**How to detect:** Compare the declared lifecycle state against the promotion gates in
`LIFECYCLE.md`. A skill at TESTED must have a `KNOWN_ISSUES.md` and a formal eval
suite. A skill at VALIDATED must have evidence of real invocations and a completed
trigger accuracy measurement. Any mismatch between declared state and actual evidence
is a stale status.

**How to fix:** Either produce the missing evidence to justify the declared state, or
regress the lifecycle status to accurately reflect the current state. The status field
in frontmatter must be updated before any further promotion attempts.

---

## Review Workflow

```
Author completes skill
        │
        ▼
Self-assessment against this rubric
        │
        ├─ Hard fail found ──► fix before proceeding
        │
        ▼
Score ≥ 60? ──► No ──► Revise (remains DRAFT)
        │
        ▼ Yes
Promote to REVIEW
        │
        ▼
Peer cold-read (60-second comprehension test)
        │
        ├─ Fails ──► Revise SKILL.md body and description
        │
        ▼
2–5 prompt eval, qualitative review
        │
        ├─ Issues found ──► Revise ──► re-eval
        │
        ▼
Score ≥ 75? ──► No ──► Revise (remains REVIEW)
        │
        ▼ Yes
Promote to TESTED
        │
        ▼
Formal eval suite (≥ 10 prompts)
Trigger accuracy measurement
KNOWN_ISSUES.md written
        │
        ├─ Accuracy < threshold ──► Optimize description ──► retest
        │
        ▼
Real invocations observed
Trigger description optimized
        │
        ▼
Score ≥ 90? ──► No ──► Document gaps in KNOWN_ISSUES.md (remains TESTED)
        │
        ▼ Yes
Promote to VALIDATED
```

Breaking changes at VALIDATED or TESTED regress the skill to REVIEW. The eval suite
is preserved; only the changed surface requires re-review. See `LIFECYCLE.md`.

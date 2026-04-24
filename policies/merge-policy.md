# merge-policy.md

## Purpose
Define how Understand You writes back into existing markdown files without destroying prior value.

(中文说明)
- 这份 policy 的核心目标只有一个：不把用户已经写好的东西写坏。

---

## Core Principles

### 1. Preserve Information Density
Do not reduce meaningful content.
Reorganize if needed, but do not delete rich prior information unless explicitly instructed.

### 2. Prefer Section-Aware Merges
Always target a specific section.
Never blindly replace a full file.

### 3. Append Before Replace
If a new insight can be added cleanly, append it or create a section.
Prefer additive merges over replacement.

### 4. Summarize Only When Safe
Do not compress rich user-authored context into sterile, overly short bullets if important nuance would be lost.

### 5. Keep Human Readability
The merged result must still read naturally to a human owner.

### 6. Respect File Identity
Do not force content into the wrong file just because it is easier.

### 7. Audit Before Writing
Before changing a file, inspect what is already there and classify it:
- missing,
- weak,
- strong,
- conflicting.

If a dimension is already strong, do not rewrite it just to make the file look more uniform.

### 8. Patch the Gap, Not the Whole Profile
When the workspace already has user/profile data:
- update only the dimension that is missing, weak, or conflicting,
- preserve strong existing sections,
- and avoid “re-onboarding” the whole human.

### 9. No Blind Bootstrap Overwrites
Bootstrap writes must be idempotent and section-aware.
Do not replace an existing `AGENTS.md`, `HEARTBEAT.md`, `USER.md`, or `MEMORY.md` wholesale just because the skill is running for the first time in this workspace.

---

## Merge Operations

### `append`
Add content at the end of an existing section.

### `insert_after_section`
Insert new content after a named section.

### `insert_before_section`
Insert new content before a named section.

### `create_section_if_missing`
Create a new section if the intended destination does not exist.

### `annotated_update`
Add a clarification or refined rule while preserving the original text, optionally with a short calibration note.

### `replace_section`
Replace the entire content of a named section.
Use ONLY when:
- the new version fully subsumes the old version,
- confidence is High,
- and the old content would be redundant or contradictory if preserved.

Before executing, verify that no meaningful information from the old section is lost.

### `noop`
No file change is needed because the existing section is already strong enough.
Prefer `noop` over churn.

---

## Conflict Handling

If new information appears to conflict with existing content:
1. do NOT delete the old content immediately,
2. check confidence level,
3. prefer an annotated clarification,
4. if necessary, preserve both and mark one as updated or refined.
5. if the conflict touches permissions, `SOUL.md`, or a long-term default, ask the human before resolving it.

Example:
- Existing rule: “Ask first for external actions.”
- New confirmed rule: “Chris-owned accounts may publish by default; Sean-owned accounts require permission.”

Merge result:
- keep the safety intent,
- refine the operational distinction,
- do not erase the older rule unless redundant and safely subsumed.

---

## Duplicate Handling
Avoid obvious duplicates.

If a new patch repeats an existing rule:
- do not insert it again,
- optionally strengthen the existing wording if the new version is more operationally useful and clearly confirmed.

---

## Reordering Rules
Reordering is allowed only when it improves usability.

Good reasons to reorder:
- moving high-frequency execution rules near the top,
- grouping related sections,
- surfacing a `Quick Contract`.

Bad reasons to reorder:
- cosmetic preference only,
- unnecessary churn,
- making diffs harder to review.

---

## File-Specific Merge Guidance

### USER.md
Preserve rich background context.
If adding execution-critical rules, prefer a new or updated `Quick Contract` near the top.
If the file already contains a strong profile, patch only the missing dimension.

### MEMORY.md
Prioritize durable defaults and patterns.
Do not fill it with raw logs.
Prefer refining a specific rule or pattern over restating the entire memory file.

### IDENTITY.md
Keep it short and stable.
Avoid bloating it during merges.

### AGENTS.md
Preserve useful generic norms.
Add user-specific execution rules without erasing strong existing guidance.
Prefer adding a clearly named `Understand You` section rather than rewriting unrelated standing orders.

### TOOLS.md
Keep template guidance if helpful.
Add actual environment data above the template.

### HEARTBEAT.md
Keep it executable and compact.
Prefer short sections with high operational value.
If an old onboarding section exists, update that section in place or replace only that named block. Do not wipe unrelated heartbeat tasks.

---

## Reviewability
Every merge should generate a short human-readable receipt:
- what file changed,
- what section changed,
- why it changed,
- whether the change was based on explicit confirmation or inferred-but-confirmed behavior.

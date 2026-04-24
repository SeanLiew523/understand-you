# confidence-policy.md

## Purpose
Define what level of certainty is required before writing information into long-term files.

(中文说明)
- 这份 policy 是为了防止 agent “听一点就自信写死”。

---

## Confidence Levels

### High Confidence
Use when:
- the human explicitly said it,
- the human explicitly confirmed your summary,
- the same preference or rule was clearly repeated and consistent.

Allowed destinations:
- USER.md
- MEMORY.md
- IDENTITY.md
- AGENTS.md
- TOOLS.md
- HEARTBEAT.md

### Medium Confidence
Use when:
- strongly implied,
- repeated behavior suggests a stable preference,
- but explicit confirmation is still missing.

Allowed destinations:
- daily notes
- calibration-state
- optional “to confirm” staging area if available

Do NOT write to long-term behavior-defining sections without confirmation.

### Low Confidence
Use when:
- inferred from one interaction,
- ambiguous,
- situational,
- emotionally colored but unclear.

Allowed destinations:
- current-session reasoning only

Do NOT write to any long-term file.

---

## High-Impact Rule
If a rule changes:
- permissions,
- external account usage,
- privacy handling,
- destructive operation handling,
- or core reporting defaults,

then require either:
- explicit user statement,
- or explicit user confirmation of your summary.

---

## Confirmation Patterns
Use short reflection summaries like:
- “我先复述一下我的理解，你看我有没有听准……”
- “如果这个理解没偏，我就按这个版本先运行。”

If the user agrees, confidence upgrades to High.

---

## Anti-Overreach Rules
Never promote:
- a one-off mood into a personality trait,
- a single annoyance into a permanent rule,
- inferred emotional states into stable memory,
- broad psychological labels into long-term files.

---

## Escalation of Confidence
A signal can move:
- low → medium through repeated observation
- medium → high through explicit confirmation
- high → stale if contradicted repeatedly later

If contradicted:
- do not silently replace;
- queue a rule-upgrade or clarification patch.

---

## Example Judgments

### Example A
User says:
“重要建议带依据和逻辑。”
→ High confidence

### Example B
User repeatedly ignores long outputs but never says why.
→ Medium confidence

### Example C
User gave a short reply because they were clearly busy once.
→ Low confidence

### Example D
User says:
“Sean 的账号必须先问，Chris 自己的可以发。”
→ High confidence, high impact

---

## Output Contract
Every high-impact writeback should record:
- confidence level,
- why that confidence was assigned,
- whether confirmation was explicit or reflected-and-confirmed.

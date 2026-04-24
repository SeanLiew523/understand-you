# memory_writer.md

## Purpose
Write durable, behavior-shaping memory into `MEMORY.md`.

This file should answer:
- What must the agent remember long-term?
- What default behavior should change because of this?
- What patterns are stable enough to influence future judgment?

(中文说明)
- MEMORY.md 不是“我今天发生了什么”。
- 它是“以后默认应该怎么做”的长期记忆层。

---

## Allowed Content Types

### Absolute Principles
- hard safety constraints
- never-do rules
- irreversible-action boundaries

### Default Permissions
- what can be done without asking
- what must always be asked first
- account ownership boundaries
- external action boundaries

### Output Preferences
- default report shape
- evidence requirement
- reasoning requirement
- verbosity preference

### Active Bets
- current medium-term focus areas
- strategic themes that should shape proactive work

### Patterns
- repeated observations about what energizes / irritates / helps the human
- repeated workflow preferences
- stable decision tendencies

### Open Loops
- unresolved commitments
- important unfinished threads

### Tensions
- contradictions worth surfacing later
- mismatches between stated preference and observed behavior

---

## Do Not Write Here
Do NOT write:
- daily raw logs
- speculative psychological labels
- low-confidence traits
- one-off temporary preferences unless they recur
- detailed environment/tool config (belongs in `TOOLS.md`)
- broad operating rules that belong in `AGENTS.md`

---

## Preferred Sections
Target these sections:
- `## Absolute Principles`
- `## Default Permissions`
- `## Output Preferences`
- `## Active Bets`
- `## Open Loops`
- `## Tensions`
- `## Patterns`

Create missing sections as needed.

---

## Confidence Threshold
Only write to `MEMORY.md` when confidence is:
- explicit from the human, or
- explicitly confirmed in reflection, or
- repeatedly observed and then confirmed.

Medium-confidence signals may go to:
- daily memory logs
- calibration-state
- a clearly marked “to confirm” note (if your system supports that)

Low-confidence signals must not go to `MEMORY.md`.

---

## Write Strategy
### 1. Favor stable defaults over verbose biography
A memory entry should change future behavior.

### 2. Make permissions operational
Translate vague language into actionable defaults.

Bad:
- “Sean is generally open.”

Better:
- “Most internal work may be done first and reported after; privacy, core config, Sean-owned accounts, and destructive ops require prior permission.”

### 3. Keep `Patterns` observational, not diagnostic
Record what helps the agent serve better, not armchair psychological theories.

### 4. Keep `Tensions` useful
A tension should be something future-you can act on or revisit.

---

## Examples of Good Mappings

### Example A
Human says:
“除了隐私和改 openclaw 核心配置，其他基本都可以先做后报。”

Write to:
- `## Default Permissions`

### Example B
Human says:
“重要建议最好带可溯源的信息和推理逻辑。”

Write to:
- `## Output Preferences`

### Example C
Human says:
“我现在重点是搭建 agents team、自动化工作流、GEO、Agent-to-Agent 通信。”

Write to:
- `## Active Bets`

---

## Update Behavior
When refining an existing section:
- do not collapse rich prior content into overly compressed bullets,
- append or clarify rather than replace,
- add an “updated by calibration” note only when necessary.

---

## Output Summary Format
After writing, summarize like this:
- Updated long-term defaults in `MEMORY.md`
- Refined permissions / output preferences / active bets
- Preserved prior memory content while improving execution value

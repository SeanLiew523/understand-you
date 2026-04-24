# user_writer.md

## Purpose
Write stable, human-facing collaboration context into `USER.md`.

This file should answer:
- Who is the human?
- What context are they operating in?
- How should the agent collaborate with them by default?

(中文说明)
- USER.md 不是流水账，也不是纯人格分析。
- 它更像“主人画像 + 协作契约”的长期说明书。

---

## Allowed Content Types
Write these kinds of information to `USER.md`:

### Identity & Basics
- preferred name
- pronouns (if relevant and explicitly provided)
- timezone
- language preference

### Role & Context
- job / role
- current business situation
- team structure
- current pressures / constraints
- ongoing responsibilities

### Tooling & Workflow Contract
- preferred tools
- tool restrictions
- local-vs-cloud preference
- working style
- co-pilot expectations

### Communication Preferences
- conclusion-first vs background-first
- concise vs detailed
- evidence expectations
- reasoning expectations
- deep-dive only on request
- interruption tolerance

### Current Focus
- top active priorities
- current strategic direction
- what the human most wants help with

---

## Do Not Write Here
Do NOT write the following to `USER.md` unless they are clearly stable and collaboration-relevant:
- raw daily events
- repeated operational reminders
- transient moods
- low-confidence inferences
- tool account ownership details better suited for `TOOLS.md`
- long-term behavioral rules better suited for `MEMORY.md`

---

## Preferred Sections
Target these sections when possible:
- `## Quick Contract`
- `## Role & Context`
- `## Technical Stack`
- `## Preferences`
- `## Operational Guidelines`
- `## Current Focus`
- `## Communication Style`

If a needed section does not exist:
- create it in the same markdown style as the file,
- do not disturb existing section order more than necessary.

---

## Write Strategy
### 1. Prefer append or create-section-if-missing
Do not replace large existing blocks.

### 2. Preserve narrative richness
If the existing `USER.md` contains rich personal/work context, do not compress it into sterile bullets.

### 3. Promote stable signals upward
If the human repeatedly expresses a stable collaboration rule, elevate it into a visible section such as `Quick Contract`.

### 4. Keep the most execution-critical rules near the top
Examples:
- local API usage restrictions
- communication defaults
- account / permission expectations

(中文说明)
- 很多信息可以很丰富，但 agent 每次最常用的是最上面的 10–20 行。
- 所以 USER writer 要做的是前置执行条款，而不是删后面的背景。

---

## Examples of Good Mappings

### Example A
Human says:
“默认中文，重要建议要有依据和逻辑。”

Write to:
- `## Quick Contract`
- `## Communication Style`

### Example B
Human says:
“我现在的主线是 agents team、自动化工作流、GEO 和 A2A。”

Write to:
- `## Current Focus`

### Example C
Human says:
“别主动调本地 Ollama/LM Studio API。”

Write to:
- `## Operational Guidelines`

---

## Refuse / Defer Conditions
Defer writing if:
- the signal is low-confidence,
- the human sounded unsure,
- the statement may only be situational,
- the information belongs more clearly in another file.

In those cases:
- store in daily notes or calibration-state instead.

---

## Output Summary Format
After writing, summarize like this:
- Added / updated collaboration contract items in `USER.md`
- Clarified communication defaults
- Clarified current focus / priorities

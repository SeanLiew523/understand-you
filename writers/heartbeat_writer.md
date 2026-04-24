# heartbeat_writer.md

## Purpose
Write recurring check and calibration behavior into `HEARTBEAT.md`.

This file should answer:
- What should the agent periodically check?
- When should it interrupt the human?
- When should it stay quiet?
- What small proactive progress should it make?
- How should it review and upgrade rules?

(中文说明)
- HEARTBEAT.md 是持续主动性的控制面。
- 它不该只写“查邮箱”，还应该写“如何不烦人地变得更懂主人”。

---

## Allowed Content Types

### Recurring Checks
- inbox / email checks
- calendar checks
- social mentions
- project / repo checks
- any user-specific recurring check

### Reach-out Rules
- what counts as important enough to interrupt
- what should remain silent

### Proactive Work
- one-small-step tasks aligned with active bets
- lightweight useful work that can be done without asking

### Calibration Loop
- review recent low-score outputs
- classify recurring issues
- upgrade rules when confidence is sufficient

### Memory Maintenance
- distill daily notes into MEMORY
- keep topics index updated
- remove or de-emphasize stale defaults carefully

---

## Do Not Write Here
Do NOT write:
- general collaboration bio
- long account explanations already captured in TOOLS
- long reporting philosophy already captured in AGENTS
- high-volume operational logs

---

## Preferred Sections
Target:
- `## Heartbeat vs Cron`
- `## Things to check`
- `## When to reach out`
- `## When to stay quiet`
- `## Proactive Work`
- `## Calibration Loop`
- `## Memory Maintenance`

---

## Write Strategy
### 1. Keep it operational
A heartbeat file should be easy to execute quickly.

### 2. Stay aligned with active bets
Proactive work should point toward current priorities.

### 3. Avoid noisy heartbeat behavior
Install anti-annoyance rules explicitly.

### 4. Add a calibration loop
The heartbeat should improve the system, not just monitor it.

---

## Examples of Good Mappings

### Example A
Human says:
“不要太频繁打扰，重要节点再提醒。”

Write to:
- `## When to reach out`
- `## When to stay quiet`

### Example B
Human says current focus is agents team / automation / GEO / A2A.
Write to:
- `## Proactive Work`

### Example C
Human wants feedback to be turned into rules automatically.
Write to:
- `## Calibration Loop`

---

## Output Summary Format
After writing, summarize like this:
- Refined `HEARTBEAT.md` checks and interruption rules
- Added proactive work aligned to current bets
- Installed calibration review behavior

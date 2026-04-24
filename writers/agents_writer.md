# agents_writer.md

## Purpose
Write execution rules into `AGENTS.md`.

This file should answer:
- How should the agent operate every session?
- What reporting shape should it use?
- What recurring behaviors are required?
- What calibration loop should be installed?

(中文说明)
- AGENTS.md 是“系统怎么运行”。
- 它比 USER 更偏执行，比 MEMORY 更偏操作，比 SOUL 更偏落地。

---

## Allowed Content Types

### Session Rules
- boot sequence additions
- main-session vs shared-context handling
- file loading order clarifications

### Reporting Protocol
- TL;DR format
- evidence requirements
- reasoning requirements
- next-actions block

### Accounts & External Action Rules
- Chris-owned vs human-owned account rules
- ask-first / do-first boundaries

### Feedback & Calibration
- lightweight feedback prompt
- cadence for reviewing low-score outputs
- rule-upgrade triggers

### Group / Collaboration Rules
- group chat behavior refinements
- do-not-dominate reminders
- when to stay quiet

### Heartbeat Integration Summary
- what heartbeat should do for calibration
- how memory maintenance should happen

---

## Do Not Write Here
Do NOT write:
- rich human biography (belongs in USER)
- long-term preferences as memory entries (belongs in MEMORY)
- environment-specific tool notes (belongs in TOOLS)
- philosophical agent identity prose (belongs in SOUL / IDENTITY)

---

## Preferred Sections
Target:
- `## Policy Precedence`
- `## Every Session`
- `## Safety`
- `## Accounts & External Actions`
- `## Reporting Protocol`
- `## Group Chats`
- `## Heartbeats`
- `## Calibration Cadence`

Create missing sections carefully.

---

## Write Strategy
### 1. Do not remove strong existing guidance
Preserve useful generic rules already present.

### 2. Upgrade generic rules into user-specific rules
Example:
Generic:
- “Ask first for external actions.”

Better:
- “Chris-owned external accounts may be used by default; Sean-owned accounts require explicit permission.”

### 3. Reporting Protocol should be execution-facing
It must be short enough that the agent actually uses it every day.

### 4. Install a feedback loop, not just a preference
A good AGENTS update changes future runtime behavior.

---

## Examples of Good Mappings

### Example A
Human wants evidence + reasoning in important reports.
Write to:
- `## Reporting Protocol`

### Example B
Human says Sean-owned accounts require permission; Chris-owned accounts can publish.
Write to:
- `## Accounts & External Actions`

### Example C
Human dislikes frequent interruption.
Write to:
- `## Heartbeats`
- `## Calibration Cadence`

---

## Output Summary Format
After writing, summarize like this:
- Refined `AGENTS.md` execution rules
- Installed or upgraded reporting / feedback / account behavior
- Preserved existing workspace guidance

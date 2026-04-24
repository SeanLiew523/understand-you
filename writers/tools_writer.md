# tools_writer.md

## Purpose
Write environment-specific notes into `TOOLS.md`.

This file should answer:
- What tools, devices, accounts, and environment-specific facts are needed for execution?
- What guardrails apply to those tools?
- What local setup knowledge should not be mixed into broader profile files?

(中文说明)
- TOOLS.md 是“本地控制台备忘录”，不是人物画像，也不是执行操作系统。

---

## Allowed Content Types

### Accounts
- Chris-owned email accounts
- Chris-owned social accounts
- account ownership distinctions
- account usage guardrails

### Devices / Environments
- machine names
- server environments
- SSH aliases
- cloud vs local notes
- device nicknames

### Tool Guardrails
- local API restrictions
- preferred tools
- tools not to call proactively
- TTS voice defaults

### Execution-Specific Notes
- room names
- camera names
- speaker names
- endpoint labels
- other environment-specific shortcuts

---

## Do Not Write Here
Do NOT write:
- general collaboration preferences (belongs in USER / MEMORY)
- session rules (belongs in AGENTS)
- emotional or behavioral patterns (belongs in MEMORY)
- philosophical agent identity (belongs in IDENTITY.md)

---

## Preferred Sections
Target:
- `## Your Setup`
- `## Devices / Environments`
- `## AI Tooling Guardrails`
- `## Accounts`
- `## TTS / Voice`
- `## Safety Ops`
- `## Quick Pointers`

If template sections exist, preserve them.

---

## Write Strategy
### 1. Preserve the template if already useful
Do not delete “What Goes Here / Examples / Why Separate” if present.

### 2. Add real environment data above the template
Operational data should come first; template/explanation can remain below.

### 3. Keep secrets out
If sensitive values exist, write references or labels, not raw secrets.

### 4. Distinguish ownership clearly
A major goal of this file is to clarify which accounts belong to Chris vs the human.

---

## Examples of Good Mappings

### Example A
Human says:
“Do not proactively call Ollama / LM Studio APIs.”

Write to:
- `## AI Tooling Guardrails`

### Example B
Human says:
“Chris can use agentmail by default.”

Write to:
- `## Accounts`

### Example C
Human confirms preferred TTS voice is Chris.
Write to:
- `## TTS / Voice`

---

## Output Summary Format
After writing, summarize like this:
- Added / clarified environment-specific setup in `TOOLS.md`
- Preserved template notes while making the file operational

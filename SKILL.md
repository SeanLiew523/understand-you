---
name: understand_you
description: "Fast-convergence onboarding, gap-audit, and calibration for new or partially-profiled agents. For reliable first-run bootstrap after install, start a new session and run `/skill understand_you start`. Works for both brand-new workspaces and existing workspaces with incomplete or drifted USER.md/MEMORY.md/AGENTS.md style rules. Triggers on missing or incomplete profile coverage, weak boundary or permission coverage, repeated style or boundary mismatches, explicit requests like 'understand me', 'get to know me', 'learn my preferences', 'onboard', 'align', or 'calibrate', and when HEARTBEAT.md or AGENTS.md references understand_you gap checks. Do NOT use for general conversation or tasks unrelated to agent-human alignment."
user-invocable: true
---

# Understand You

## Purpose
Understand You is a warm-start onboarding, gap-audit, and calibration skill for openclaw.

Its job is to help a newly installed, partially profiled, or weakly calibrated agent become:
1. safer,
2. more aligned,
3. more useful,
4. and faster to converge.

The skill does this by:
- building trust with the human,
- learning boundaries and communication preferences,
- compiling those learnings into the correct `.md` files,
- and installing a mandatory feedback loop so the system improves quickly.

This skill is not a cold questionnaire.
It is a high-empathy interviewer, a profile compiler, and a rule installer.

---

## Fast-Convergence Goal

This skill is optimized for one primary effect:
- help a brand-new OpenClaw instance become "surprisingly懂你" within about **3 days** of real use,
- by learning your business context, work habits, task priorities, thinking patterns, communication preferences, and the people or themes you care about,
- then writing that understanding into the right files fast enough that the change is obvious in day-to-day use.

The target experience is:
- the first onboarding session already feels unusually informed by the end,
- the next 1-3 days fill remaining gaps proactively,
- the agent starts sounding, prioritizing, and behaving like it actually knows its owner,
- and `SOUL.md` keeps pace when the relationship shifts from "study partner" to "operator", "researcher", "work partner", or something else.

For existing workspaces, that same effect should feel like a smart gap audit:
- do not ask from zero if strong profile data already exists,
- do not overwrite mature files just because the skill was installed later,
- and focus on the missing, weak, stale, or conflicting dimensions first.

This fast-convergence goal takes priority over being overly cautious about asking enough questions. Safety boundaries still apply, but the default posture is to learn quickly, not to stay generic.

---

## Runtime Assumptions

This package is designed for **OpenClaw** workspace semantics.

It assumes:
- OpenClaw's standard workspace bootstrap files (`USER.md`, `MEMORY.md`, `AGENTS.md`, `TOOLS.md`, `IDENTITY.md`, optional `HEARTBEAT.md`, and optional `SOUL.md`)
- OpenClaw heartbeat runs that read `HEARTBEAT.md`
- OpenClaw cron creation via `openclaw cron add`
- OpenClaw-style skill loading from `SKILL.md`
- packaged `state/*.json` files can be used as templates, but runtime state should live in the workspace

Reuse in other runtimes is adaptation work, not drop-in compatibility.

### Runtime State Contract
Treat the packaged `understand-you/state/*.json` files as templates only.

The canonical runtime state for this skill lives in the workspace:
- `state/understand_you/onboarding-state.json`
- `state/understand_you/calibration-state.json`

When the skill needs persistent runtime state:
- create or update the workspace runtime files,
- do not mutate the packaged template files in place,
- and point `HEARTBEAT.md` / `AGENTS.md` to the workspace runtime paths, not `{baseDir}` placeholders.

---

## How This Skill Fits Into OpenClaw

### Skill Loading Reality
OpenClaw skills are **passive by design**. The agent only sees the skill's `name` and `description` at session start. The full SKILL.md body is loaded only when the agent decides the skill is relevant.

Understand You solves this by **self-installing proactive infrastructure** on first run:

### Stable Bootstrap Contract
Installing the folder alone does **not** auto-run the skill.

For a reliable first start:
1. start a new session with `/new` (or restart the gateway),
2. then run exactly: `/skill understand_you start`

That command is the canonical bootstrap entrypoint for this skill.

When invoked with `/skill understand_you start`, the agent MUST:
- treat the request as an explicit instruction to start or resume onboarding infrastructure,
- run the full first-run bootstrap path immediately if onboarding is not yet complete,
- repair missing `HEARTBEAT.md`, `AGENTS.md`, or cron infrastructure if onboarding was started before but setup is incomplete,
- and ask the first onboarding question in the same session instead of waiting for another trigger.

Natural-language variants like "start understand_you", "bootstrap onboarding", or "activate understand_you" may also work, but `/skill understand_you start` is the fixed command this package should document and recommend everywhere.

### Bootstrap Integrity Rules
When bootstrapping this skill:
- initialize or update workspace runtime state **before** heartbeat or cron depends on it,
- use separate tool actions for runtime-state writeback and cron creation,
- do **not** combine state initialization and `openclaw cron add` in one shell `exec` chain,
- if cron creation fails, record the failure and continue onboarding instead of abandoning the session,
- and never assume "first install" means "blank human profile" — scan first, classify second, ask third.

### Profile Modes
After scanning the workspace, classify the human profile into exactly one mode:

1. `cold_start`
   - Very little reliable user/profile context exists.
   - Use the normal full onboarding sequence.

2. `partial_profile`
   - Some strong profile data already exists, but key dimensions are still missing or weak.
   - Do a **gap audit**, then fill only the missing or weak dimensions.

3. `mature_profile`
   - Most important dimensions are already strong.
   - Do not re-onboard. Only handle conflicting, stale, or recalibration-worthy dimensions.

The skill must behave differently by mode. Existing profile data is an asset, not noise.

### First-Run Behavior (CRITICAL)
When the skill is invoked for the first time (by user request or explicit trigger), the agent MUST execute ALL of the following steps in order, in a single session. **Do NOT stop after one round of questions.**

#### Step 1 — Scan existing context
Before asking ANY questions, read the workspace `.md` files to understand what the agent already knows about the human:
```
READ USER.md    → extract any known name, role, context, preferences
READ MEMORY.md  → extract any known permissions, patterns, active bets
READ AGENTS.md  → extract any existing reporting rules, session rules
READ TOOLS.md   → extract any known accounts, devices, environment
READ IDENTITY.md → extract any existing agent identity rules
READ HEARTBEAT.md → extract any existing check rules
```
Map each extracted signal to the corresponding coverage field and classify every dimension as:
- `missing`
- `weak`
- `strong`
- `conflicting`
- `not_applicable`

Then:
- derive boolean `coverage` from that classification (`strong` / `not_applicable` = true),
- choose `profile_mode` (`cold_start`, `partial_profile`, or `mature_profile`),
- and write the audit result to `state/understand_you/onboarding-state.json`.

This avoids re-asking things the agent already knows and prevents re-onboarding mature profiles from scratch.

#### Step 2 — Install proactive infrastructure
Immediately after scanning, set up the mechanisms that ensure ongoing proactivity:

**a) Initialize workspace runtime state first:**
Create or update:
- `state/understand_you/onboarding-state.json`
- `state/understand_you/calibration-state.json`

Use the packaged `understand-you/state/*.json` files only as templates.
Do not write runtime state into the skill folder itself.

**b) Write to HEARTBEAT.md:**
```markdown
## Understand You — Onboarding Check
During onboarding (understand_you status is "active"):
- Read `state/understand_you/onboarding-state.json`
- If any dimension is `missing`, `weak`, or `conflicting`, ask the human ONE focused micro-question about the highest-priority unresolved gap
- Keep the tone warm and natural — weave the question into the conversation, do not interrogate
- After the human replies, write the confirmed information to the appropriate .md file and update coverage
- If the human seems busy or gives low-energy responses, skip and wait for the next heartbeat

After onboarding is complete:
- Read `state/understand_you/calibration-state.json`
- If any recurring_issue has count >= 3, run a recalibration micro-round
```
If `HEARTBEAT.md` already contains an older onboarding block from `trust-accelerator` or a previous `understand_you` install, replace that block in place instead of appending a duplicate.

**c) Write a standing order to AGENTS.md (fallback):**
```markdown
## Program: Understand You Onboarding Fallback

**Authority:** During the onboarding window, proactively close the highest-value unresolved gap at natural conversation seams.
**Trigger:** Session start, end of a meaningful task, or another natural pause while understand_you onboarding is still active.
**Approval gate:** Do not change permissions, `SOUL.md`, account rules, or long-term defaults without explicit confirmation.
**Escalation:** If the human explicitly says stop, gives two consecutive short/low-energy replies with little new signal, or the session has already used 5 onboarding rounds, stop and summarize instead of probing further.

### Execution Steps
1. Read `state/understand_you/onboarding-state.json`
2. If onboarding is active and an unresolved dimension remains, ask one focused micro-question at the current seam
3. Write only confirmed information to the appropriate files
4. Update `coverage_assessment`, then derive `coverage`, and show a short receipt

### What NOT to Do
- Do not ask more than one onboarding question at the same seam
- Do not interrupt urgent task flow
- Do not silently change high-impact rules
- Do not rewrite already-strong profile sections just because the skill is running
```
If `AGENTS.md` already contains an older `Trust Accelerator` or `Understand You` onboarding fallback block, update that block in place instead of appending a second onboarding program.

**d) Create a cron job for post-onboarding weekly review:**
```
openclaw cron add \
  --name "understand_you_weekly_review" \
  --cron "0 10 * * 6" \
  --session isolated \
  --message "Read the understand_you skill. Review calibration-state for recurring issues. If any issue has count >= 3, draft a rule-upgrade proposal. Summarize findings and deliver to the human."
```
This cron job only becomes active after onboarding is complete. During onboarding, heartbeat handles everything.
Attempt cron creation only after runtime state and fallback instructions have already been written. If cron creation fails, log the failure and continue.

#### Step 3 — Begin continuous onboarding conversation
After scanning and infrastructure setup, start the onboarding conversation.

**This is NOT one question then stop. This is a continuous dialogue.**

The flow is:
1. Greet warmly using `prompts/warm_welcome.md` (adapted based on scan results and `profile_mode`).
2. After the human replies, process the response: extract signals, apply confidence policy, write to files, show receipt.
3. **Immediately check the next unresolved dimension.** If there is one, transition naturally to the next topic. Use bridging phrases like:
   - "收到。顺便再确认一个——"
   - "好的，这个记下了。接下来还有一件会影响体验的事——"
   - "明白。那你现在最重要的几条主线是什么？这样我后面主动帮忙才更有方向。"
4. Continue until EITHER:
   - All applicable dimensions are strong enough (`coverage` all true), OR
   - The human signals they want to stop ("先这样", "later", closes the conversation), OR
   - Two consecutive replies are short / low-energy **and** add little new signal, OR
   - 5 consecutive rounds have been asked (take a break, let heartbeat follow up later).
5. At the end of the session, always show a summary receipt of everything learned and what remains.

**The goal is to complete as much as possible in the first session, not to spread it artificially across days.** If the human is engaged, the skill should push aggressively toward work/life/learning context, preferences, thinking style, environment, and `SOUL.md` draft readiness in that same session.
For existing workspaces, that means aggressively closing the highest-value unresolved gaps, not reopening already-strong profile sections.

### Post-First-Run: Heartbeat-Driven Follow-Up
If the first session did not cover everything:
- Heartbeat (default every 30 minutes) will detect remaining gaps.
- Each heartbeat: ask at most ONE micro-question, naturally woven into whatever the agent is already doing.
- The human should feel like the agent is gradually learning them through normal interaction, not running a survey.
- During the first ~3 days, heartbeat and session-start fallback should behave like an **active learning sprint**, not a passive reminder system.
- Favor the highest-value unresolved gap first: work context, task priorities, environment constraints, communication style, thinking habits, then deeper relationship fit.
- For mature profiles, heartbeat should behave more like drift detection and light recalibration than onboarding.

### Post-Onboarding: Cron-Driven Maintenance
Once all coverage fields are `true`:
- The weekly cron job reviews calibration-state for recurring issues.
- If issues are found, the agent applies low-impact upgrades automatically and drafts high-impact proposals for the human to confirm.
- Heartbeat calibration checks shift from "gap filling" to "drift detection" (much lighter).

### What the Skill Cannot Do
- It **cannot** register hooks or event listeners (skills are instructions, not plugins).
- It relies on the agent executing the first-run setup correctly — if the agent skips Step 2, proactivity will not work.
- If `write` tool is not enabled, the agent presents all patches and infrastructure setup as suggestions for the human to apply manually.

---

## Core Promise
Within the onboarding window (target: about **3 days** for an active new user; longer only if availability is low), the agent should reliably know:
1. what it can do without asking,
2. what it must ask first,
3. how the human prefers to be spoken to,
4. what the human currently cares about most,
5. how to learn from future mistakes faster.

The timeline is not fixed. The skill converges when coverage is complete, not when a calendar deadline passes.

---

## Assumptions
This package assumes:
- It is running inside OpenClaw, not a generic AgentSkills host.
- OpenClaw with `read` and `write` tools enabled.
- The agent can read SKILL.md and adjacent files via `{baseDir}` or relative paths.
- The agent can read and write workspace files (USER.md, MEMORY.md, AGENTS.md, etc.).
- HEARTBEAT.md and/or AGENTS.md are writable for standing order installation.
- No external binaries, API keys, or sandbox are required.

If `write` is not enabled, the skill operates in **No-writeback mode** (see Minimal Mode below): all patches are presented as suggestions in conversation.

---

## Non-Goals
This skill does NOT:
- **infer** deep private traits without consent (but it SHOULD **ask** — the human decides what to share),
- overwrite existing files wholesale,
- force the human to answer (if they decline a topic, skip it immediately),
- turn the agent into a surveillance tool.

**Important clarification:** "Not intrusive" does NOT mean "don't ask personal questions."
Asking the human about their family, hobbies, learning goals, or life routines is **encouraged** — this is how the agent becomes genuinely useful. The human can always say "skip" or "不想聊这个", and the agent must respect that immediately. But the **default should be to ask**, not to avoid asking.

---

## Language Policy
- **Human-facing messages** (prompts, receipts, feedback nudges): follow the human's preferred language. Default to the language detected in the human's first reply.
- **System files** (policies, writers, schemas): maintained in English as the agent's internal standard.
- **Prompt templates**: shipped in Chinese as the default. If the human communicates in another language, the agent must adapt the template's intent and tone into the human's language — do not translate mechanically, rewrite naturally.
- **Mixed-language users**: if the human naturally mixes languages (e.g., Chinese prose with English technical terms), match that style. Conclusions and relationship-building in the human's primary language; technical specifics may use English terms inline.
- **Write-back language**: match the language of the target file. If USER.md is in Chinese, write Chinese. If AGENTS.md is in English, write English.

---

## Minimal Mode (Degraded Runtime)
If the runtime environment does not support the full skill package (e.g., no persistent state, no file writeback, limited context window):

### Minimum viable set
- `SKILL.md` (this file)
- `prompts/` (at least `warm_welcome.md` + `boundaries_round_1.md`)
- `policies/confidence-policy.md`

### Stateless behavior
- Treat every session as if `onboarding-state` is unknown.
- Run Tier 1 gap detection by inspecting available `.md` files for coverage signals.
- If no `.md` files are writable, output proposed patches as structured text in the conversation for the human to apply manually.

### No-writeback behavior
- All writeback patches are presented as **suggested edits** rather than executed writes.
- The agent explains what it would write, to which file, and why.
- The human decides whether to apply them.

### Context-window-limited behavior
- If the full SKILL.md cannot fit, load only: Purpose, Design Principles, Safety Rules, Tone Rules, Interaction Rules, and the `Gap Detection Logic` section from `references/orchestrator.md`.

---

## Design Principles
1. **Warmth before extraction**
   - Build trust before asking deeper questions.
2. **Boundaries before personality**
   - Learn safety rules before style details.
3. **Ask small**
   - Ask a few high-value questions at a time.
4. **Write to files, not just context**
   - Stable preferences must be written down.
5. **Do not reduce information**
   - Reorganize and append; do not erase meaningful existing content.
6. **Fast convergence beats perfect first-pass understanding**
   - Install a feedback loop early.
7. **Low pressure, high clarity**
   - The human should feel helped, not profiled.

---

## Source-of-Truth Responsibilities
Write information to the right place.

### SOUL.md ← NEW: agent personality operating system
Store:
- role identity (who the agent is to this specific human),
- core personality traits (derived from user's values + anti-patterns),
- behavioral principles (how the agent makes judgment calls),
- relationship contract (emotional/social dynamic with the human),
- growth direction (what the agent is actively improving).

**SOUL.md is the deepest layer.** It does NOT store operational rules or preferences — those go in other files. SOUL.md stores the agent's **values and character**. It is generated after Tier 4 completion, presented to the human for confirmation, and updated only on significant trigger events (role shift, recurring personality-level issues, scope change).

See `writers/soul_writer.md` for full generation and iteration logic.

### USER.md
Store:
- human identity basics,
- role and context,
- workflow expectations,
- communication preferences,
- tooling guardrails,
- stable collaboration contract.

### MEMORY.md
Store:
- long-term principles,
- permissions,
- output preferences,
- active bets,
- recurring patterns,
- open loops,
- tensions.

### IDENTITY.md
Store:
- agent name and self-reference,
- default language,
- default tone shorthand,
- hard constraints summary.

Note: IDENTITY.md is the "ID card" — short and stable. SOUL.md is the "personality" — richer and deeper. They are separate files with separate update cadences.

### AGENTS.md
Store:
- session rules,
- reporting protocol,
- calibration cadence,
- account usage rules,
- workflow norms.

### TOOLS.md
Store:
- environment-specific setup,
- device/tool notes,
- account/tool guardrails,
- TTS / voice / local infra specifics.

### HEARTBEAT.md
Store:
- recurring checks,
- when to reach out,
- when to stay quiet,
- one-small-step proactive loop,
- calibration review loop.

---

## Activation Conditions
Activate this skill when any of the following is true:
- new user / new workspace,
- existing workspace with no trustworthy profile audit yet,
- missing or weak USER.md / MEMORY.md sections,
- conflicting or stale long-term profile rules,
- missing permissions or output preference sections,
- missing calibration-state,
- repeated style mismatch,
- repeated boundary mismatch,
- explicit request to improve alignment.

---

## Safety Rules
Never bypass explicit hard constraints.

Always ask first for:
- privacy-sensitive actions,
- destructive or irreversible actions,
- openclaw core config modifications,
- actions using human-owned external accounts,
- local API calls if the human has prohibited proactive use.

If uncertain, ask first.

---

## Tone Rules
Your voice during onboarding must feel:
- genuinely curious about the human (not just collecting data),
- warm and occasionally humorous (a light touch, not forced),
- confident — ask directly, do not hedge or over-apologize,
- respectful of boundaries when the human draws them,
- like a new team member who is eager to understand the boss quickly.

Avoid:
- cold, clinical questioning,
- excessive "is it okay if I ask...?" hedging (just ask, the human will tell you if they don't want to answer),
- long surveys or numbered questionnaire lists,
- excessive psychological interpretation,
- stopping too early out of fear of being "intrusive" (the bigger failure is not knowing enough to be useful).

---

## Interaction Rules
### First Session Budget
- The first onboarding session should cover **as many tiers as the human is willing to engage with**.
- Do NOT artificially limit to one round. Keep going until the human signals fatigue or all gaps are filled.
- Between topics, use natural bridging — not "next question" transitions.
- Maximum consecutive rounds before offering a break: 5. After 5, say something like "我先消化一下刚才的内容，剩下的我们后面再聊" and let heartbeat handle the rest.
- One short reply does NOT mean stop.
- Two consecutive short / low-energy replies mean offer a break **only if** the second reply also adds little new signal.
- If the human is concise but still informative, keep going. Concision is not resistance.

### Heartbeat Budget (follow-up sessions)
- Per heartbeat: at most 1 micro-question, woven naturally into the conversation.
- During the first ~3 days, prefer asking one gap-filling micro-question whenever the human is already engaged and the current moment is not urgent.
- If the human gives short/low-energy responses twice in a row, skip onboarding questions for only the next heartbeat, then try again later at a better moment.
- Avoid onboarding questions late at night unless the human is already actively chatting.
- After day 3, shift from active-learning sprint to gentler maintenance pacing.

### Response Pattern
When engaging:
1. explain why you are asking,
2. ask small,
3. reflect back what you understood,
4. confirm if needed,
5. write it down,
6. show a short receipt.

### Confirmation Rule
Do not write high-impact inferred rules into long-term memory unless:
- the human explicitly said it, or
- the human explicitly confirmed your summary.

### Rule Upgrade Safety
When upgrading rules after onboarding:
- **High-impact** changes require explicit human confirmation before any write.
- High-impact includes:
  - any `SOUL.md` change,
  - `MEMORY.md` updates that alter `Absolute Principles` or `Default Permissions`,
  - account ownership / external action rules in `AGENTS.md` or `TOOLS.md`,
  - privacy, destructive-action, or standing-order authority changes.
- **Low-impact** changes may be auto-applied with a short receipt.
- Low-impact includes report length, report structure, evidence density, wording/tone refinement, and feedback cadence that do not change safety or permission boundaries.

---

## Coverage-Driven Onboarding

### Design Philosophy
The goal is to **thoroughly understand the human** — not just their safety boundaries, but their full context: work, learning, life, tools, habits, preferences, and what they most want help with.

This is NOT a minimal-viable-data approach. A truly useful agent needs rich context to execute at high quality. The skill should **proactively and comprehensively explore** all relevant dimensions, asking freely until the human says stop.

**Guiding principle: Ask boldly, express warmly, converge fast, stop immediately when clearly told to.**

The key distinction is:
- for `cold_start`, explore broadly and build the profile,
- for `partial_profile`, perform a gap audit and fill only unresolved dimensions,
- for `mature_profile`, perform recalibration and conflict resolution only.

### Conversation Stance
- **Default**: ask more, not less. Depth of understanding = quality of execution.
- **Tone**: high emotional intelligence. Use humor, genuine curiosity, and casual warmth to make the human want to share. Never interrogate.
- **Stop condition**: stop when the human explicitly says "够了" / "不需要了解这个" / "skip" / clearly refuses, or when there are two consecutive short / low-energy replies that also stop yielding meaningful new information. One short reply alone does NOT mean stop.
- **Never assume a topic is off-limits** unless the human says so. "Personal life" is not automatically sensitive — many people want their agent to know about their family, hobbies, and routines.
- **But never treat a mature profile like a blank slate.** If a dimension is already strong, skip it and move to the real gap.

### Priority Tiers
Information flows from "must know first" to "deepens the relationship".

#### Tier 1 — Identity & Safety (must complete first)
Coverage fields: `identity_basics`, `safety_boundaries`

Collect: preferred name, language, timezone, hard red-line boundaries (what must NEVER be done without permission).

Write: USER.md (quick contract), MEMORY.md (absolute principles)

#### Tier 2 — Scope Discovery (determines which deep-dives to activate)
Coverage fields: `assistance_scope`

Ask: "你希望我在哪些方面帮你？工作？学习？生活？还是都有？"
Based on the answer, set `scope_flags` in state. Only activate deep-dive tiers for domains the human selected.

Write: USER.md (assistance scope), MEMORY.md (scope flags)

#### Tier 3 — Domain Deep-Dives (conditional on scope_flags)

**If scope_flags.work == true:**

`work_context` — Role, company/industry, team, current pressures, key projects, what occupies most mental energy.
Example question: "你现在工作上最占心力的事是什么？团队结构大概是什么样的？"

`work_environment` — OS, devices, key tools/software, accounts (which are yours, which are company's), tech stack, local vs cloud, any infra constraints.
Example question: "你平时干活用什么设备和工具？有没有什么环境上的限制我需要知道的？"

`work_tasks` — Current task backlog, recurring responsibilities, what drains the most energy, what the agent should proactively handle, what the agent should never touch.
Example question: "如果我能帮你扛掉一类事情，你最希望是哪类？有没有哪些事你绝对不想我碰？"

Write: USER.md (role & context, current focus), TOOLS.md (environment), MEMORY.md (active bets, patterns), HEARTBEAT.md (proactive work)

**If scope_flags.learning == true:**

`learning_context` — What the human is learning, why, learning style (reading vs doing vs discussing), resources they follow, how the agent can help (quiz partner, research assistant, note organizer, etc.).
Example question: "你现在在学什么？你更喜欢看文章自学、还是讨论着学？我可以怎么配合你？"

Write: USER.md (learning context), MEMORY.md (learning patterns)

**If scope_flags.life == true:**

`life_context` — Family situation (if willing to share), hobbies, interests, daily routines, things they care about outside work, how the agent can help in life (reminders, planning, research, creative projects, etc.).
Example question: "生活方面有没有你希望我帮忙的？比如提醒、规划、帮你查东西、或者其他？你愿意分享多少都行，不想说的直接跳过。"

Write: USER.md (personal context), MEMORY.md (life patterns)

#### Tier 4 — Cross-Domain Preferences
Coverage fields: `personal_preferences`, `communication_style`, `thinking_habits`, `boundaries_and_sensitivities`

These apply regardless of scope.

`personal_preferences` — What the human most values in an assistant, what annoys them most about AI, general taste.
Example: "你用过其他 AI 助手吗？最受不了什么毛病？最希望我做到什么？"

`communication_style` — Conclusion-first vs background-first, evidence depth, length tolerance, humor tolerance, formality level.
Example: "我给你汇报时，你更喜欢极简还是带论据？长了你会烦吗？"

`thinking_habits` — How the human makes decisions, what frameworks they like, big-picture vs detail-oriented, structured vs intuitive.
Example: "你做决策时偏直觉型还是分析型？喜欢我帮你拆解还是直接给建议？"

`boundaries_and_sensitivities` — Topics to handle carefully, emotional boundaries, things the agent should never joke about or casually bring up.
Example: "有没有什么话题你不希望我主动提起，或者需要我特别注意分寸的？"

Write: USER.md (preferences), MEMORY.md (patterns, sensitivities), IDENTITY.md (default tone)

#### Tier 4.5 — SOUL Generation (between deep-dive and finalization)
This is NOT a coverage field — it is a **synthesis step** that happens once Tier 4 is complete.

After collecting enough context about the human's values, relationship expectations, thinking style, and anti-patterns, the agent MUST:
1. Read `writers/soul_writer.md` for the derivation logic.
2. Generate a SOUL.md draft by **deriving** personality traits and behavioral principles from all collected signals (do NOT copy user's words — synthesize).
3. Present the draft to the human using `prompts/soul_draft.md`.
4. Wait for confirmation before writing to SOUL.md.
5. If the human declines, skip and proceed to Tier 5.

This step is what turns the agent from "a configurable tool" into "a personality that fits this specific human."

Write: SOUL.md (role identity, core personality, behavioral principles, relationship contract)

#### Tier 5 — Style Calibration & Feedback Loop
Coverage fields: `style_calibration`, `feedback_loop_installed`, `cron_job_created`

`style_calibration` — A/B test with real examples, confirm the communication style.
`feedback_loop_installed` — Explain the feedback mechanism, install scoring prompts.
`cron_job_created` — Verify the weekly cron job is running (retry if failed earlier).

Write: AGENTS.md (reporting protocol, calibration cadence), HEARTBEAT.md (review loop)

### Scope-Conditional Logic
Not every human needs every deep-dive. The agent MUST respect scope_flags:

```
IF scope_flags.work == false:
  SKIP work_context, work_environment, work_tasks
  SET these coverage fields = true (not applicable)

IF scope_flags.learning == false:
  SKIP learning_context
  SET coverage = true (not applicable)

IF scope_flags.life == false:
  SKIP life_context
  SET coverage = true (not applicable)
```

This means a human who only wants help with learning will not be asked about work tasks or family life.

### Gap Detection Logic

**During first session (continuous mode):**
```
1. SCAN all workspace .md files → classify every dimension as missing / weak / strong / conflicting / not_applicable
2. CHOOSE profile_mode:
   - cold_start: most core dimensions missing
   - partial_profile: some dimensions strong, some unresolved
   - mature_profile: most dimensions strong, only drift/conflict/weak spots remain
3. DERIVE boolean coverage from the classification:
   - strong / not_applicable => true
   - missing / weak / conflicting => false
4. WRITE the runtime state to `state/understand_you/onboarding-state.json`
5. FOR each tier (1 → 5), in priority order:
   FOR each applicable coverage field in that tier:
     IF field is false AND field is applicable (per scope_flags):
       SELECT matching prompt or craft a natural question
       ENGAGE — ask with warmth, curiosity, and a touch of humor
       WAIT for human reply
       IF human explicitly refuses this topic:
         SET coverage_assessment field = "not_applicable"
         SET coverage field = true (marked as "declined")
         CONTINUE to next field
       EXTRACT signals → APPLY confidence policy
       WRITE confirmed info to appropriate .md files
       UPDATE coverage_assessment field:
         - strong if now sufficient
         - weak if partially clarified but still not enough
         - conflicting if it still clashes with existing rules
       DERIVE coverage field again from the new assessment
       SHOW short receipt (2-3 lines)
       BRIDGE naturally to next gap
6. WHEN all applicable fields are true OR human explicitly stops:
   SHOW full summary of everything learned
   UPDATE `state/understand_you/onboarding-state.json`
```

**During heartbeat (follow-up mode):**
```
1. READ `coverage_assessment` and `coverage` → find the next highest-value unresolved field
   priority order: conflicting > missing > weak
2. IF gap exists AND human is in active conversation:
   ASK one warm micro-question, woven naturally into the current interaction
3. IF no gap → onboarding complete
```

### Completion Condition
Onboarding is complete when ALL applicable coverage fields are `true` (either because they are strong or explicitly not applicable / declined).

### Partial Coverage Behavior
The agent does not wait for full coverage to be useful.
At any point, the agent uses confirmed rules + conservative defaults for uncovered areas.
For existing workspaces, the agent should also preserve already-strong dimensions and avoid rephrasing them unless doing so closes a real conflict.

### Three-Day Convergence Strategy
During approximately the first 3 days after onboarding starts:
1. Prioritize the highest-value missing fields over lower-value polish.
2. Use first session + heartbeat + session-start fallback + post-task seams together as a coordinated learning loop.
3. Prefer asking one more good question now over leaving the agent generic for another day.
4. Aim to reach:
   - Tier 1-3 coverage,
   - most Tier 4 coverage,
   - and a `SOUL.md` draft opportunity
   by the end of this window for an active user.

---

## State Machine
- `detect`
- `gap_audit`
- `warm_welcome`
- `scope_discovery`
- `work_deep_dive`
- `learning_deep_dive`
- `life_deep_dive`
- `environment_tools`
- `conflict_resolution`
- `recalibration_audit`
- `preferences_and_style`
- `feedback_install`
- `observe_adapt`
- `maintenance`

Transition when the current tier's coverage fields are filled.
Skip phases whose scope_flags are false.

---

## Confidence Policy
### High Confidence
Use when:
- explicitly stated by human,
- explicitly confirmed by human,
- repeated and consistent.

May write to long-term files.

### Medium Confidence
Use when:
- strongly implied,
- repeated behavior suggests a stable preference,
- but explicit confirmation is still missing.

Write to daily notes or calibration-state only.

### Low Confidence
Use when:
- lightly inferred,
- single-instance impression.

Do not write to long-term files.

---

## Writeback Rules
1. Never overwrite entire files unless explicitly instructed.
2. Prefer section-aware insertion.
3. Preserve prior meaning and information density.
4. Add rather than replace when possible.
5. Annotate updates when needed.
6. Low-confidence inference stays out of long-term memory.

---

## Mandatory Feedback Loop
After important outputs, optionally ask one short question:
- “0–10 分就行，这次更接近你想要的状态了吗？”
- “如果只改一处，你最想我改哪里？”

Map recurring issues to system files:
- verbosity issue → USER / MEMORY / AGENTS
- weak evidence → AGENTS reporting protocol
- boundary miss → MEMORY default permissions
- wrong tool usage → TOOLS guardrails
- over-interruption → HEARTBEAT reach-out rules

---

## Orchestrator — Runtime Decision Tree

For detailed pseudocode of all execution paths, read `references/orchestrator.md`.

**Execution modes:**

| Mode | Trigger | Behavior |
|------|---------|----------|
| Cold start | New workspace or profile_mode = `cold_start` | Scan .md files → create workspace runtime state → install heartbeat + cron + standing order → **continuous multi-round onboarding conversation** until gaps filled or human stops |
| Partial profile audit | Existing workspace with some strong, some unresolved dimensions | Scan .md files → classify dimensions → preserve strong sections → ask only about `missing` / `weak` / `conflicting` dimensions |
| Mature profile recalibration | Existing workspace where most dimensions are already strong | Audit for drift or conflict only; do not re-onboard from zero |
| Heartbeat follow-up | HEARTBEAT.md entry, every 30 min | ONE micro-question per heartbeat, naturally woven into conversation. Skip if human is not active. |
| Post-task feedback | AGENTS.md standing order, after meaningful tasks | ONE feedback nudge per 3 meaningful outputs. Classify issues, queue rule upgrades. |
| Weekly cron review | Cron job, Saturday 10 AM, isolated session | Review calibration-state, auto-apply low-impact upgrades, queue high-impact proposals, deliver summary. |

**Prompt routing by gap:**

| Coverage gap | Prompt | Tier |
|-------------|--------|------|
| `identity_basics` | `prompts/warm_welcome.md` | 1 |
| `safety_boundaries` | `prompts/boundaries_round_1.md` | 1 |
| `assistance_scope` | `prompts/scope_discovery.md` | 2 |
| `work_context` | `prompts/work_context.md` | 3 (if scope.work) |
| `work_environment` | `prompts/work_environment.md` | 3 (if scope.work) |
| `work_tasks` | `prompts/workflow_round_1.md` | 3 (if scope.work) |
| `learning_context` | `prompts/learning_context.md` | 3 (if scope.learning) |
| `life_context` | `prompts/life_context.md` | 3 (if scope.life) |
| `personal_preferences` | `prompts/boundaries_sensitivities.md` | 4 |
| `communication_style` | `prompts/communication_round_1.md` | 4 |
| `thinking_habits` | `prompts/thinking_habits.md` | 4 |
| `boundaries_and_sensitivities` | `prompts/boundaries_sensitivities.md` | 4 |
| SOUL generation (after Tier 4) | `prompts/soul_draft.md` | 4.5 |
| `style_calibration` | `prompts/style_ab_test.md` | 5 |
| `feedback_loop_installed` | `prompts/feedback_nudge.md` (install mode) | 5 |
| `cron_job_created` | Retry `openclaw cron add` if previously failed | 5 |
| Recalibration (post-onboarding) | `prompts/recalibration_entry.md` | — |

**Critical rule: The first-run session MUST be continuous.** Scan → classify → choose mode → create workspace runtime state → setup → greet → ask → receipt → bridge → ask next → ... until the human stops or all gaps are filled. The agent should aim to cover ALL applicable unresolved dimensions in the first session if the human is engaged.

---

## Success Criteria
The skill is successful when:
- the human feels less friction,
- the agent makes fewer boundary mistakes,
- reports feel more natural and better reasoned,
- the system updates itself after mistakes,
- the human starts feeling "this agent is learning me quickly",
- the agent's personality (SOUL.md) feels like a natural fit, not a generic bot.

---

## SOUL.md Iteration (Post-Onboarding)

SOUL.md is the most stable file in the system. Update only on significant trigger events.
See `writers/soul_writer.md` for full logic.

**Triggers:**
1. **Role shift** — human explicitly redefines the agent's role → update Role Identity + Relationship Contract
2. **Personality-level calibration issue** — recurring issue maps to personality, not operations (e.g., "too passive" = SOUL; "reports too long" = AGENTS) → patch Behavioral Principles
3. **Scope change** — human adds/drops major domain → review Role Identity
4. **Explicit user request** — "adjust your personality" → highest priority, draft + confirm

**Anti-churn rule:** If SOUL.md was updated < 2 weeks ago, prefer adding to Growth Direction rather than modifying Core Personality.

**Every SOUL.md update requires user confirmation before writing.**

---

## Failure Recovery
If the human appears tired, resistant, or busy:
- stop probing,
- summarize what was learned,
- switch to observation mode,
- try again later in a smaller, more concrete way.

If style drift persists:
- use A/B mini-calibration,
- review prior high-scoring outputs,
- upgrade a single rule rather than many at once.

If personality drift persists (agent feels "off" despite operational rules being correct):
- re-read SOUL.md,
- check if Role Identity or Behavioral Principles have become stale,
- draft a SOUL.md patch and present to the human.

---

## Final Rule
This skill exists to reduce future friction.
If its behavior creates more friction than it removes, it is failing.

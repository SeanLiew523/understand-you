# Orchestrator — Detailed Pseudocode

This file contains the full decision tree for all Understand You invocation paths.

---

## Runtime Paths

Use these workspace runtime files during execution:

- `state/understand_you/onboarding-state.json`
- `state/understand_you/calibration-state.json`

Use the packaged `understand-you/state/*.json` files only as templates.
Do not treat the packaged copies as the live runtime state once the skill is installed.

---

## Bootstrap: Stable First Trigger

This skill has one canonical bootstrap command:

`/skill understand_you start`

Treat that command as an explicit request to install or repair the onboarding loop.
It is the stable first-run entrypoint after the user installs the skill and starts a new session.

```
ENTRY:
  IF the skill is invoked with input containing "start", "bootstrap", "init", or "install":
    TREAT it as bootstrap intent
    LOAD workspace `state/understand_you/onboarding-state.json` if present
    IF missing:
      LOAD the packaged template from {baseDir}/state/onboarding-state.json

    IF runtime onboarding state is missing:
      RUN the full first-run path below

    IF onboarding.status IN ["active", "paused", "not_started"]:
      REPAIR any missing infrastructure:
        - HEARTBEAT.md onboarding section
        - AGENTS.md fallback standing order
        - understand_you_weekly_review cron job
      THEN continue the onboarding conversation in the same session

    IF onboarding.status IN ["completed", "completed_partial"]:
      VERIFY infrastructure still exists
      REPAIR missing infrastructure if needed
      TELL the human onboarding is already installed
      OFFER a short recalibration pass if they want to refresh the profile
```

---

## First-Run: Full Onboarding Session

This runs the FIRST time the skill is invoked (by user or trigger).
It is a **continuous conversation**, not a single question.

```
STEP 1 — SCAN EXISTING CONTEXT:
  READ USER.md, MEMORY.md, AGENTS.md, TOOLS.md, IDENTITY.md, HEARTBEAT.md
  FOR each file:
    EXTRACT signals relevant to coverage fields
    CLASSIFY each dimension as:
      - missing
      - weak
      - strong
      - conflicting
      - not_applicable
    LOG the strongest evidence source for each dimension
  
  Example mappings:
    USER.md contains preferred name       → identity_basics = strong
    MEMORY.md contains permission rules   → safety_boundaries = strong
    TOOLS.md contains OS/device info      → work_environment = strong
    USER.md says "work only"              → life_context = not_applicable
    USER.md and MEMORY.md disagree        → corresponding field = conflicting

  CHOOSE profile_mode:
    IF most core dimensions are missing:
      profile_mode = cold_start
    ELSE IF most core dimensions are already strong:
      profile_mode = mature_profile
    ELSE:
      profile_mode = partial_profile

  DERIVE boolean coverage:
    strong / not_applicable => true
    missing / weak / conflicting => false

STEP 2 — INSTALL PROACTIVE INFRASTRUCTURE:
  a) WRITE runtime state first:
     CREATE or UPDATE:
       - state/understand_you/onboarding-state.json
       - state/understand_you/calibration-state.json
     SOURCE defaults from packaged templates only if runtime files do not exist

  b) WRITE to HEARTBEAT.md:
     ADD or REPLACE the "## Understand You — Onboarding Check" section
     IF an older "Trust Accelerator" or stale onboarding block exists:
       REPLACE that block in place instead of appending a duplicate
     (see SKILL.md "Step 2a" for exact content)
  
  c) WRITE to AGENTS.md:
     ADD or REPLACE the "## Understand You" standing order section
     IF an older "Trust Accelerator" or stale onboarding fallback exists:
       REPLACE that block in place instead of appending a duplicate

  d) TRY to CREATE cron job:
     RUN a dedicated cron creation action:
       openclaw cron add \
       --name "understand_you_weekly_review" \
       --cron "0 10 * * 6" \
       --session isolated \
       --message "Read the understand_you skill. Review calibration-state..."

     CRITICAL:
       - do NOT combine runtime-state initialization and cron creation in one shell exec chain
       - cron creation happens only after runtime state + AGENTS/HEARTBEAT writes exist
     
     IF cron creation FAILS (e.g., 1000 normal closure, timeout, gateway error):
       SET coverage_assessment.cron_job_created = weak
       SET coverage.cron_job_created = false
       LOG the error in state/understand_you/onboarding-state.json notes[]
       ADD to HEARTBEAT.md: "## Cron Retry — understand_you_weekly_review cron job 
         failed to create. On each heartbeat, retry openclaw cron add. 
         Once successful, remove this retry section and set cron_job_created = true."
     IF cron creation SUCCEEDS:
       SET coverage_assessment.cron_job_created = strong
       SET coverage.cron_job_created = true
 
  e) UPDATE runtime onboarding state:
     SET status = "active", profile_mode, coverage_assessment, derived coverage, started_at = now

STEP 3 — CONTINUOUS ONBOARDING CONVERSATION:
  SET rounds_this_session = 0
  
  GREET using prompts/warm_welcome.md:
    ADAPT based on scan results and profile_mode
    IF scan found the human's name: use it immediately
    IF profile_mode == mature_profile:
      start with a narrow recalibration opener, not a full welcome
  
  LOOP:
    WAIT for human reply
    
    EXTRACT signals from reply
    APPLY {baseDir}/policies/confidence-policy.md
    GENERATE writeback patches
    APPLY {baseDir}/policies/merge-policy.md
    ROUTE to writers → EXECUTE writes
    UPDATE coverage_assessment first
    DERIVE coverage fields again
    SHOW short receipt (2-3 lines max)
    
    INCREMENT rounds_this_session
    
    FIND next_gap using TIER PRIORITY + SCOPE FLAGS:
      conflicting > missing > weak
    
      Tier 1 (must complete first):
        identity_basics          → prompts/warm_welcome.md
        safety_boundaries        → prompts/boundaries_round_1.md
      
      Tier 2 (determines deep-dive scope):
        assistance_scope         → prompts/scope_discovery.md
      
      Tier 3 (conditional on scope_flags):
        IF scope_flags.work:
          work_context           → prompts/work_context.md
          work_environment       → prompts/work_environment.md
          work_tasks             → prompts/workflow_round_1.md
        IF scope_flags.learning:
          learning_context       → prompts/learning_context.md
        IF scope_flags.life:
          life_context           → prompts/life_context.md
      
      Tier 4 (cross-domain, always active):
        personal_preferences     → prompts/boundaries_sensitivities.md (first half)
        communication_style      → prompts/communication_round_1.md
        thinking_habits          → prompts/thinking_habits.md
        boundaries_and_sensitivities → prompts/boundaries_sensitivities.md
      
      Tier 4.5 (SOUL generation — after Tier 4 is complete):
        IF SOUL.md does not exist or is generic:
          → prompts/soul_draft.md (generate, present, confirm)
      
      Tier 5 (finalization):
        style_calibration        → prompts/style_ab_test.md
        feedback_loop_installed  → prompts/feedback_nudge.md (install mode)
        cron_job_created         → retry cron creation if previously failed
    
    IF next_gap is null:
      ALL COVERAGE COMPLETE
      SET onboarding.status = "completed"
      SHOW full summary receipt of EVERYTHING learned
      BREAK
    
    CHECK fatigue:
      IF human explicitly says stop ("够了", "先这样", "later", "不聊了"):
        SET signals.explicit_stop = true
        SAY "没问题，我先按目前了解的来。剩下的我后面慢慢跟你确认。"
        SHOW summary of known + remaining dimensions
        BREAK
      
      IF human reply was very short AND previous reply was also very short:
        IF the second reply adds little new signal:
          SAY "我先按目前了解的来，剩下的后面找合适时机慢慢补。"
          SHOW summary of known + remaining
          BREAK
      
      IF rounds_this_session >= 5:
        SAY "我先消化一下这些信息，还有几个方面后面我找机会再聊。"
        SHOW summary of known + remaining
        BREAK (but do NOT mark explicit_stop — heartbeat will continue)
    
    BRIDGE naturally to next topic:
      LOAD the prompt matching next_gap
      USE that prompt's bridging phrases
      ASK the next question
  
  END LOOP
  
  SAVE state/understand_you/onboarding-state.json
```

---

## Heartbeat: Follow-Up Gap Filling

Triggered by HEARTBEAT.md entry. Runs every heartbeat cycle (default 30 min).

```
LOAD workspace `state/understand_you/onboarding-state.json` AS onboarding

IF onboarding.status NOT IN ["active", "paused"]:
  CHECK if cron_job_created == false:
    TRY to create cron job again
    IF success: SET cron_job_created = true, SAVE state
  SKIP further onboarding actions

FIND next_gap = highest-priority unresolved dimension by tier priority (respecting scope_flags)
  where conflicting > missing > weak

IF no gap:
  SET onboarding.status = "completed"
  SKIP

IF signals.explicit_stop == true:
  DO NOT ask proactive onboarding questions
  ONLY fill gaps opportunistically when the human's current conversation 
    naturally touches a relevant topic
  RETURN

CHECK context:
  IF human is currently in active conversation:
    ASK one micro-question about next_gap
    WEAVE it naturally into the current topic if possible
    PROCESS reply → WRITE → UPDATE coverage
  ELSE:
    SKIP (do not send unsolicited onboarding messages when human is idle)

IF onboarding.day_index <= 3:
  TREAT onboarding as an active learning sprint:
    - prefer the highest-value remaining gap first
    - use session start, task handoff, and task close as natural seams
    - after two low-energy replies, skip only the next heartbeat attempt unless explicit_stop == true
```

---

## Post-Task: Feedback Nudge

Triggered by AGENTS.md standing order after meaningful task completion.

```
IF onboarding.status IN ["completed", "completed_partial"]:
  IF task was meaningful (not trivial):
    CHECK feedback budget (no more than 1 nudge per 3 meaningful outputs)
    IF eligible:
      LOAD {baseDir}/prompts/feedback_nudge.md
      ASK one question
      IF score <= 6:
        CLASSIFY issue_type
        RECORD to calibration.recent_feedback
        INCREMENT calibration.recurring_issues if pattern matches
        IF recurring_issues count >= 3 for any type:
          QUEUE rule_upgrade
      UPDATE workspace `state/understand_you/calibration-state.json`
```

---

## Cron: Weekly Calibration Review (Post-Onboarding Only)

Triggered by the cron job created during first-run Step 2b.
Runs in an isolated session, Saturday 10 AM.

```
LOAD workspace `state/understand_you/onboarding-state.json` AS onboarding
LOAD workspace `state/understand_you/calibration-state.json` AS calibration

IF onboarding.status NOT IN ["completed", "completed_partial"]:
  SKIP (onboarding still active, heartbeat handles this)

REVIEW calibration.recurring_issues:
  FOR each issue with count >= 3:
    IDENTIFY target file and section
    CLASSIFY impact:
      HIGH impact if the change touches:
        - SOUL.md
        - MEMORY.md sections such as Absolute Principles or Default Permissions
        - account ownership / external action rules
        - privacy or destructive-action handling
        - standing-order authority
      LOW impact if the change only refines:
        - report length / structure
        - evidence depth
        - wording / tone intensity
        - feedback cadence
    
    IF impact == "high":
      QUEUE proposal with:
        status = "proposed"
        impact_level = "high"
        requires_confirmation = true
    ELSE:
      QUEUE proposal with:
        status = "queued"
        impact_level = "low"
        requires_confirmation = false
    
REVIEW calibration.rule_upgrade_queue:
  FOR each item with status == "queued" AND impact_level == "low":
    GENERATE writeback patch
    APPLY merge policy
    EXECUTE write
    SET status = "applied"

  FOR each item with status == "proposed" AND impact_level == "high":
    KEEP as proposal
    DO NOT write yet

  FOR each item with status == "approved" AND impact_level == "high":
    GENERATE writeback patch
    APPLY merge policy
    EXECUTE write
    SET status = "applied"

DELIVER summary to human:
  "本周校准回顾：已自动修正 M 条低影响规则；另有 N 条高影响提案待你确认。详细如下：..."

REVIEW SOUL.md for personality drift:
  FOR each recurring_issue:
    IF issue maps to personality layer (passivity, bluffing, wrong tone):
      DRAFT SOUL.md patch
      PRESENT to human for confirmation in the delivery summary
    IF issue maps to operational layer (length, format, evidence):
      SKIP SOUL, handle via AGENTS.md rule upgrade

REVIEW MEMORY.md:
  PRUNE stale active bets if contradicted by recent context
  DISTILL any confirmed patterns from recent interactions

LATER, when the human confirms a high-impact proposal in a normal session:
  UPDATE that queue item:
    status = "approved"
  THEN allow the next calibration pass to apply it safely
```

---

## Post-Install First-Run Detection

When the skill is loaded but state files don't exist yet:

```
IF workspace `state/understand_you/onboarding-state.json` does not exist:
  CREATE it from {baseDir}/state/onboarding-state.json template
  WRITE the new runtime copy to `state/understand_you/onboarding-state.json`

IF workspace `state/understand_you/calibration-state.json` does not exist:
  CREATE it from {baseDir}/state/calibration-state.json template
  WRITE the new runtime copy to `state/understand_you/calibration-state.json`

IF onboarding.status == "not_started":
  EXECUTE "First-Run: Full Onboarding Session" (see above)
```

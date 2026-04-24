# understand_you

`understand_you` is an OpenClaw skill for one job: help a newly installed or partially profiled agent become noticeably more aligned with its owner in about 3 days of real use.

It is designed for two kinds of workspaces:

- brand-new workspaces that need full onboarding
- existing workspaces that already have some memory/config, but still have profile gaps, weak rules, or drifted calibration

## What Makes It Different

Most onboarding skills ask a few questions and stop. This one is built to keep converging.

Its core behavior is:

- first session: actively ask several high-value questions instead of staying generic
- after first session: keep filling missing context through heartbeat and natural follow-ups
- during normal use: absorb feedback and tighten rules without forcing the user to repeat themselves
- over time: update `SOUL.md` when the relationship changes from "study partner" to "operator", "work partner", "researcher", and so on

The target user experience is:

- the agent quickly learns your business context, work habits, tools, task priorities, and communication preferences
- it understands your boundaries and permission rules
- it writes the understanding into the right OpenClaw files
- it stops sounding generic and starts feeling like it actually knows how to work with you

## Key Highlights

1. Fast convergence

The skill is optimized for a visible change in agent behavior within about 3 days, not for a one-off questionnaire.

2. Works for both new and old workspaces

It supports:

- `cold_start`
- `partial_profile`
- `mature_profile`

So it does not blindly re-onboard an existing user from zero.

3. Gap-audit instead of repetition

Before asking questions, it scans the current workspace and classifies each dimension as:

- `missing`
- `weak`
- `strong`
- `conflicting`
- `not_applicable`

Then it only asks about the weak or missing parts.

4. Writes to the files that actually matter

Depending on confirmed information and approval level, it can update:

- `USER.md`
- `MEMORY.md`
- `TOOLS.md`
- `AGENTS.md`
- `HEARTBEAT.md`
- `SOUL.md`

5. High-impact changes still need confirmation

This skill is proactive, but it does not silently change high-impact rules such as permissions, identity posture, or major `SOUL.md` shifts without explicit confirmation.

## Installation

Copy this folder into your OpenClaw workspace `skills/` directory.

Example:

```bash
cp -R understand-you /path/to/your/openclaw-workspace/skills/understand-you
```

## Important: Install Does Not Auto-Run

OpenClaw skills are passive by default. Installing the folder makes the skill available, but it will not reliably start onboarding by itself.

For a clean first run:

1. start a new session with `/new`
2. run:

```text
/skill understand_you start
```

That is the canonical bootstrap command for this skill.

## Recommended Test Flow

For the cleanest test:

1. prepare a fresh workspace
2. copy this skill into `skills/understand-you`
3. start a new session
4. run `/skill understand_you start`
5. answer naturally for several rounds
6. inspect the generated workspace files

## What Happens On First Run

The skill is designed to:

1. scan the current workspace files first
2. decide whether this is `cold_start`, `partial_profile`, or `mature_profile`
3. create runtime state under:
   - `state/understand_you/onboarding-state.json`
   - `state/understand_you/calibration-state.json`
4. install proactive follow-up infrastructure in `AGENTS.md` and `HEARTBEAT.md`
5. begin the onboarding conversation in the same session
6. write confirmed information into the appropriate `.md` files

## OpenClaw Scope

This package is designed for OpenClaw workspace semantics. It assumes the usual OpenClaw bootstrap files and OpenClaw-style heartbeat/cron behavior.

It is not marketed as a generic multi-runtime skill package.

## Repository Layout

```text
understand-you/
  SKILL.md
  README.md
  policies/
  prompts/
  references/
  schemas/
  state/
  writers/
```

## Usage Notes

- If you are testing on an old workspace, remove stale onboarding sections from earlier skills before judging results.
- If the gateway or chat channel is unstable, fix that first. A broken Discord/WebSocket session can interrupt onboarding and leave partial state.
- The packaged `state/*.json` files are templates. Runtime state should live in the workspace under `state/understand_you/`.

## License

No license file is included yet. Add one before wider redistribution if you want clear reuse terms.

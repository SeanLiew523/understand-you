> Chinese reader? Start here: [打开中文 README](README.zh-CN.md)

# understand_you

Language: **English** | [中文](README.zh-CN.md)

`understand_you` is an OpenClaw skill built for one job: help a newly installed or partially profiled agent become noticeably more aligned with its owner in about 3 days of real use.

## Why This Exists

Most onboarding skills ask a few questions and stop. That leaves the agent generic.

`understand_you` is designed to keep converging:

- the first session actively learns the highest-value context
- the next 1-3 days keep filling profile gaps through heartbeat and natural follow-ups
- confirmed information is written into the right OpenClaw files
- calibration feedback turns into better rules instead of being forgotten
- `SOUL.md` can evolve when the agent's role changes over time

The intended effect is simple: the agent should stop sounding generic and start feeling like it actually knows how to work with you.

## Key Highlights

1. **Fast convergence**

   The skill is optimized for a visible behavior change within about 3 days, not for a one-off questionnaire.

2. **Works for both new and existing workspaces**

   It supports three profile modes:

   - `cold_start`
   - `partial_profile`
   - `mature_profile`

   So it does not blindly re-onboard an existing user from zero.

3. **Gap-audit before asking**

   Before it asks anything, it scans the current workspace and classifies each dimension as:

   - `missing`
   - `weak`
   - `strong`
   - `conflicting`
   - `not_applicable`

   Then it only asks about weak, missing, or conflicting dimensions.

4. **Writes to the files that matter**

   Depending on confirmed information and approval level, it can update:

   - `USER.md`
   - `MEMORY.md`
   - `TOOLS.md`
   - `AGENTS.md`
   - `HEARTBEAT.md`
   - `SOUL.md`

5. **High-impact changes still require confirmation**

   The skill is proactive, but it does not silently change permissions, identity posture, or major `SOUL.md` shifts without explicit confirmation.

## Installation

Copy this folder into your OpenClaw workspace `skills/` directory.

Example:

```bash
cp -R understand-you /path/to/your/openclaw-workspace/skills/understand-you
```

## How To Start It

Installing the folder makes the skill available, but it will not reliably auto-run by itself.

For a clean first run:

1. start a new session with `/new`
2. run:

```text
/skill understand_you start
```

That is the canonical bootstrap command for this skill.

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

## Recommended Test Flow

1. prepare a fresh workspace
2. copy this skill into `skills/understand-you`
3. start a new session
4. run `/skill understand_you start`
5. answer naturally for several rounds
6. inspect the generated workspace files

## OpenClaw Scope

This package is designed for OpenClaw workspace semantics. It assumes the usual OpenClaw bootstrap files and OpenClaw-style heartbeat and cron behavior.

It is not presented as a generic multi-runtime skill package.

## Repository Layout

```text
understand-you/
  SKILL.md
  README.md
  README.zh-CN.md
  policies/
  prompts/
  references/
  schemas/
  state/
  writers/
```

## Usage Notes

- If you are testing on an old workspace, remove stale onboarding sections from earlier skills before judging results.
- If the gateway or chat channel is unstable, fix that first. A broken Discord or WebSocket session can interrupt onboarding and leave partial state.
- The packaged `state/*.json` files are templates. Runtime state should live in the workspace under `state/understand_you/`.

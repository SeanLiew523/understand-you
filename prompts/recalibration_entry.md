# Recalibration Entry

## Goal
Re-align the agent after onboarding is complete, triggered by repeated mismatches or explicit user request.
This is NOT the warm welcome flow — the relationship already exists.

## When to Use
- repeated style mismatch (recurring_issues count >= 3 for style-related types)
- repeated boundary mismatch (recurring_issues count >= 3 for boundary-related types)
- explicit user request to improve alignment
- agent detects significant drift between current behavior and stored rules

## Style
- direct, not ceremonial
- acknowledge the specific issue, not vague "let me recalibrate"
- short — one focused correction per round
- respectful of the existing relationship

## Template — Issue-Specific Recalibration

### For style drift
我最近注意到有几次输出可能偏离了你的期望，想快速校准一下。

我目前的默认是 [current rule summary]。
你觉得这个方向还对吗？还是有哪里需要我调整？

你只要告诉我最想改的一点就行。

### For boundary miss
我最近有 [N] 次可能踩到了边界，我想把这个规则再确认一遍。

我目前的理解是：[current boundary rule]
这个理解还准吗？还是需要修正？

### For general drift
最近几次合作，我感觉有些地方可能偏了。我不确定具体是哪里，所以直接问你：

如果只改一处让我更好用，你最想改哪里？

### For explicit user request
收到，我来做一次快速校准。

我先把我目前运行的几条核心规则列给你看，你告诉我哪条需要改：
1. [boundary summary]
2. [output preference summary]
3. [proactive behavior summary]

你只要指出不对的编号就行。

## After Human Reply
1. Reflect back the correction clearly
2. Apply confidence policy (explicit statement → High)
3. Generate writeback patch targeting the specific rule
4. Apply merge policy (prefer annotated_update or replace_section)
5. Show receipt
6. Reset the relevant recurring_issue count in calibration-state

## Rules
- Do NOT re-run the full onboarding flow
- Do NOT ask unrelated questions during recalibration
- Fix one thing well, then return to normal operation
- If the human says "没什么问题", accept it and stop

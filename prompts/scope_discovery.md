# Scope Discovery

## Goal
Understand which life domains the human wants the agent to help with.
This determines which deep-dive dimensions to activate next.

## Template
边界和基本规则我记住了。接下来我想搞清楚一件影响后面所有事的事：

你希望我在哪些方面帮你？
- **工作**（项目、任务、资讯、协作）
- **学习**（研究、阅读、知识整理）
- **生活**（日程、提醒、兴趣、家庭相关）
- 还是 **全都有**？

你告诉我范围，我后面问的问题才不会跑偏。

## After Reply (CRITICAL — DO NOT STOP HERE)
1. Set `scope_flags` based on the answer.
2. Write to USER.md (assistance scope).
3. Update coverage: `assistance_scope = true`.
4. **Immediately bridge to the first active deep-dive tier.**
   - If work: "好，那我先了解一下你的工作背景——"
   - If learning: "那你现在在学什么？怎么学的？"
   - If life: "生活方面，你最希望我帮上忙的是哪块？"
   - If all: "那我们一个一个来，先从工作开始——"
5. Continue the conversation — do NOT end here.

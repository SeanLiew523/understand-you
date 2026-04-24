> English reader? Start here: [Open English README](README.md)

# understand_you

语言：**中文** | [English](README.md)

`understand_you` 是一个给 OpenClaw 用的技能，核心目标只有一个：让一个刚装好的，或者还不够懂主人的 agent，在大约 3 天的真实使用里明显变得更懂你。

## 它为什么存在

很多 onboarding skill 只是问几句，然后就停了。结果就是 agent 还是很泛，很像“知道一点点你”，但不会真的按你的方式工作。

`understand_you` 的设计目标不是“问完一轮”，而是“持续收敛”：

- 第一次会优先主动了解最高价值的信息
- 接下来 1 到 3 天会通过 heartbeat 和自然对话继续补齐缺口
- 确认过的信息会写进正确的 OpenClaw 配置文件
- 后续使用中的反馈会变成规则修正，而不是被遗忘
- 当 agent 和你的关系发生变化时，`SOUL.md` 也可以跟着调整

最终想实现的体感很直接：让它尽快摆脱“泛泛而谈”，开始像一个真的懂你、会按你的方式配合你的 OpenClaw。

## 最大亮点

1. **快速收敛**

   这个 skill 的目标不是做一次性问卷，而是让 agent 在大约 3 天内出现明显变化。

2. **既适合新工作区，也适合老工作区**

   它支持三种模式：

   - `cold_start`
   - `partial_profile`
   - `mature_profile`

   所以它不会把一个已经用过一段时间的用户，当作全新用户重新从零问起。

3. **先做 gap audit，再决定问什么**

   在真正提问之前，它会先扫描当前 workspace，并把每个维度判断为：

   - `missing`
   - `weak`
   - `strong`
   - `conflicting`
   - `not_applicable`

   然后只针对缺失、偏弱、冲突的部分继续补。

4. **会把信息写到真正关键的文件里**

   根据你已经确认的信息和影响级别，它可以更新：

   - `USER.md`
   - `MEMORY.md`
   - `TOOLS.md`
   - `AGENTS.md`
   - `HEARTBEAT.md`
   - `SOUL.md`

5. **高影响修改仍然要求确认**

   这个 skill 会主动，但不会擅自改高影响内容。像权限边界、身份姿态、较大的 `SOUL.md` 调整，仍然需要你明确确认。

## 安装方式

把这个文件夹复制到 OpenClaw 工作区的 `skills/` 目录下。

例如：

```bash
cp -R understand-you /path/to/your/openclaw-workspace/skills/understand-you
```

## 如何启动

把 skill 文件夹放进去之后，它会变成“可用”，但不会稳定地自己自动开始跑。

正确的首次启动方式是：

1. 先开一个新会话：`/new`
2. 再执行：

```text
/skill understand_you start
```

这条命令就是这个 skill 的标准 bootstrap 入口。

## 首次运行会做什么

这个 skill 的设计流程是：

1. 先扫描当前 workspace 里的已有文件
2. 判断当前属于 `cold_start`、`partial_profile` 还是 `mature_profile`
3. 在下面两个路径创建 runtime state：
   - `state/understand_you/onboarding-state.json`
   - `state/understand_you/calibration-state.json`
4. 在 `AGENTS.md` 和 `HEARTBEAT.md` 里安装后续主动补盲的基础设施
5. 在同一轮会话里开始 onboarding 对话
6. 把确认过的信息写入对应的 `.md` 文件

## 推荐测试方式

1. 准备一个干净的 workspace
2. 把这个 skill 复制到 `skills/understand-you`
3. 开新会话
4. 执行 `/skill understand_you start`
5. 自然回答几轮问题
6. 检查生成出来的 workspace 文件

## OpenClaw 适用范围

这个包是按 OpenClaw 的 workspace 语义设计的，默认假设存在 OpenClaw 的 bootstrap 文件、heartbeat 机制和 cron 行为。

它不是一个主打多 runtime 通用的 skill 包。

## 仓库结构

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

## 使用说明

- 如果你是在旧 workspace 里测试，先清理以前 skill 留下的旧 onboarding section，再判断效果。
- 如果 gateway 或聊天通道本身不稳定，先把这个问题处理掉。Discord 或 WebSocket 断线会打断 onboarding，留下半成品状态。
- 包里的 `state/*.json` 只是模板。真正运行时状态应该写到 workspace 下的 `state/understand_you/`。

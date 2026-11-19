# Question Forge

把粗糙、模糊、信息缺失的问题，锻造成**决策级问题**或**可执行任务书**。

它不是替你回答问题，而是先把事实查清，再把真正的决定一条条摆到你面前：每个问题带选项、推荐答案、默认值和猜错代价。问完最后一轮，你会得到一个能直接拿去讨论、写规格或交给 agent 执行的问题。

> 一句话版：**你出模糊想法，我把它问成一张能拍板的决策清单。**

## 它解决什么

原始请求往往长这样：

- “帮我做个 AI 周报工具，越快越好。”
- “我想做一个个人知识库，你看着办。”
- “帮我写个爬虫，抓点数据。”
- “这个方案怎么样？”

这些句子里缺的不是答案，而是**问题本身**：为谁做、解决哪一步、边界在哪、什么算成功、谁拍板、哪些不能动。

Question Forge 用一套固定流程把这些补回来：

1. **先查事实**——文件、代码、文档、网络能回答的，不问用户；
2. **建决策树**——把问题拆成目标、现状、范围、约束、验收、风险六类；
3. **逐轮提问**——每轮只问“前沿”，最多 5 个，每个带推荐答案和默认代价；
4. **收敛**——输出锻造后的问题、决策台账、未知项，确认共享理解后才行动。

## 三种模式

| 模式 | 什么时候用 | 产出 |
|---|---|---|
| **Forge** | 有对话、可以多轮 | 锻造后的问题 + 决策台账 + 未知项 |
| **One-shot** | “别问了，直接改”/ 问题要发给不会回复的人 | 改写后的问题 + 假设清单 + 待确认项 |
| **Brief** | 要把问题交给执行 agent | ≤4000 字符的任务书 |

## 安装

### DeepSeek Harness

DSH 会自动扫描 `~/.agents/skills` 和 `~/.dsh/skills` 下的一层目录，读取 `SKILL.md` frontmatter 生成目录，不需要重启。

```bash
git clone https://github.com/xxuucc/question-forge.git ~/.agents/skills/question-forge
```

如果只想给 DSH 用：

```bash
git clone https://github.com/xxuucc/question-forge.git ~/.dsh/skills/question-forge
```

### Claude Code

```bash
git clone https://github.com/xxuucc/question-forge.git ~/.claude/skills/question-forge
```

### Codex / 其他 Agent Skills 客户端

```bash
git clone https://github.com/xxuucc/question-forge.git ~/.agents/skills/question-forge
```

仓库根目录就是 skill 目录，`SKILL.md` 在根上。

## 触发方式

模型会在这些场景自动加载：

- “帮我把问题问清楚”
- “我只有一个模糊想法”
- “需求不清晰 / 信息不全”
- “grill me / 拷问我 / 挑战一下这个方案”
- “写个 brief / 任务书给 agent”
- “这个 prompt 怎么改”

也可以手动：

```text
/question-forge
```

## 一个最小例子

原始问题：

> 帮我做个 AI 周报工具，越快越好。

第一轮可能只问四个：

```text
❓ Q1 — 为谁做：第一版的直接用户是谁？
A. 你自己  B. 3–8 人团队  C. 20+ 人部门
➡️ 推荐：B。默认：B；猜错代价：范围和数据源全变。

❓ Q2 — 最痛的动作：现在写周报最耗时间的是哪一步？
A. 收集素材  B. 判断重点  C. 组织成文  D. 反复修改
➡️ 推荐：A。默认：A；猜错代价：做出来的功能不是第一痛点。

❓ Q3 — 数据源：第一版允许接哪些数据？
A. 只接 Git  B. Git + 任务系统  C. 再加聊天记录  D. 先手动粘贴
➡️ 推荐：A。默认：A；猜错代价：接太多延期，接太少价值不足。

❓ Q4 — 成功标准：什么证据算“成了”？
A. 自己用 2 周  B. 3 人用 2 周、时间减半  C. 领导说以后就这么交
➡️ 推荐：B。默认：B；猜错代价：验收太松，做出来没人用。
```

答完两三轮，你会拿到一份这样的东西：

```markdown
**一句话**：为 3–8 人团队做一个基于 Git 提交的 AI 周报工具，第一版只解决“收集本周做了什么”，人工确认后输出 Markdown，用 2 周内周报时间减半来验收。

**范围**：包含 Git 采集、AI 归纳、Markdown 草稿、人工确认；不包含聊天记录、Jira、自动发送。

**关键假设**：假设团队都用同一个 Git 仓库（未确认）。

**待确认**：仓库是否统一；提交信息质量是否够；谁来做最终验收。
```

## 设计来源

这个 skill 是两个开源 skill 的**综合再创作**，不是二选一，也不是逐字拼接：

- **[leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader)**（数字生命卡兹克，MIT）：事实先查、一轮 ≤5 个问题、选项 + 推荐、没问的决定记账、执行任务书的分节结构与防作弊。
- **[grilling / grill-me](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling)**（Matt Pocock，MIT）：决策树、前沿、轮次、事实与决定分离、共享理解确认门。

Question Forge 在两者之上加了三样东西：

1. **信息缺口八项**：把“信息丢失”变成可检查的清单；
2. **≤5 与前沿的合并规则**：前沿全问，但超过 5 个按“影响 × 不可逆性”分批，依赖问题不拆轮；
3. **质量门与反模式**：防止“只改措辞不补信息”和“审问式提问”。

详细来源与 MIT 许可见 [ATTRIBUTION.md](ATTRIBUTION.md)。

## 仓库结构

```text
question-forge/
  SKILL.md                       # 主技能
  references/
    interview-playbook.md        # 决策树、前沿、问题模式
    brief-template.md            # 给执行 agent 的任务书模板
    examples.md                  # Forge / One-shot / Brief 三个例子
  README.md
  ATTRIBUTION.md
  LICENSE
```

## 许可

本仓库原创文本以 MIT 发布。上游 leader 与 grilling 同为 MIT，完整许可见 [ATTRIBUTION.md](ATTRIBUTION.md)。

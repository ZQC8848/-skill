# 架构

**最后核实时间：2026-08-21**（设计阶段——还没有代码存在；这里描述的是达成一致的形态，不是一个在运行的系统）

## 范围

一个以 Claude Skill 形式实现的 MVP（prompt + reference 文件 + 持久化的 markdown 状态，没有自定义的编排代码），协助用户处理**dating app 上同时进行的多个 match**——不是线下追求，也不是长期/婚姻导向的关系维护。为什么现在是 Skill 而不是写代码的多 Agent 系统，见 [decisions/mvp-skill-form-not-multiagent.md](decisions/mvp-skill-form-not-multiagent.md)。

## 这套设计围绕的唯一硬问题

用户会同时经营好几个 match。其他一切（信号识别、阶段模型、人格包装）都是内容层面的事；**match 之间的隔离是塑造整个系统的结构性约束**。见 [decisions/isolation-conversation-per-match.md](decisions/isolation-conversation-per-match.md)。

## 三个入口，三种不同的读写范围

```
┌─────────────────────┐   ┌──────────────────────┐   ┌────────────────────┐
│   单对象对话           │   │   Self-model 汇总      │   │   Dashboard /       │
│  (per-match)          │   │  (aggregation)         │   │   全局概览           │
│                      │   │                        │   │                    │
│  读写：                │   │  读：self/inbox.md     │   │  只读：             │
│  profiles/<id>.md     │   │  写：self/policy.md     │   │  profiles/_index.md│
│  追加写入：             │   │                        │   │  ——绝不打开任何      │
│  self/inbox.md        │   │                        │   │  单个对象的档案文件    │
└─────────────────────┘   └──────────────────────┘   └────────────────────┘
```

除了 `self/inbox.md`（已脱敏，不带可识别细节）和 `profiles/_index.md`（只有状态字段，不含内容）之外，没有任何东西能跨越这三块的边界。这正是让"看全局"（dashboard）和"跨 match 学习"（self-model）不会变成绕开单对象隔离的后门的关键。

### 1. 单对象对话（per-match conversation）

一个 match 一个对话/线程，对应 dating app 自己的收件箱本来就是"一个 match 一个线程"——隔离机制和最终产品的 UX 是同一套结构，不是两套设计硬拼在一起。

- 开始时：先确认这是哪个 match（见下面的归属网关），重新 `Read` 该对象的 `profiles/<id>.md`——不依赖对话记忆作为真相来源，哪怕是跟同一个人聊了很久的线程也一样（防的是**时间上的**漂移，不只是跨人污染）。
- 每条回复开头带状态标签（`【hinge-sarah-0818 · 阶段：破冰 · 好感度 6/10 ↑】`）——既是 UX 模式（借鉴自 HowToGetAlongWithGirls），也是强制自检：标签错了，用户能立刻发现。
- 出现值得沉淀的结果时，往 `self/inbox.md` 追加一条脱敏后的记录（不带该 match 的可识别细节）。

### 2. 归属网关（在单对象对话开始时、或输入信息有歧义时触发）

硬规则，不是"不清楚就问"这种软规则：

- **只有**满足以下条件才能自动归属到某个已有档案：截图/文字里出现了 app 内昵称，且**唯一匹配**某一个档案；或者用户明确点名。
- 其他任何情况——包括存在两个可能的候选——都要停下来问，绝不能挑"看起来更像"的那个。

### 3. Self-model 汇总

跟任何单对象对话分开运行（目前：在 Claude Code 里用一次性的 Agent 调用；以后是定时的后台任务）。只读 `self/inbox.md`，写 `self/policy.md`（对**这个用户**普遍有效的打法——不绑定任何具体 match）。

### 4. Dashboard / 全局概览

回答"谁在降温、该放弃谁"这类问题——**只**读 `profiles/_index.md`（阶段 / 好感度 / 趋势 / 最后更新时间 / 生命周期状态），从不打开任何单个 match 的完整档案或聊天记录。

## 文件结构

```
profiles/
├── _index.md              # 仅供 dashboard 使用：id、阶段、分数、趋势、状态、最后更新时间
├── _template.md
└── <app>-<名字>-<match日期>.md   # 例：hinge-sarah-0818.md —— 一个 match 一个文件
self/
├── inbox.md                # 只增不改，脱敏后的结果观察记录
└── policy.md                # 汇总后的"对这个用户有效的打法"——跨 match 共享
```

match id 命名规则：`<app>-<名字>-<match日期>`，不能只用名字——dating app 上名字重名的概率很高（见隔离决策里对归属网关的讨论）。

### 生命周期状态（放在 `_index.md` 上，不放在单个档案里）

`active`（还在聊）/ `quiet`（没回复，观察中）/ `ghosted`（没回复 ≥ N 天）/ `archived`（已 unmatch 或用户主动结束）。`archived` 的条目默认不进入归属网关的候选池，这样能同时降低干扰和在 dating app 规模下（可能同时 10-30+ 个活跃 match，远超这套模式借鉴的 IRL 档案模式所假设的"几个人"）的误判风险。

## 还悬而未决的事

见 [handoff.md](handoff.md)。

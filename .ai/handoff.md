# 当前状态

**最后更新：2026-08-21**

> 这个文件会过期——每次工作阶段结束时更新它。想要长期保持真实的内容应该放进
> `architecture.md` 或 `decisions/`；路由规则见 [README](README.md)。

## 现在进展到哪

纯设计阶段——还没有 SKILL.md，没有 reference 文件，没有代码。目前存在的东西：

- `gpt对话记录.txt` —— 这个仓库连上 GitHub 之前，跟 GPT 一起想出来的原始多 Agent 设计
- `.ai/architecture.md` —— 本次讨论达成一致的 v1 形态：Skill 形式、三个入口（单对象对话 / self-model 汇总 / dashboard 概览）、文件结构
- `.ai/decisions/` —— 三条已定决策：match 隔离靠一对象一对话、v1 先做 Skill 不做写代码的多 Agent 系统、MBTI/好感度打分保留为产品层但底层是行为推断
- `.claude/skills/fieldnotes/` —— 已作为项目内 skill 安装（`.ai/` 这套结构本身的来源）

三个参考仓库克隆到了 scratchpad 里做对比用（没有 vendor 进这个仓库）：`hotcoffeeshake/tong-jincheng-skill`、`Mayuqi-crypto/HowToGetAlongWithGirls`、`ZQC8848/Fieldnotes`（这套 `.ai/` 模式的来源）。

## 下一步

设计 dating app 专属的阶段模型和信号库——这是唯一没有现成参照可以直接用的部分（HowToGetAlongWithGirls 的 lifecycle/signal 文件是为线下/长期关系设计的，已经在决策里明确否掉了直接照搬；见 `decisions/mvp-skill-form-not-multiagent.md`）。具体来说：把 `吸引→平台期→暧昧→关系→长期→婚姻` 换成适合 `匹配 → 开场白 → 筛选/来回调侃 → 势头推进 → 转移到app外/第一次见面` 的阶段模型，并把"信号识别三铁律"（看行为不看话语 / 看趋势不看单次 / 看对比不看绝对值）适配成 dating app 原生的信号（unmatch 风险、已读不回、prompt/语音消息互动率、主动要求转移到其他平台作为关系升级信号），而不是微信/朋友圈那套信号。

这部分定下来之后，再起草 SKILL.md 本身（路由表、architecture.md 里的三个入口、把归属网关写成硬规则）。

## 进行中 / 待定

- `profiles/_index.md` 的具体字段结构和 `profiles/_template.md`——architecture.md 里只是高层描述，还没写成实际文件。
- `self/inbox.md` 的条目格式——需要脱敏到绝不泄漏 match 可识别细节的程度，格式还没定。
- self-model 汇总这一步在当前开发阶段的环境里具体怎么触发——是用 Claude Code 的 Agent 工具一次性调用，还是别的机制。

## 会让人踩坑的地方

- `profiles/`、`self/`、`.claude/skills/dating-coach/`（或者最终 skill 会取的任何名字）现在都还不存在——architecture.md 描述的是意图，不是磁盘上的文件。
- 三个参考仓库放在会话的 scratchpad 里（`.../scratchpad/refs/...`），不在这个仓库里——如果开新会话需要重新看，得重新 clone。

# context-compression

一个用于长会话压缩的跨 Agent skill。

它关注的不是把上下文压得更短，而是压缩后还能继续做对的事。

## 这个 skill 解决什么问题

长对话经过压缩后，常见问题是：
- 只留下最早任务，丢掉后来的方向修正
- 文件、链接、ID、错误信息被压没
- 有总结，但没有下一步
- 旧 TODO 残留，被误当成当前目标
- 多阶段任务失去顺序

这个 skill 的目标是让压缩后的上下文仍然可执行。

## 它保留什么

相比普通 summarization，它强制保留：
- 当前目标
- 最近一次方向变更
- timeline（done / now / next）
- active TODOs
- 已改动文件 / 资产
- blocker
- 下一步动作

## 关键设计

### 当前目标优先
压缩恢复后，用户最新一条消息优先级最高。

核心规则：
- latest user turn beats compressed summary
- latest user turn beats old todo state

### timeline
每次压缩后都保留最小时间线：
- done
- now
- next

### TODO
当前工作必须保留成明确的优先级列表。

规则：
- 只能有一个 `in_progress`
- 用户一旦改方向，TODO 必须重建
- stale / cancelled 的旧任务不能悄悄复活

## 适用场景

- 超长会话
- 多阶段开发 / 发布 / 写作任务
- 容易被压缩打断的连续工作
- 需要跨多轮保持主线的 agent

## 文件
- `SKILL.md`：主 skill
- `skill.json`：安装元数据
- `README.md`：中文说明
- `README_EN.md`：英文说明

## Agent 兼容矩阵

| Agent | 兼容度 | 说明 |
|---|---|---|
| Claude Code | 高 | 原生支持 skills / SKILL.md，最接近当前结构 |
| Cline | 高 | 支持 Skills 与 SKILL.md，迁移成本很低 |
| Cursor | 高 | 支持 Skills / Rules / Subagents，核心内容可直接迁移 |
| Codex | 中高 | 支持 project-local skills，但目录与包装需适配 |
| Gemini CLI | 中高 | 核心方法可迁移，通常要改写成 GEMINI.md / .gemini 风格 |
| OpenHands | 中高 | 支持 skills 与 AGENTS.md，但更适合做成 OpenHands 风格封装 |
| Hermes | 高 | 同样适用 |

## 怎么理解这个仓库

这不是某一个 Agent 独占的格式。

它更接近一种开源精神下的通用方法：
- 优先服务更广泛的 Agent 生态
- 不把方法论锁死在单一产品里
- 鼓励不同 Agent 按自己的方式适配与演化

真正可迁移的是：
- 压缩后保留当前目标
- latest user turn beats stale summary
- timeline（done / now / next）
- active TODOs
- blocker / next action

真正需要按 Agent 换壳的是：
- 安装命令
- 目录结构
- frontmatter / 元数据
- 自动触发方式

所以更准确的说法是：
这是一个跨 Agent 可迁移的 context engineering skill，遵循开源精神，欢迎不同 Agent 生态复用、改写、再分发，而不是单一平台专属包。

## 一句话

好压缩，不是更短。
是压完之后还能继续把事做对。

# context-compression

一个用于长会话压缩的 Hermes skill。

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

## 安装

```bash
npx skills add <owner>/<repo>
```

或手动把这个目录放进本地 skills 目录。

## 一句话

好压缩，不是更短。
是压完之后还能继续把事做对。

# context-compression

一个公开可分发的 Hermes skill。

它解决的不是“怎么把上下文压短”这么表面的事。
它解决的是：压缩之后，Agent 为什么会继续做错的事。

## 这个 skill 解决什么问题

长对话一旦经过压缩，最容易坏在这几件事上：
- 只留下最早任务，丢掉后来的方向修正
- 文件、链接、ID、错误信息被压没
- 看起来有总结，实际上没有下一步
- 旧 TODO 残留，被误当成当前目标
- 多阶段任务失去顺序，Agent 不知道现在处于哪一段

这个 skill 的目标是：
让压缩后的上下文仍然可继续执行，而不是只剩“摘要感”。

## 它增加了什么

相比普通 summarization，它额外强制保留：
- 当前目标
- 最近一次方向变更
- timeline（done / now / next）
- active TODOs
- 已改动文件 / 资产
- blocker
- 下一步动作

也就是把“规划骨架”一起压进去。

## 关键设计

### 1. 当前目标优先
压缩恢复后，不允许旧 summary、旧 todo 压过用户最新一条消息。

核心规则：
- latest user turn beats compressed summary
- latest user turn beats old todo state

### 2. timeline 元素
这个 skill 要求每次压缩后都保留时间线，不只是事实列表。

最小 timeline：
- done
- now
- next

这样压缩后还能知道：
- 哪些阶段已经完成
- 当前卡在哪
- 下一步该接什么

### 3. TODO 元素
它要求把当前工作保留成明确的优先级列表，而不是散落在叙述里。

规则：
- 只能有一个 `in_progress`
- 用户一旦改方向，TODO 必须重建
- stale / cancelled 的旧任务不能悄悄复活

## 适用场景

- 超长会话
- 多阶段开发 / 发布 /写作任务
- 容易被压缩打断的连续工作
- 需要跨多轮保持主线的 agent
- 用户已经指出“你又回到旧任务了”这类问题时

## 文件
- `SKILL.md`：主 skill
- `skill.json`：安装元数据
- `README.md`：中文说明
- `README_EN.md`：英文说明

## 安装

计划安装方式：

```bash
npx skills add <owner>/<repo>
```

或手动把这个目录放进本地 skills 目录。

## 发布后要做的事

推到 GitHub 后，把 `skill.json` 里的占位链接替换成真实地址：
- `https://github.com/<owner>/context-compression-skill`
- `https://raw.githubusercontent.com/<owner>/context-compression-skill/main/SKILL.md`

## 一句话

好压缩，不是更短。
是压完之后还能继续把事做对。

# Agent Loop：Agent、Turn、Step 与工具调用

## 这一章的叙事主线

一个用户目标可能需要多次模型请求。DSH 用 Agent、Inbox、Turn、Step、Live Event 和 Session Event 把这个循环拆开，使输入接管、模型请求、工具执行、错误恢复和停止边界都有明确位置。

## 组件词典

| English | 中文 | 角色 | 状态 |
| --- | --- | --- | --- |
| Agent | 智能体句柄 | 当前运行和输入接管 | idle / running |
| AgentRegistry | 智能体注册表 | 跟踪 live Agent | registered / disposed |
| Inbox | 输入收件箱 | 接收 followup、steer、inject | pending / claimed |
| Turn | 轮次 | 一次输入处理外壳 | open / stopping / ended |
| Step | 步骤 | 一次模型请求及其工具执行 | started / ended |
| Round | 策略轮次 | Goal/Ralph 等外层计数 | active / complete |
| Waterfall | 瀑布事件 | 可 `next()` 的控制链 | entered / delegated / rejected |

## 完整流程

```text
turn/start [Durable]
-> claim inbox [Live]
-> agent/pre-step [Live]
-> step/start [Durable]
-> agent/request -> llm/stream [Live]
-> assistant/message [Durable]
-> tools/pre-execute -> execute -> post-execute [Live]
-> tool/result [Durable]
-> step/end [Durable]
-> turn-stopping [Live]
-> turn/end [Durable]
```

一个 Turn 可以包含多个 Step；工具失败可能让模型修正参数、重新读取环境或请求批准，不等于立刻退出。`agent/pre-step` 可以拒绝或改写输入；即使没有 Step，Turn 仍可能作为“发生过一次尝试”的事实关闭。

## Inbox 与取消

多个来源不能各自启动 Loop，否则会并发修改同一 Session。输入先进入 inbox，再在 Turn/Step 边界 claim。取消则从 Agent handle 穿透 stream、Tool Provider 和下一步决策：

```text
cancel -> LLM AbortSignal -> Tool AbortSignal -> no next step
```

## 普通 Agent 与 DSH

普通 while 循环容易实现，但通常没有明确的 claim、Turn、Live/Durable、取消和错误恢复协议。DSH 更重，却能被 UI、审批、遥测、Scope 和恢复机制观察与介入。

源码导航：`docs/agent-lifecycle.md`、`packages/core/agent`、`packages/core/agent-loop/README.zh.md`、`docs/subsystems/core.md`。

## 下一章要解决的问题

Loop 正在运行时会产生很多状态：哪些输入已经 claim，哪个工具正在执行，哪些结果必须留下，刷新后如何恢复。下一章把这些状态按 Config、Live、Durable 和 Derived 四种时间尺度拆开。

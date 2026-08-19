# Agent、Turn 与 Step 如何运转

## 这一章的叙事主线

用户发送一次消息，为什么可能触发多次模型请求？本章把 DSH 的 Agent Loop 拆成 Turn 和 Step，并跟踪一条输入如何变成模型请求、工具调用和下一步输入。

## 从一次请求到多步执行

```text
用户输入
-> claim inbox
-> 组装 prompt sections + tool schemas
-> agent/pre-step
-> agent/request
-> llm/stream
-> tool calls
-> tools pipeline
-> tool results
-> 下一次 request 或 turn end
```

一个 **Step** 是一次模型请求以及它引起的工具执行。一个 **Turn** 是从输入被接管到系统确认没有未完成工作的外层过程，所以一个 Turn 可以包含零个或多个 Step。

## Turn、Step、Round 不要混用

- Turn 是 Agent 对一批 admitted input 的一次处理。
- Step 是其中的一次模型请求。
- Round 是更外层策略的计数单位，例如 Goal continuation 或 Ralph workflow。

如果把所有计数都叫“轮次”，最大步数、目标续作次数和用户对话轮次会互相污染，停止策略也会变得不可解释。

## Agent Loop 的关键事件

DSH 的流程可以压缩成一条带有持久和实时边界的链：

```text
turn/start [durable]
  -> agent/pre-step [live, 可拒绝或改写]
  -> step/start [durable]
  -> agent/request [live]
  -> llm/stream [live]
  -> assistant/message [durable]
  -> tools/pre-execute [live]
  -> tools/execute [live]
  -> tool/result [durable]
  -> step/end [durable]
  -> agent/turn-stopping [live]
  -> turn/end [durable]
```

`agent/pre-step` 的特殊之处在于它决定模型看到什么：监听器可以重写已 claim 的消息，也可以拒绝这一步。即使第一次 claim 被拒绝，系统仍可能记录一个没有 Step 的 Durable Turn，因为“用户尝试过一次”本身是事实。

## 为什么不是简单 while 循环

表面代码可以写成：有 tool call 就执行，没有就结束。但 DSH 还要处理取消、队列输入、工具失败、空的第一次 claim、事件顺序和日志一致性。Loop 的职责是编排这些状态，而不是把所有工具策略写在一起。

## 一个 Turn 中有多个 Step

用户说“找出支付模块的超时问题并修复”，一次 Turn 可能包含：

```text
Step 1 Grep 搜索配置
-> Step 2 ReadFile 阅读调用链
-> Step 3 EditFile 修改逻辑
-> Step 4 Bash 运行测试
-> Step 5 根据失败结果继续修改
```

这不是五次用户输入，而是一次目标驱动的连续推进。工具失败也不一定结束：参数错误可能重试，环境变化可能重新读取，权限拒绝则可能等待用户决定。

## 取消为什么需要跨层传播

```text
Agent handle 收到取消
-> LLM stream 停止
-> Tool Provider 收到 AbortSignal
-> Loop 在 step/turn 边界不再继续
```

只停止 UI 的 spinner 不算取消；Host 进程、模型请求和 Session 事实都必须有清楚的停止语义。

## 本章结论

Agent Loop 是一台受事件约束的流程机器：它负责推进，不负责垄断能力。理解 Turn 和 Step 后，下一章才能准确判断哪些信息应进入可恢复日志，哪些只在运行时活着。

源码导航：`docs/agent-lifecycle.md`、`docs/architecture.md`、`packages/core/agent`、`packages/core/agent-loop`。

## 普通 while Loop vs DSH Agent Loop

普通 Loop 通常只判断“有没有 tool call”。DSH 还要处理 inbox claim、Turn/Step 边界、取消传播、空 Turn、事件拦截、工具错误和 Durable 记录。

普通 Loop 的优势是容易复制；DSH Loop 的优势是能被 UI、审批、遥测、Scope 和恢复机制观察或介入。代价是事件契约更多，但复杂行为不再隐藏在一段不可分解的 while 循环里。

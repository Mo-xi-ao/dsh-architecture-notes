# DSH 中的状态与无状态设计

## 这一章的叙事主线

“无状态”经常被当成一种现代系统的口号，但 Agent 系统不可能真的没有状态。模型请求需要历史，工具执行需要当前上下文，浏览器需要恢复 UI，目标续作需要记住进度。真正的问题不是“有没有状态”，而是：**状态由谁拥有、是否可持久化、是否可以重建、是否允许跨边界传递**。

本章用一张更细的地图解释 DSH：哪些部分尽量无状态，哪些部分必须有状态，哪些状态是 Live 的，哪些状态是 Durable 的，以及为什么不能把它们全部塞进一个 `AgentState` 对象。

## 从一个刷新页面的场景开始

假设 Agent 正在执行：

```text
读取配置
-> 修改文件
-> 运行测试
-> 根据失败结果继续修复
```

这时浏览器刷新了。系统至少要回答四个问题：

- 模型之前看过哪些用户消息和工具结果？
- 当前 Turn 是否已经结束？
- 哪个工具正在执行，结果是否已经写入？
- 刷新后的 Client 怎样知道已经发生过什么？

如果所有信息只在浏览器内存里，刷新后就丢了。如果把一个正在执行的 `Agent` 对象整体序列化，又会遇到函数、连接、AbortSignal、Context 和操作系统进程无法可靠序列化的问题。

所以 DSH 采用的不是“全有状态”或“全无状态”，而是**状态分层**：把可以重建的事实留下，把只能服务当前过程的运行对象留在 Live 世界。

## 先区分四种状态

```text
配置状态       -> 启动前决定插件树
领域事实       -> Session Log 中可持久化、可回放
运行时状态     -> 当前 Agent、Context、Fiber、连接和取消控制
派生视图       -> 从事实和运行状态计算出的 UI / 查询结果
```

这四类状态的生命周期不同：

| 状态类型 | 典型例子 | 是否持久化 | 权威来源 | 是否可重建 |
| --- | --- | --- | --- | --- |
| 配置状态 | Profile、Bundle、Patch rows | 可以保存 | 配置层 | 通过重新加载组合 |
| 领域事实 | `user/message`、`tool/result`、`turn/end` | 是 | Session Log | 是 |
| 运行时状态 | Agent handle、Fiber、AbortSignal、Socket | 通常不直接保存 | 当前 Context | 需要重新创建 |
| 派生视图 | 当前消息列表、工具进度、UI projection | 通常不作为事实保存 | Event / Query projection | 是 |

最容易犯的错误，是把派生视图当成权威状态，把运行时对象当成可持久化事实。

## 哪些部分尽量无状态

这里的“无状态”不是完全没有输入，而是同样的输入可以得到同样的结果，或者组件不拥有跨请求的业务事实。

### LLM Adapter 的无状态面

模型 API 本身接收一次完整的 messages、tools 和 system prompt，然后返回一次 stream。Adapter 可以把供应商协议转换成 DSH 的统一消息和流事件，但不应该自己成为 Session 历史的所有者。

```text
messages + tools + options
-> LLM Adapter
-> normalized stream
```

下一次调用需要什么历史，由 Session Log 投影出的 model history 决定，而不是依赖上一次 Adapter 调用残留的对象。

### 工具执行函数的无状态面

一个工具的输入、执行上下文和 Provider 决定本次结果。工具可以使用当前的 `agent.ctx`、Shell 或 FS，但“模型以前调用过几次”不应隐式藏在一个模块级变量里。需要保存的工具事实应进入 `tool/call`、`tool/result` 等 Session Event。

### Remote descriptor 的无状态面

Typert 生成的 descriptor 描述方法、参数和 codec。它是跨 Host/Client 的协议合同，不是某个 live Agent 的快照。真正的 Agent 或 Context 在调用时通过 lookup provider 解析。

## 哪些部分必须有状态

### Session 是 Durable 状态的容器

Session Log 记录模型可见的历史和生命周期事实。它不是“方便 UI 显示的聊天数组”，而是恢复、fork、resume、transcript 和 telemetry 的共同来源。

```text
SessionEvent[]
-> deriveMessages()
-> model history

SessionEvent[]
-> UI projection
-> message list / tool status
```

同一条事实可以有多个投影，但不能让每个投影自己发明一份历史。

### Agent 是 Live 状态的容器

Agent handle 持有当前 inbox、取消控制、运行上下文和 scoped registrations。它需要状态，因为 Agent 正在做事；它又不能成为唯一事实来源，因为它会结束、卸载、重建或被另一个进程替代。

可以这样记：

```text
Agent 负责“现在如何继续”
Session 负责“已经发生了什么”
```

### Context 是能力状态的容器

Cordis Context 中的 service、listener、Fiber 和 Effect 都属于当前运行时。它们决定这次调用能使用什么能力，但不应该被粗暴地当作整个应用的可序列化业务状态。

## 状态与无状态的真正边界

有些组件不是简单地归入一边，而是同时拥有两面：

| 组件 | 无状态部分 | 有状态部分 |
| --- | --- | --- |
| Agent Loop | 推进规则和事件编排 | 当前 Turn、inbox、取消和 step 进度 |
| Tool Registry | 工具定义和 schema | scoped registration、限制和生命周期 |
| API Gateway | descriptor 校验和分派规则 | 当前 lookup provider、请求关联和取消 |
| Client Remote | 生成的调用合同 | mounted namespace、in-flight call |
| System Prompt | 组装规则 | 当前 scope 的注册和动态 section |
| Profile Loader | 合成算法 | 当前插件树、Fiber 和加载错误 |

这张表说明：架构讨论中“这个模块有状态吗”通常不是二元问题，而要继续问“哪一层有状态、谁是权威来源、状态是否跨边界”。

## Reducer 为什么重要

当事件不断追加时，系统需要得到“现在的状态”。Reducer 或 Projection 做的事情可以写成：

```text
初始状态
-> apply(turn/start)
-> apply(user/message)
-> apply(tool/call)
-> apply(tool/result)
-> apply(turn/end)
-> 当前投影状态
```

所谓字段级更新，是某个事件只改变它负责的状态字段：工具结果更新对应调用的 `status`、`output` 和 `error`；新的 assistant chunk 更新对应消息的增量内容。它不是把任意对象做深合并，也不是让 UI 自己维护第二份真相。

## 一个失败的设计：把所有状态放进 Agent

假设我们设计：

```ts
type AgentState = {
  messages: Message[]
  currentTool: ToolCall
  socket: WebSocket
  abortController: AbortController
  context: Context
}
```

这个对象看起来方便，但它同时混合了：

- Durable 事实：`messages`。
- Live 控制：`abortController`、`socket`。
- Cordis 运行时：`context`。
- 派生进度：`currentTool`。

结果是无法可靠序列化，刷新无法恢复，测试必须构造真实 Socket，UI 也会依赖 Agent 内部字段。DSH 的分层正是在阻止这种“万能状态对象”。

## 一个成功的恢复流程

```text
Client 断开
-> Host 继续或停止当前 Live Agent
-> Session Event 持续成为事实来源
-> Client 重新连接
-> 根据 Session Log 建立 projection
-> 通过 Remote / event frame 获取当前可见状态
-> 必要时重新 resume 一个 Agent
```

注意：恢复不是把旧 Agent 从磁盘反序列化回来，而是从 Durable 事实重建足够的状态，再创建新的 Live 运行对象。

## 为什么这和无状态 LLM API 是同一条设计线

LLM API 无状态，意味着客户端每次都要提供完整上下文。DSH 没有试图把“记忆”塞进模型或某个不可见的 Provider，而是明确让 Session Log 承担历史，让 Adapter 保持协议转换，让 Agent Loop 决定下一步。

```text
无状态 LLM API
-> 客户端提供可重建的 model history
-> Agent 通过 Live Loop 推进
-> Session 保存新的 Durable facts
```

这条链把记忆、行动和协议分开，因此模型可以替换，Agent 可以恢复，Client 可以重连。

## 本章结论

DSH 的状态设计不是“尽量无状态”，而是把状态放在正确的时间尺度上：

- 配置状态决定启动时装载什么。
- Session 状态记录已经发生的事实。
- Agent/Context 状态服务当前运行。
- Projection 状态服务查询和渲染。
- LLM Adapter、Remote codec 和部分执行逻辑尽量保持输入驱动。

最终可以用一句话记住：**Durable 状态回答“发生过什么”，Live 状态回答“现在怎么继续”，无状态机制回答“给我输入，我如何处理”。**

源码导航：`docs/architecture.md`、`docs/subsystems/session.md`、`docs/subsystems/core.md`、`docs/api-gateway.md`、`packages/core/session`、`packages/core/agent`。


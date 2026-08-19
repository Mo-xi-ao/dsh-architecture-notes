# Session Log：Live 与 Durable

## 这一章的叙事主线

刷新页面后，为什么 DSH 还能恢复模型已经说过的话和工具结果？反过来，为什么一个正在执行的 `AbortSignal` 不能被简单写进日志？答案在于 Live 与 Durable 是两条不同的轨道。

## 两条轨道

```text
Live：当前运行中的对象、控制权、拦截点、流式增量
  -> 让过程发生

Durable：SessionEvent、消息、工具结果、Turn 边界
  -> 让事实可恢复
```

Live 事件服务正在发生的过程，例如 `agent/request` 可以拦截请求，`llm/stream` 可以消费流。Durable Session Event 是已经发生、应当能重放的事实，例如 `assistant/message` 和 `tool/result`。

## Model-visible means logged

DSH 的强约束可以这样说：凡是进入模型请求、并且会影响模型判断的事实，都必须能从 Session Log 重建。否则一次运行看似成功，恢复后模型会面对一个不同的世界。

这也是为什么新的模型可见输入不能只存在某个内存变量里，而要扩展 `SessionEventMap`，再从日志渲染成模型历史。

## Reducer、Projection 与 Patch 的区别

三个词都涉及“更新”，但层级完全不同：

| 机制 | 发生时间 | 输入 | 结果 |
|---|---|---|---|
| Profile Patch | 启动前 | 配置行 | 新的插件组合 |
| Reducer | 运行时读取 | 一个个事件 | 更新后的状态字段 |
| Projection | 读取或回放 | 事件流 | 面向查询或 UI 的派生视图 |

“字段级 Reducer 合并”指投影器收到事件后，只更新受该事件影响的状态字段。例如 `tool/result` 更新某个工具调用的结果和状态，不等于把任意两个配置对象做深合并。Session Log 仍是事实来源，状态对象只是从事实计算出来的方便视图。

## 为什么流式 chunk 也要记录

最终的 `assistant/message` 能满足模型历史，但 UI 还需要看到流式过程，调试和回放也可能需要原始边界。因此 DSH 保留 `assistant/chunk` 等事件，让“最终内容”和“过程 fidelity”都能从日志派生，而不是要求 UI 自己猜。

## Fork、Resume 和 Durable

Fork 是从事件边界建立新的 Session 事实流；Resume 是根据已有事实重新获得一个可运行 Agent。二者都依赖日志，而不是依赖某个旧的 live Agent 对象。Live 对象会被销毁，Durable 事实才能跨进程、刷新和重启存在。

## 本章结论

Live 让系统能够控制现在，Durable 让系统能够解释过去。把二者混在一起会导致两种错误：要么无法恢复，要么把不可序列化的运行时对象伪装成领域事实。

源码导航：`docs/architecture.md` 的 Session log 与 Events、`docs/subsystems/session.md`、`packages/core/session`。


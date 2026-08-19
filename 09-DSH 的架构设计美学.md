# DSH 的架构设计美学

## 这一章的叙事主线

前八章讲了机制。本章不再增加一个新组件，而是把那些机制放在一起，观察它们反复表达的设计判断。所谓架构美学，不是给代码加诗意，而是看一个系统在变化、失败、恢复和替换时是否仍然保持清楚。

## 无特权核心

“一切皆插件”意味着 Agent、Tools、Session 和 LLM 都通过 Context 进入系统。没有一个超级核心可以绕过生命周期规则。它带来的实际结果是：能力可以被替换，测试可以局部装载，卸载可以验证副作用是否归零。

## 组合优于分叉

Profile、Bundle 和 Patch 把 Web、Headless 和用户定制表达为不同组合，而不是不同实现分支。Patch 的整行替换语义也体现了这种克制：覆盖者必须明确自己要运行什么，系统不偷偷替它深合并未知字段。

## 事实与过程分离

Live event 让现在的过程可拦截，Durable Session Event 让过去的事实可恢复。两者分离后，刷新页面不会要求旧的 live 对象复活；同时，运行中的审批和取消也不会被误写成可重放的领域事实。

## 机制与策略分离

Agent Loop 提供推进机制，Provider、Listener、Restriction、Approval 和 Profile 决定策略。同一个 Loop 可以接本地 Shell 或 Sandbox；同一个 Tool Pipeline 可以被不同权限配置约束。机制稳定，策略可组合，这是 Harness 能承受产品变化的原因。

## 边界比实现更稳定

Capability Seam 让 Definition、Provider 和 Consumer 各自承担清楚责任。Remote 只允许显式方法跨 Host/Client 边界。边界设计得越清楚，内部实现越可以更换，而不把变化扩散给所有调用者。

## 生命周期必须可逆

插件注册、事件监听、Remote mount 和 scoped contribution 都应该有对应的退出路径。可逆性不是“开发模式才需要”的便利，它是判断一个扩展是否真正拥有边界的测试：卸载之后，旧能力是否从 Prompt、Registry、事件和连接中一起消失？

## 总结整套系统

```text
配置层决定组合
-> Loader 建立插件树
-> Context 提供可注入能力
-> Agent Loop 推进 Live 过程
-> Tool / LLM / Provider 执行动作
-> Session Log 保存 Durable 事实
-> Reducer / Projection 重建状态
-> Host / Client 在协议边界上消费
```

DSH 的美学最终可以压缩成一句话：让变化发生在它所属的边界，让事实留在可恢复的日志，让实现通过可逆的组合进入系统。

## 一套架构判断题

面对新功能，可以这样判断它应该放在哪里：

| 问题 | 倾向的落点 |
| --- | --- |
| 只改变产品组合？ | Profile / Bundle / Patch |
| 需要替换实现？ | Capability Seam |
| 只观察或拦截运行过程？ | Live Event |
| 影响模型未来判断且需恢复？ | Session Event |
| 只服务当前 UI？ | Projection |
| 需要跨 Host/Client 调用？ | Remote 或专用数据协议 |

这不是机械分类表，但它能迫使设计者先说明状态、生命周期和边界，再开始写代码。

## 本章结论

一个好的 Harness 不会让所有模块看起来一样，而是让每种变化都有合适的落点：产品差异去 Profile，能力替换去 Seam，运行控制去 Live Event，恢复事实去 Session Log，跨进程调用去 Remote。系统因此不是没有复杂度，而是把复杂度放在可以解释、可以测试、可以撤销的位置。

源码导航：`docs/architecture.md`、`docs/glossary.md`、`docs/capability-seams.md`、`docs/api-gateway.md`。

## DSH 的优势取决于问题规模

一个十秒结束的单工具脚本不一定需要完整 Harness。普通 Agent 更轻、更容易部署；DSH 的优势在长会话、多运行模式、多执行环境、跨进程 UI、动态扩展和恢复需求出现时才会显现。

所以 DSH 不是“功能更多就一定更好”，而是把必要复杂度放到可命名、可测试、可替换和可撤销的边界。架构判断的关键不是追求最少抽象，而是避免未来所有变化都挤进同一个主循环。

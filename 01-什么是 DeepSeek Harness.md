# 什么是 DeepSeek Harness

## 这一章的叙事主线

先不要把 DSH 想成一个“带聊天界面的模型客户端”。本章从一个任务开始：用户让 Agent 修改代码、运行测试、根据报错继续修复。我们沿着这条任务链看见 DSH 的整体形状，再回答 Harness 比普通 Agent 多承担了什么。

## 从一个场景开始

用户输入：“修复登录接口的空指针问题，并运行测试。”

一个只会调用 LLM 的程序可以给出修改建议，但它不能可靠地完成闭环。真正的系统需要：

- 知道当前运行的是哪个 Profile。
- 读取已组合的工具、模型和权限配置。
- 让 Agent 选择工具并执行。
- 将工具结果写入可回放的 Session Log。
- 在下一步模型请求中重新构造上下文。
- 让浏览器看到过程，让刷新后的页面仍能恢复结果。

这就是 Harness 的职责：不是替模型思考，而是提供一个能承载模型决策、环境行动和事实恢复的运行环境。

## LLM、Agent 和 Harness 的三层区别

可以先用三层理解：

```text
LLM       -> 根据输入生成下一段输出
Agent     -> 让 LLM 通过工具和循环持续推进目标
Harness   -> 组织 Agent 的能力、生命周期、事实、权限和边界
```

LLM 是决策部件，Agent 是行动循环，Harness 则负责“这个循环在什么世界里运行”。在 DSH 中，模型适配器、工具注册表、Session Log 和 Agent Loop 本身都是插件，意味着它们不是被写死在一个超级核心对象里。

## DSH 的第一张地图

```text
CLI 选择 Profile
  -> Bundle 与 Patch 形成配置层
  -> Loader 装载 Cordis Plugin Tree
  -> Agent 使用 Session / System Prompt / LLM / Tools
  -> 工具访问 FS / Shell / Sandbox / Subagent
  -> Session Event 投影到 Web、CLI 和持久化层
```

这条链有一个重要方向：配置从上往下决定能力，事件从运行时向外传播事实。UI 不应该反过来成为 Agent 内部状态的唯一来源。

## 为什么 DSH 不是一个“更大的 Agent Loop”

如果所有事情都写进 Loop，替换模型、增加沙箱、支持 Web、恢复历史都会变成修改同一个中心。DSH 把这些责任分开：

- Loop 决定何时进入下一步。
- Session 决定哪些事实可被重放。
- Tool Pipeline 决定一次工具请求如何被校验、审批、执行和记录。
- Profile 决定本次运行装载哪些插件。
- Client 只消费 Host 暴露的投影和 Remote 能力。

这不是为了增加文件数量，而是为了让变化有地方落。一个功能如果无法说明自己属于哪个能力边界，通常意味着设计还没有完成。

## 本章结论

DSH 的核心不是“让模型更聪明”，而是把模型、工具、状态、配置和边界组织成可组合的运行时。后面所有章节都在回答同一件事：如何让这个运行时既能扩展，又能恢复和替换。

源码导航：`docs/architecture.md`、根目录 `AGENTS.md`、`packages/core/agent`、`packages/core/session`。


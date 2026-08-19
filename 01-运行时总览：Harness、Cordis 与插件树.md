# 运行时总览：Harness、Cordis 与插件树

## 这一章的叙事主线

普通 Agent 通常从一个 `while` 循环开始：收消息、调用模型、执行工具、继续调用。DSH 要解决的是更大的问题：这个循环由哪些能力组成，能力如何进入运行时，卸载时如何退出，刷新后如何恢复，Web 和 Headless 如何使用不同组合。

## 从一个请求看见整棵树

```text
用户输入
-> Client / Connection
-> Host Agent
-> Session + System Prompt + Tool Registry
-> LLM Adapter
-> Tool Provider / Sandbox / FS
-> Session Events
-> Client Projection
```

这条链不是一个类内部的方法调用，而是一棵 Cordis Plugin Tree。模型、Session、Tools、Agent Loop 都是插件，Context 是它们共享能力的运行边界。

## Harness、Agent、Cordis 的分工

| English | 中文 | 主要职责 | 状态类型 |
| --- | --- | --- | --- |
| Harness | 智能体运行时 | 组织能力、生命周期、恢复和边界 | 组合状态 |
| Agent | 智能体 | 推进当前目标和输入队列 | Live |
| Cordis Context | 上下文 | 承载 Service、Event、Effect | Runtime |
| Plugin | 插件 | 安装能力并绑定生命周期 | Configured / Runtime |
| Service | 服务 | 提供稳定能力词汇 | Runtime |
| Effect | 可逆副作用 | 注册和撤销监听、工具、资源 | Lifecycle |

## 为什么“一切皆插件”不是目录口号

普通模块化只把代码拆成文件，消费者仍可能直接 import 具体实现。DSH 的插件化多了三个约束：能力通过 Context 注入，注册属于 Effect，进入系统必须经过 Loader/Bundle/Profile 的接纳。

因此替换本地 Shell、沙箱 Provider 或模型适配器时，Consumer 不必复制一份。代价是依赖关系不只存在于 import，还存在于 Service key、scope 和生命周期。

## 插件的状态流转

```text
configured
-> loading
-> mounted
-> active
-> disposing
-> disposed
```

这里的阶段用于理解 Loader 生命周期；具体插件可能暴露 `pending`、`failed` 或 live registry 状态。关键不是记住一个枚举，而是知道“能力存在”本身就是状态，并且必须可撤销。

## 普通 Agent 与 DSH

普通 Agent 的优势是短、直观、容易复制；DSH 的优势在长会话、多 Provider、HMR、测试隔离和多产品组合。DSH 不是让十秒脚本更简单，而是避免所有变化都挤进主循环。

源码导航：`docs/architecture.md`、`docs/cordis-primer.md`、`packages/boot/app-boot`、`packages/core/agent`。

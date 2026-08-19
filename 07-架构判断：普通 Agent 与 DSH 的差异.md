# 架构判断：普通 Agent 与 DSH 的差异

## 这一章的叙事主线

前面的章节不是在证明 DSH 永远更好，而是在建立判断标准：什么时候一个普通 Agent 足够，什么时候需要 Harness 的配置、状态、Seam 和边界协议。

## 差异总表

| 维度 | 普通 Agent | DSH |
| --- | --- | --- |
| 主循环 | 一个 while | Agent Event / Turn / Step |
| 历史 | messages 数组 | Session Event + Projection |
| 能力 | 直接 import 工具 | Service / Provider / Consumer |
| 配置 | 环境变量和 if-else | Profile / Bundle / Patch |
| 安全 | Prompt + execute 内判断 | Scope、Restriction、Approval、Sandbox |
| UI | 直接读内存状态 | Host/Client protocol + projection |
| 恢复 | 重新开始 | replay、resume、fork |
| 卸载 | 通常重启进程 | Effect disposer |

## DSH 的新观念

DSH 把“状态、能力和边界”都变成可组合的设计对象：状态有权威来源，能力有 Definition/Provider/Consumer，跨进程调用有 Remote contract，注册本身有生命周期。

这不是没有代价。事件词汇要演进，Projection 要测试，Loader 和 Context 增加认知负担，短脚本不值得承担全部设施。但当任务跨越长会话、多模型、多执行环境和多界面时，这些边界能阻止复杂度集中爆炸。

## 新功能应该放在哪里

```text
改变产品组合       -> Profile / Bundle / Patch
替换能力实现       -> Capability Seam
拦截当前流程       -> Live Event / Waterfall
影响恢复和模型历史 -> Session Event
只服务查询和 UI    -> Projection
跨 Host/Client      -> Remote 或专用数据协议
```

## 本章结论

普通 Agent 追求“先跑起来”，DSH 追求“运行很久以后仍能解释、恢复、替换和撤销”。两者不是高低关系，而是问题规模不同。DSH 的架构美学可以收束为：让变化发生在所属边界，让事实留在可重放日志，让实现通过可逆组合进入系统。

源码导航：`docs/architecture.md`、`docs/glossary.md`、`docs/api-gateway.md`。

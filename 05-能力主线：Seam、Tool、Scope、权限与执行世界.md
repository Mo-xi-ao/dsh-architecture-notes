# 能力主线：Seam、Tool、Scope、权限与执行世界

## 这一章的叙事主线

模型请求工具只是开始。DSH 还要回答工具是否对当前 Agent 可见、参数是否有效、是否需要批准、在哪个执行世界里运行、结果如何记录，以及 Provider 卸载后能力是否消失。

## Capability Seam 的三角色

| English | 中文 | Shell 示例 |
| --- | --- | --- |
| Service Definition | 服务定义 | `dsh-shell` |
| Service Provider | 服务提供者 | `dsh-bash-local` / sandbox |
| Consumer | 能力消费者 | `dsh-tool-bash` |

普通 interface 只描述方法签名；Seam 还描述注册、选择、错误、取消和卸载。换 Provider 不应要求 Consumer 分叉。

## Tool Pipeline 的状态流

```text
registered
-> visible in schema
-> requested by model
-> pre-execute
-> approved / rejected
-> executing
-> completed / errored / cancelled
-> tool/result durable
```

`registered` 和 `visible` 是 Registry/Scope 状态，`pre-execute` 是 Live 控制，`tool/result` 是 Durable 事实。把它们全部塞进 `execute()` 会失去观察和恢复边界。

## Scope 与 Restriction

全局工具对所有 Agent 可见；`agent.ctx.tools.register()` 注册到当前 Agent scope。Restriction 先过滤全局集合，再加入 scoped registration。被过滤的工具既不进入 Prompt，也不应在执行阶段被绕过。

## 执行世界

Shell、FS、Subprocess、PTY 和 LSP 如果看到不同文件视图，Agent 就会读一个世界、改另一个世界。DSH 通过 Capability Seam 让本地或沙箱 Provider 一起决定执行世界。

## 普通 Tool Calling 与 DSH

普通工具 `{ name, description, execute }` 适合演示；DSH 的 Registry、Schema、Scope、Approval、Waterfall、Provider 和 Durable result 更重，但能支持长任务、安全策略、并发调度和审计。

源码导航：`docs/capability-seams.md`、`docs/tool-execution-pipeline.md`、`packages/core/tools`、`packages/shell`、`packages/sandbox`。

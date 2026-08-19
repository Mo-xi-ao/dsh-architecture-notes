# 边界主线：Host、Client、Remote 与 UI 投影

## 这一章的叙事主线

浏览器需要看到 Agent 和 Session，但不能直接获得 Host 的 live 对象。DSH 用 Host/Client 运行边界、Connection、Remote 和 Session projection 把执行世界与交互世界分开。

## 组件词典

| English | 中文 | 角色 | 状态 |
| --- | --- | --- | --- |
| Host | 宿主端 | 持有 Agent、Session、FS、Shell | live runtime |
| Client | 客户端 | 交互、渲染、提交输入 | UI state |
| Connection | 连接层 | RPC、事件帧、取消、信任检查 | connected / closed |
| Remote | 远程方法 | unary Host method contract | mounted / withdrawn |
| Gateway | 网关 | descriptor、lookup、调用和校验 | dispatching |
| Projection | 投影 | 从事件产生 UI/query view | derived |

## Remote 调用路径

```text
Client concrete method
-> Connection /api RPC
-> Typert Gateway
-> lookup Agent / scoped Context
-> live Host service
-> validate return
-> Client result
```

Client 传递的是 `agentId` 等 wire identity，Gateway 在 Host 侧找回 live Agent；不会把 Agent、Context、Socket 或 AbortController 序列化到浏览器。

Remote 只适合 unary method call。Session stream、增量 projection、分页和实体子流需要专用数据协议。`api-remotes` 显式选择 contribution，意味着 Host 新增方法不会自动暴露给所有 Client。

## 普通前后端 DTO 与 DSH

手写 DTO 更快，但字段、取消、身份解析和版本演进容易分叉。DSH 通过生成 descriptor、codec 和 Client contribution 形成严格合同；代价是必须遵守 Host/Client 双面构建和显式 assembly。

源码导航：`docs/api-gateway.md`、`packages/api/remotes`、`packages/api/gateway`、`packages/client/connection`。

## 下一章要解决的问题

到这里，系统的启动、运行、状态、能力和边界都已经出现。最后一章不再介绍新组件，而是把这些机制放在普通 Agent 旁边比较，回答 DSH 的复杂度究竟换来了什么。

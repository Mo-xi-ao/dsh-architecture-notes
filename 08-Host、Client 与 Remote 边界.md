# Host、Client 与 Remote 边界

## 这一章的叙事主线

浏览器界面需要看到 Agent 和 Session，但它不能直接拿到 Host 的 Agent 对象。这个限制不是麻烦，而是进程边界、类型边界和信任边界共同形成的架构约束。

## 两个运行世界

```text
Host：Agent / Session / LLM / FS / Shell / Sandbox / API Gateway
  <-> Connection / RPC / Event frames
Client：Web UI / Remote methods / Session projection / rendering
```

Host 持有真实运行时和副作用；Client 持有交互、渲染和用户操作。Client 看到的是协议和投影，不是一个可以任意调用内部方法的 live Host 对象。

## Remote 解决什么问题

`@Remote` 和 `@RemoteScope` 把明确允许跨边界的方法生成 Host/Client 合同。Typert 生成 descriptor、schema 和 Client contribution；Gateway 负责查找 descriptor、解析 identity、校验参数、调用 live service 和校验返回值。

```text
Client concrete method
-> Connection /api RPC
-> Typert Gateway
-> lookup Agent or scoped Context
-> live Cordis service
-> validated result
```

Remote 只处理 unary method call。Session event stream、增量数据和 projection 有自己的数据协议，不应伪装成 Remote 方法。

## 为什么需要显式 assembly

Client 的 `api-remotes` 会显式导入并 mount 被应用允许的 Remote contribution。这样 Host 新增一个 `@Remote` 方法，并不会自动让所有浏览器页面获得它。能力进入 Client 必须经过应用组合者的选择，这保留了产品所有权和信任边界。

## Remote 与 API Proxy 的分工

Connection 负责传输、RPC id、取消和响应 envelope；Gateway 负责 Typert descriptor 和业务分派；API Proxy 处理没有 Remote descriptor 的传统接口。分层的好处是未来替换传输 carrier，不需要改业务方法的 wire contract。

## Remote 不是所有数据的通道

Remote 适合“一次请求、一次结果”的 unary method。Session stream、增量 projection、分页和实体子流需要各自的数据协议。把所有东西都包装成 Remote，会让取消、背压、重连和回放语义变得含糊。

另一个边界是对象身份：Client 传递的是 `agentId` 或其他 wire identity，Gateway 在 Host 侧通过 lookup provider 找回 live Agent，而不是把 Agent 对象序列化到浏览器。

## 本章结论

Host/Client 不是简单的前后端目录划分，而是两个能力世界之间的协议边界。显式 Remote、严格生成和运行时校验共同防止“能在 Host 上调用”被误解为“浏览器自动拥有它”。

源码导航：`docs/api-gateway.md`、`packages/api/remotes`、`packages/api/gateway`、`packages/client/connection`。

## 普通前后端 DTO vs DSH Remote

普通应用常手写 DTO 和路由，前端字段、取消和对象身份靠约定维护。DSH 通过生成 descriptor、codec 和 Client contribution 形成更严格的合同，Gateway 在 Host 侧解析 lookup identity 并校验参数和返回值。

优势是 Host 仍掌握 live service 和权限，Client 只能获得显式 namespace；代价是必须遵守 Host/Client 有序构建，不能只改浏览器代码就期待自动发现新能力。

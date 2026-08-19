# 状态主线：Session、Live、Durable 与状态注册

## 这一章的叙事主线

DSH 最值得学习的地方不是“有状态”或“无状态”，而是把状态拆到不同时间尺度：配置决定启动，Live 状态控制现在，Durable Event 保存事实，Projection 服务查询和 UI。

## 四种状态

| English | 中文 | 例子 | 权威来源 |
| --- | --- | --- | --- |
| Config state | 配置状态 | Profile、Bundle、Patch | 配置层 |
| Live state | 运行状态 | Agent、Fiber、AbortSignal、pending call | 当前 Context |
| Durable fact | 持久事实 | `user/message`、`tool/result`、`turn/end` | Session Log |
| Derived state | 派生状态 | UI、SessionSurface、approval view | Reducer / Projection |

```text
Live：现在如何继续
Durable：已经发生什么
Derived：现在如何查看
```

## 为什么普通 messages 数组不够

普通 Agent 把 `messages[]` 当作历史、UI 状态和恢复依据。DSH 的 Session 是 append-only typed log，模型历史和 UI 都从事件投影而来。刷新时重建 Projection，而不是序列化旧 Agent。

## 一个新状态如何注册

以“工具等待审批”为例，状态不是直接加到 `AgentState`，而是沿不同领域流转：

```text
Approval Service Definition
-> Provider 持有 pending request [Live]
-> tools/pre-execute 控制是否继续 [Live]
-> approval/decided 写入 Session [Durable]
-> Reducer 生成 waiting/approved/rejected [Derived]
-> Client 渲染状态
-> Effect unload 撤销 Provider 和 listener
```

### Service：定义能力，不直接等于事实

```ts
export abstract class ApprovalService extends Service {
  constructor(ctx: Context) { super(ctx, 'approvals') }
  abstract request(input: ApprovalInput): Promise<'approved' | 'rejected'>
}
```

### SessionEventMap：声明可恢复事实

```ts
declare module '@deepseek-ai/dsh-session' {
  interface SessionEventMap {
    'approval/decided': {
      callId: string
      decision: 'approved' | 'rejected'
    }
  }
}
```

### Reducer：从事件计算视图

```ts
if (event.type === 'approval/decided') {
  state[event.data.callId] = { status: event.data.decision }
}
```

这个 Reducer 不弹窗、不执行工具、不写日志；它只计算状态，因此可以刷新、重放和测试。

## 新状态的注册检查表

1. 它是配置、事实、控制、缓存还是派生视图？
2. 重启后是否必须存在？
3. 模型恢复后是否必须看见？
4. 它属于 Agent、Session 还是 Context？
5. 并发事件的顺序是否影响结果？
6. 卸载时 pending operation 如何取消？

## DSH 相比普通 Agent 新在哪里

普通 Agent 把状态理解为对象字段；DSH 把状态理解为领域之间的协议。新增能力不只是“加字段”，而是选择 Event vocabulary、Live extension point、Projection 和 lifecycle owner。这带来恢复、替换、隔离和可测试性，也带来事件演进和投影维护成本。

源码导航：`docs/subsystems/session.md`、`docs/agent-lifecycle.md`、`packages/core/session`、`packages/core/agent`、`packages/core/agent-loop`。

## 下一章要解决的问题

状态边界明确后，还要回答能力如何进入这些边界：一个 Tool 怎样注册，一个 Provider 怎样替换，Scope 怎样限制可见性，沙箱怎样保证 Shell 和 FS 处在同一个执行世界。下一章进入 Capability Seam 和 Tool Pipeline。

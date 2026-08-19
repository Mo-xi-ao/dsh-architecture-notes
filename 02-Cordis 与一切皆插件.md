# Cordis 与一切皆插件

## 这一章的叙事主线

DSH 最容易被低估的设计是“每一部分都是插件”。这句话不是目录组织方式，而是运行时的权力分配：没有一个模块拥有永久特权，能力通过 Context 注入，注册通过 Effect 建立，卸载时可以撤销。

## 如果有一个超级核心会怎样

直觉方案是做一个 `Harness` 类，把模型、工具、Session、UI 和权限都放进去。开始时很顺手，但很快会出现三个问题：

- 替换模型需要修改核心构造函数。
- 测试一个工具时必须启动整套系统。
- 卸载插件时不知道哪些监听器、服务和注册仍然残留。

DSH 用 Cordis Context 把“共享能力”变成服务。插件不是往超级核心里塞字段，而是向 Context 贡献 Service、Typed Event 和 Effect。

## Context 不是全局变量

可以把 Context 理解成一张有生命周期的能力表：

```text
Plugin.setup(ctx)
  -> 注册 service / event listener / command / tool
  -> Loader 发布并运行
Plugin unload
  -> Effect unwind
  -> 注册、监听和资源回收
```

`ctx.sessions`、`ctx.tools`、`ctx.llm` 这些 key 不是普通的全局单例。它们属于某个 Context，并受该 Context 的装载和卸载边界约束。正因为如此，同一个服务定义可以在不同组合中由不同 Provider 实现。

## Plugin、Service、Effect 的关系

- **Plugin**：声明一组安装动作和生命周期。
- **Service**：向 Context 提供可被其他插件注入的能力。
- **Effect**：描述一次可撤销的副作用，例如注册监听器或加入工具表。
- **Event**：让插件在不直接依赖实现的情况下观察或拦截流程。

一个插件的价值不只是“导出一个类”，而是完整表达它进入系统、参与运行、退出系统的方式。

## 为什么可逆性是功能

开发模式下 HMR、Profile 切换、测试隔离和动态扩展都会要求插件退出。如果注册没有对应的撤销，旧工具可能继续出现在 Prompt，旧监听器可能执行两次，旧 Remote 可能还保留着失效方法。

所以 DSH 的生命周期审美是：安装不是永久写入，安装是一项有边界的 Effect。可逆性让系统能试错，也让测试可以验证“卸载后世界恢复原样”。

## 本章结论

“一切皆插件”真正表达的是无特权核心。Core 只是由若干同样遵守生命周期规则的插件组成。下一章要看这些插件如何在启动时被配置成不同的产品。

源码导航：`docs/cordis-primer.md`、`docs/architecture.md`、`packages/boot/app-boot`。


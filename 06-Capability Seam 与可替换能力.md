# Capability Seam 与可替换能力

## 这一章的叙事主线

`seam` 不是“某个接口很抽象”的高级说法。它描述的是一整个可替换能力：谁定义契约，谁提供实现，谁消费它。缺少任何一个角色，都还只是一个零件。

## 先用插座理解

插座标准定义连接方式，发电设备提供电力，电器消费电力。换一个发电设备，不应要求所有电器重写。DSH 的 Capability Seam 也有同样结构：

```text
Service Definition
  -> Service Provider(s)
  -> Consumer(s)
```

## Shell 是 canonical example

在 DSH 中可以这样对应：

- `dsh-shell`：定义 `ShellExecutor` 服务和相关词汇。
- `dsh-bash-local` / `dsh-bash-sandbox`：提供本地或沙箱执行实现。
- `dsh-tool-bash`：把 Shell 能力包装成模型可调用的 Consumer。

模型只知道 Bash Tool 的契约，Tool 不需要知道 Provider 是本机进程还是远程沙箱。更换 Provider 后，Bash、PTY、LSP 等共享同一执行世界的能力可以一起移动。

## Seam 与普通 Extension Point

一个事件监听器通常是 Extension Point；一个 `ctx.shell` Provider 加上它的 Definition 和 Consumers，才是完整 Seam。把单个 `Service` 类直接称为 Seam，会让读者误以为“有接口就可替换”，却看不到谁负责实现选择和消费语义。

## 设计新能力时的三个问题

1. 其他包依赖的稳定词汇是什么？这属于 Definition。
2. 哪些实现可以在不改变消费者的情况下替换？这属于 Provider。
3. 谁把能力变成用户或模型真正能使用的动作？这属于 Consumer。

如果三个问题都答不上来，新功能可能只需要一个局部插件，而不值得制造新的 Seam。

## 本章结论

Seam 的价值不在抽象本身，而在替换发生时消费者无需分叉。它把“实现可以变”变成系统的正常状态，而不是一次大规模重构。

源码导航：`docs/glossary.md` 的 capability-seam、`docs/capability-seams.md`、`packages/shell`。


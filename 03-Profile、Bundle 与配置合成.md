# Profile、Bundle 与配置合成

## 这一章的叙事主线

同一套源码为什么能启动 Web 应用，也能启动没有服务器的 Headless Runner？答案不是复制两份主程序，而是让运行时由配置层组合出来。

## 从两个产品开始

Web 需要浏览器模块、Connection 和 Web Server；Headless 只需要模型、工具、Session 和终端输出。它们共享底层能力，却不应该共享完全相同的插件树。

```text
同一批 package
  + web profile rows     -> Web Runtime
  + headless profile rows -> Headless Runtime
```

## 三个词的分工

- **Profile**：一个具名运行组合，记录要叠加哪些 Bundle 和用户 Patch。
- **Bundle**：可分发的 Cordis 配置层，包含它要挂载的代码和 config rows。
- **Patch**：按 entry id 修改或插入配置行的覆盖层。

启动时，配置层按顺序作用于一个空的 entry list：

```text
bundles in profile order
-> profile cordis.patch.yml
-> home-level patch
-> CLI --patch overlay
-> Loader
```

## Patch 不是字段级深合并

这是一个很容易误解的地方。Patch 找到某个 entry id 后，替换的是这一行的完整 config；如果没有这个 id，才插入新行。它不是把两份任意 JSON 做递归合并。

例如原配置是：

```yaml
- id: llm
  module: dsh-llm-deepseek
  config:
    model: deepseek-chat
    timeout: 30
```

Patch 如果只写了 `model`，并不意味着 `timeout` 自动从旧对象继承。这样的语义故意要求覆盖者明确声明完整行，避免一个隐藏的深合并规则改变插件启动行为。

## 为什么组合优于分叉

如果 Web 和 Headless 各自维护一份代码，底层修复需要同步两条分支。Bundle 把“哪些能力一起出现”从实现代码中抽出来，产品差异成为配置选择。

代价是配置层需要清晰的 entry id、顺序和依赖关系；收益是用户可以用自己的 Patch 改变模型、工具或权限，而不用复制整个产品。

## 配置状态与运行状态不是一回事

配置行存在，只说明系统“计划挂载”某个模块，并不证明它已经成功运行：

```text
配置存在 -> module 找不到 / service 缺失 / Fiber 失败
         -> pending 或 failed
```

因此 `--dump-config` 只能回答“最终配置是什么”，不能完整回答“哪些插件真的激活了”。配置来源诊断和 Loader 运行诊断应当分成两层。

## 本章结论

Profile 是产品形状，Bundle 是可分发的组合单元，Patch 是明确的覆盖动作。它们把变化放在启动边界，避免把产品差异硬编码进运行时。

源码导航：`docs/architecture.md` 的 Profiles and bundles、`packages/boot/app-boot/README.md`、`packages/boot/app-boot/src/profile.ts`。

## 普通环境变量分支 vs DSH 配置组合

普通 Agent 常用 `WEB=true`、`HEADLESS=true`、`USE_SANDBOX=true` 控制主入口。选项增多后，分支会和实现代码纠缠。DSH 把差异放进 Profile、Bundle 和 Patch，使最终插件树能被 dump、覆盖和审查。

优势是产品差异成为配置数据；代价是 entry id、layer 顺序和整行替换语义必须被团队共同维护。它没有消灭复杂度，而是把隐含分支变成可检查的组合。

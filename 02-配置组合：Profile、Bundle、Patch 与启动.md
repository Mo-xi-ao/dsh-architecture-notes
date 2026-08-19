# 配置组合：Profile、Bundle、Patch 与启动

## 这一章的叙事主线

普通 Agent 常用环境变量和 if-else 切换 Web、Headless、本地模型和沙箱。DSH 把这些差异变成配置层：Profile 选择产品形状，Bundle 提供发行单元，Patch 覆盖配置行，Loader 将最终 rows 变成 Plugin Tree。

## 组件词典

| English | 中文 | 角色 | 状态 |
| --- | --- | --- | --- |
| Profile | 运行配置 | 选择 Bundles 和用户 Patch | selected |
| Bundle | 配置发行单元 | 提供 Cordis rows 与代码 | available / applied |
| Patch | 覆盖补丁 | 替换或插入 entry row | pending / applied |
| Entry row | 配置行 | 描述 module、id、config | declared |
| Loader | 插件加载器 | 把 rows 装进 Context | loading / active / failed |
| Plugin Tree | 插件树 | 启动后的实际运行结构 | mounted / disposed |

## 配置合成流程

```text
empty entry list
-> bundles in profile order
-> profile cordis.patch.yml
-> home-level patch
-> CLI --patch
-> final rows
-> Loader
-> Plugin Tree
```

Patch 依据 entry id 定位并替换整行配置，不是任意 JSON 的字段级深合并。这样覆盖者必须明确声明最终 module、依赖和 config，避免隐式继承危险字段。

## 具体例子

```text
Base Bundle：llm = deepseek-chat，tools = bash/fs/terminal
Web Bundle：加入 Client、Connection、WebServer
Profile Patch：llm = deepseek-reasoner
Home Patch：tools = bash/fs
```

最终 Loader 只处理合成后的 rows，但诊断工具应能追溯每行的来源。配置存在不代表插件已成功激活，仍可能出现 module missing、service missing、pending 或 failed。

## 普通 Agent 与 DSH

普通 Agent 的环境变量分支更轻，但配置来源和优先级会隐含在代码中。DSH 的优势是最终组合可 dump、可审查、可复用、可由用户 Patch；代价是 entry id、层顺序和整行替换语义需要维护。

源码导航：`docs/architecture.md`、`packages/boot/app-boot/README.md`、`packages/boot/app-boot/src/profile.ts`。

## 下一章要解决的问题

配置已经形成了 Plugin Tree，但插件树只是静态组合。下一步要看用户输入如何进入 Agent，以及一次 Turn 为什么会包含多个 Step 和工具调用。

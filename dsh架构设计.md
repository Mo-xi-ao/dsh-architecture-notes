# DeepSeek Harness 架构设计

## 这套笔记的阅读主线

DSH 不是在 LLM 外面简单加几个工具。它更像一个为 Agent 准备的运行时：配置决定哪些能力被装载，Cordis 把能力组织进 Context，Agent Loop 推进一次次模型请求，Session Log 保存可恢复的事实，Host 与 Client 再把同一个运行世界投影到不同边界。

```text
Profile / Bundle / Patch
-> Cordis Plugin Tree
-> Agent + Session + LLM + Tools
-> Live 控制流 + Durable Session Log
-> Host Runtime / Client Projection
```

## 章节目录

- [[01-什么是 DeepSeek Harness]]
- [[02-Cordis 与一切皆插件]]
- [[03-Profile、Bundle 与配置合成]]
- [[04-Agent、Turn 与 Step 如何运转]]
- [[05-Session Log：Live 与 Durable]]
- [[10-DSH 中的状态与无状态设计]]
- [[06-Capability Seam 与可替换能力]]
- [[07-工具、权限与执行世界]]
- [[08-Host、Client 与 Remote 边界]]
- [[09-DSH 的架构设计美学]]

## 阅读方式

如果刚接触 DSH，按顺序阅读。遇到某个术语时，再回到仓库的 [Glossary](../deepseek-harness-master/deepseek-harness-master/docs/glossary.zh.md) 和每章的源码导航。每章的目标不是记住所有包名，而是回答一个架构问题：它解决了什么耦合，代价是什么，下一层如何接住它。

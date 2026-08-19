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

这套笔记按一条运行时因果链组织，而不是按包名排列：先理解系统是什么，再理解它如何启动、如何运行、如何保存状态、如何接入能力，最后才讨论跨 Host/Client 边界和架构取舍。

- [[01-运行时总览：Harness、Cordis 与插件树]]
- [[02-配置组合：Profile、Bundle、Patch 与启动]]
- [[03-Agent Loop：Agent、Turn、Step 与工具调用]]
- [[04-状态主线：Session、Live、Durable 与状态注册]]
- [[05-能力主线：Seam、Tool、Scope、权限与执行世界]]
- [[06-边界主线：Host、Client、Remote 与 UI 投影]]
- [[07-架构判断：普通 Agent 与 DSH 的差异]]

旧版按主题拆分的章节仍保留在目录中，作为详细参考材料；上面七章是推荐主线。

## 阅读方式

如果刚接触 DSH，按顺序阅读。遇到某个术语时，再回到仓库的 [Glossary](../deepseek-harness-master/deepseek-harness-master/docs/glossary.zh.md) 和每章的源码导航。每章的目标不是记住所有包名，而是回答一个架构问题：它解决了什么耦合，代价是什么，下一层如何接住它。

# DeepSeek Harness 架构设计笔记

> 笔记原创：墨兮奥

这是一套面向学习者的 DeepSeek Harness（`dsh`）中文架构笔记，写作上参考 `mewAgent` 的章节化叙事，内容依据 DSH 仓库当前公开的架构文档、源码和术语整理。

## 内容范围

笔记围绕以下主线展开：

```text
Profile / Bundle / Patch
-> Cordis Plugin Tree
-> Agent + Session + LLM + Tools
-> Live 控制流 + Durable Session Log
-> Host Runtime / Client Projection
```

从 [总目录](./dsh架构设计.md) 开始阅读。

## 推荐阅读顺序

```text
01 运行时总览
-> 02 配置组合
-> 03 Agent Loop
-> 04 状态主线
-> 05 能力主线
-> 06 Host / Client 边界
-> 07 架构判断
```

这不是按目录名称排序，而是按一次 DSH 运行从“被组合”到“被恢复”的因果顺序排序。

## 参考输入

`参考输入/mewAgent-全部图片文字提取.md` 保存了 `mewAgent` 目录中 50 张图片的 OCR 文字，作为本套 DSH 笔记的叙事和讲解风格参考。它不是 DSH 源码，也不替代上游项目文档。

## 来源与归属

- DeepSeek Harness 源码、官方文档和项目名称归 [DeepSeek AI](https://github.com/deepseek-ai/deepseek-harness) 及其相应贡献者所有。
- DeepSeek Harness 上游项目采用 MIT License；如一并分发上游源码，必须保留上游 `LICENSE` 和 `THIRD_PARTY_NOTICES.md`。
- 本目录中的中文架构笔记、章节组织、解释文字和总结由“墨兮奥”原创整理。
- 本笔记是非官方学习资料，不代表 DeepSeek AI，也不是 DeepSeek Harness 官方文档或官方支持渠道。

## 发布建议

推荐将本目录作为独立笔记仓库发布，或作为你个人 DSH 学习仓库中的 `docs/dsh-architecture` 子目录。除非确实需要维护源码镜像，否则不要把上游完整源码、压缩包和本地工作草稿一起复制进个人仓库。

本目录目前没有单独声明开放内容许可证。转载、改编或商用前，请联系作者“墨兮奥”取得许可；上游源码仍受其原有许可证约束。

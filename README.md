# 知识卡片 · Knowledge Cards of Fei

个人文献精读与术语沉淀仓库：读过的论文落成长文笔记，遇到的术语整理成卡片。
内容以中文书写，术语与论文原名保留英文。

## 目录结构

| 目录 | 内容 | 是否入库 |
| --- | --- | --- |
| `notes/` | 论文精读笔记（`__paper`）与概念长文（`__concept`） | ✅ |
| `Terms/` | 术语卡片库，按 Concept / Method / Tool 三类归档 | ✅ |
| `Papers/` | 论文 PDF 原文与少量附件，体积大，本地留存 | ❌ 默认忽略 |


## 笔记约定

文件名沿用时间戳前缀，便于按时间排序：

```
notes/YYYYMMDDTHHMMSS--<slug>__<type>.md
# type: paper | concept
```

笔记按 **ljg-paper** skill 的流程产出：先承认自己缺哪些概念，再把这些概念接成能解释现象、能被证据约束的模型，而不是把论文压缩成一份摘要（见文末[致谢](#致谢)）。

每篇笔记的结构：

- **头部元信息**：一句话主标题 + 副标题 / 日期 / 来源 / 作者 / 发表场所
- **速读** —— 问题、洞见、带走三句话，串成一条因果链
- **它到底在解决什么** —— 原锚点：旧解释在哪里失灵
- **它真正看见了什么** —— 承重概念按依赖顺序接成能解释现象的模型
- **我能带走什么** —— 一个以后能直接拿去用的判断

## 术语卡片约定

```
Terms/{Concept,Method,Tool}/<Term>.md
```

- **Concept**：抽象概念、范式、理论构造、任务定义，不含具体实现
- **Method**：可实现的算法、学习算法、模型架构
- **Tool**：软件库、框架、SDK、语言或平台

分类边界与判例见 [Terms/README.md](Terms/README.md)。

卡片内容：JSON 头（`term` / `full_form` / `folder` / `definition` / `tags` / `storage_path`）+ 📖 详细定义 / 🔬 前沿研究 / 🔗 相关概念 / ⚖️ 优缺点。

## 术语卡片生成 Agent

`Terms/` 下的卡片由同一份 agent 设定生成，随仓库版本化，换台机器也能复现同样的产出：

- [`.joycode/modes/rules-ep69s-oo/agentDefinition.md`](.joycode/modes/rules-ep69s-oo/agentDefinition.md) —— 术语学习与归档助手：输出 JSON 卡片 + 精简 Markdown，并直接落盘
- [`Terms/.joycode/modes/rules-ep69s-oo/agentDefinition.md`](Terms/.joycode/modes/rules-ep69s-oo/agentDefinition.md) —— 在 `Terms/` 工作区下使用的同内容副本
- [`Terms/.joycode/memory/feedback_term_teacher.md`](Terms/.joycode/memory/feedback_term_teacher.md) —— 反馈约束：卡片写进 `Terms/<分类>/`，而不是只在回复里输出

其余 `.joycode` 内容（memory、skills、其它 mode）属于本地运行态，由 `.gitignore` 忽略。

## 致谢

`notes/` 的读法与写法来自 **ljg-paper** —— 一个面向非专业读者的 paper reading skill。它不满足于「知道论文说了什么」，而是要求先找出自己缺的承重概念，再让这些概念按依赖关系接成一个能解释原问题、能区分邻近情况、能被证据约束的模型；本仓库笔记的命名、四段结构与「不把承重概念压缩掉」的取舍，都沿用它。

感谢 ljg 公开分享这份 skill。


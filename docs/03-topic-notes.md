# 按话题写笔记

只靠压缩会丢掉细节。话题结束时，把「问题 → 结论」写成仓库里的文件并提交。这些文件以后可以被搜索、被读取。它们不会在每次新对话里自动整份进入提示，产品也没有保证 Agent 每次都会引用它们。

压缩只影响这条聊天里模型还拿着的窗口：较早内容变成摘要。已经写进 Git 的笔记留在仓库里，不受这次压缩影响。

## 写进 Git 之后，下次会话能怎样用到

没被 `.gitignore` 或 `.cursorignore` 排除的 Markdown，属于这个工作区的代码库。以后的对话可以用 Agent 自己的搜索、读文件，或在输入里 `@` 文件、文件夹找到它。项目规则和 `AGENTS.md` 跟仓库走。提交后换机器 clone 再打开，同一套规则仍在。

这些笔记不会整份自动进入每一次未来对话的 prompt。上下文开头放的是**被应用的规则正文**。普通文件要等 Agent 决定去搜、去读，或你用 `@` 附上。

文档没有写「每一轮都自动搜完整仓库」。写明的是按需：

- 不知道该看哪些文件时可以不 `@`，Agent 会用自己的搜索找相关文件。
- 你提到具体符号时，它会自动用 grep。
- 它判断任务需要大范围搜索时，会自动用 Explore 子代理。你也可以直接要求它用子代理搜。
- 过去的聊天：需要时 Agent 可以自己查对话，你也可以 `@Chats`。这和仓库文件是两套东西。

现行检索说明还写明：Instant Grep 在本机构建并查询索引，不用配置，支持正则。Cursor 不会为搜索上传文件路径或代码，也不会为搜索保存代码库的 embeddings。Agent 打开某个匹配后，该文件内容仍可能进入模型请求。文档没有写「所有 `.md` 都会被语义索引」。能确定的是：没被 ignore 的项目文件在 Agent 可搜索、可读取的范围内。

因此文件名和标题里要带上以后会搜的词（模块名、接口名、错误名）。一篇只叫「本周总结」的笔记，比标题里写着具体对象的笔记更难被精确匹配命中。

## `@` 会把什么带进这次对话

| 写法 | 作用 |
| --- | --- |
| `@` 文件或文件夹，例如 `@auth.ts`、`@src/components/` | 把这些文件或文件夹放进这次对话。文件夹后再打 `/` 可以往下走 |
| `@Chats` | 引用以前某次对话的上下文 |
| `@Terminals` | 终端输出 |
| Git diff | 未提交改动，或相对 main 的分支 diff |
| `@Browser` | 内置浏览器的上下文 |

可以多次 `@`，每一项都加进对话。

当前的上下文说明和 prompting 文档没有 `@Codebase` 这个符号。`@` 菜单里的 Docs 和 “Add New Custom Docs” 已经去掉，因为它们不再把文档加进 Agent 上下文。旧对话里的 Docs 芯片还会显示，但不再起作用。

## 用规则把笔记目录接上

模型本身不跨轮记东西。规则被应用时，正文放在模型上下文的开头。不是所有规则文件都永远在 prompt 里。

| 类型 | 何时进入上下文 |
| --- | --- |
| Always Apply（`alwaysApply: true`） | 每次对话都包含。globs 和 description 被忽略 |
| Apply to Specific Files | 上下文里有匹配 glob 的文件时自动附上 |
| Apply Intelligently | 写了 description、没有 globs 时，Agent 根据 description 决定要不要把这条规则拉进来 |
| Apply Manually | 只有你在聊天里 `@` 这条规则时才包含 |

`AGENTS.md` 是项目根或子目录里的普通 Markdown，Cursor 会自动读取。子目录里的 `AGENTS.md` 在处理该目录及其子目录时与上层合并，更具体的优先。Agent 在某子目录里编辑或新建文件时，会收到该目录的嵌套 `AGENTS.md` 和 always-apply 规则，即使它还没读过那里的文件。每条在一次对话里只加一次。

一条 Always Apply 的规则如果写了「涉及既有决策时，先看 `docs/decisions`」，这句话会在上下文开头。规则里的 `@某文件` **不会**把文件内容内联进 prompt。Agent 用工具在需要时去读。必须每次都在上下文里的内容，要直接写在规则正文里。文档没有保证模型每次都会照做并搜到。

帮助文档说项目规则、用户规则、团队规则在 Agent / Ask / Plan / Debug 都生效，不因模式而关掉。这是说跨模式生效，不是说每一种条件规则都会无视条件、整份塞进每次 prompt。

开始一段相关工作时，再 `@` 那个笔记文件夹，把这一次要用的文件直接放进对话。

## 忽略规则

`.cursorignore` 使用 gitignore 语法。被忽略的文件：Agent 和 Tab 不能访问，`@` 也不能引用。终端和 MCP 不在这套限制里，仍可能读到。`.gitignore` 里的文件同样排除出 AI 上下文。`.cursorignore` 是在这之外再排除。

默认还会忽略 `.env*`、`.git/`、锁文件、一批二进制和办公文档（含 `*.pdf`、`*.csv`、常见图片等）。**默认列表里没有 `*.md`。** 决策笔记放在仓库里、且没有写进 `.gitignore` / `.cursorignore`，就不会因为「它是 Markdown」被默认排除。若笔记目录被 gitignore，Agent 的搜索和 `@` 都看不到。

## 笔记里留什么

每条话题单独一个文件。正文保留压缩会丢掉的部分：

- 问题是什么，当时的约束
- 结论，以及否决过的方案和原因
- 关键路径、配置、接口名、数字
- 还没做完的后续

目录旁放一份短索引：日期、话题、结论一行。Always Apply 规则或根目录 `AGENTS.md` 里写明笔记目录，以及「涉及既有决策时先读这个目录」。需要出处时，直接说「引用你读过的笔记」。

读过的官方文档里没有「Agent 一定会引用它读过的仓库文件」这种保证。相近但不一样的一点是 Agent Store 的 file citation：点开会打开对应文件。那是 Agent Store，不是仓库笔记的引用保证。

下面是一份可直接改路径后使用的 Always Apply 规则正文。它只指出目录和何时去读，不把笔记全文抄进规则。规则建议控制在 500 行以内。

```md
---
description: 涉及既有决策时先读话题笔记
alwaysApply: true
---

话题笔记在 docs/decisions/，索引是 docs/decisions/README.md。
问题涉及已有结论、否决方案、接口、配置或路径时，先读索引，再打开对应笔记。
回答时引用读过的笔记路径。
```

单篇笔记可以按这个结构写：

```md
# 话题标题

- 日期：
- 状态：结论已定 / 仍开放

## 问题

当时要决定什么，约束是什么。

## 结论

采用的做法。

## 否决的方案

每个方案一行原因。

## 关键事实

路径、配置、接口名、数字。

## 后续

还没做完的事。
```

## 依据

- [Search](https://cursor.com/docs/agent/tools/search)
- [Prompting](https://cursor.com/docs/agent/prompting)
- [@ mentions](https://cursor.com/help/customization/context)
- [Agent mode](https://cursor.com/help/ai-features/agent)
- [Subagents](https://cursor.com/docs/subagents)
- [Rules](https://cursor.com/docs/rules)
- [规则存放位置](https://cursor.com/help/customization/rules)
- [Ignore files](https://cursor.com/help/customization/ignore-files)
- [ignore-file 参考](https://cursor.com/docs/reference/ignore-file)
- [Agent troubleshooting](https://cursor.com/help/troubleshooting/agent-issues)
- [IDE release notes](https://cursor.com/docs/release-notes/ide)（检索相关的 3.19、3.21、3.27、3.28）
- [Agent 概览](https://cursor.com/docs/agent/overview)
- [对话搜索](https://cursor.com/help/ai-features/conversation-search)

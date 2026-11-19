# 上下文存在哪里

一年的开发会话里，**完整对话记录**和**模型单次能看见的内容**是两回事。文档没有写「一个聊天可以原样记住一整年」。模型上下文是固定窗口，接近上限时较早内容会被压成摘要。

以一个持续约一年的实施项目为例：细节若只留在这条聊天里，后面的推理看到的是摘要。要留下来的约束、否决方案和路径，需要写进仓库。写进去之后怎样被下次读到，见 [按话题写笔记](03-topic-notes.md)。

## 本地对话记录

本地 Agent 的对话存在本机的本地数据库里，对话多了可以到数 GB。官方没有写出这个数据库的具体文件路径。

跨历史对话的搜索索引也在本地。Agents 窗口里用 `Cmd+K` / `Ctrl+K` 搜索过去的记录。当前对话里用 `Cmd+F` / `Ctrl+F`。文档写这个本地索引可以覆盖数千段对话。

清理是手动的：命令面板里的 `Delete Old Chats…` 按「上次更新」超过 30、60、90 或 180 天来删，再用 `GC Agent KV Blobs` 把数据库文件缩小。这是主动清理。文档没有写聊天会自动过期，也没有写本地对话会整份同步到 Cursor 账号。

## 检查点

检查点是本机上的代码快照，和 Git 分开。Agent 在较大改动前自动创建，时间线上可以预览并恢复文件。恢复只回滚文件，不会删对话消息。文档没有写检查点的大小上限或保留期限，并写明长期版本管理用 Git。

## 模型窗口

每个聊天共用一个固定上下文窗口。文件、工具结果和消息把窗口填满后，Cursor 会把较早部分压成摘要，给新对话留位置。输入框旁的上下文环可以拆开看，其中单独有「已摘要的对话」和「当前对话」。

因此：长对话的记录仍可打开、在记录里搜索。模型下一次推理看到的是窗口里还留着的近期内容，加上较早内容的摘要。文档没有写摘要之后，磁盘上的原始消息是否逐字保留。

Max Mode 只存在于旧的按请求计费方案。当前按用量计费的方案没有 Max Mode。文档没有把 Max Mode 说成可以装下一年原始聊天。

## 规则、说明文件、记忆

规则文档写明：大模型在两次补全之间不保留记忆。持久上下文靠规则，应用时放在模型上下文开头。

| 内容 | 存在哪里 |
| --- | --- |
| 项目规则 | 仓库 `.cursor/rules/*.mdc`，随 Git |
| `AGENTS.md`（以及同样方式读取的 `CLAUDE.md`） | 项目根目录或子目录，在仓库里 |
| 设置里的 User Rules | Cursor 账号，登录后跨设备同步 |
| `~/.cursor/rules`（Windows：`%USERPROFILE%\.cursor\rules`） | 只在本机，不同步 |
| Team Rules | Cursor 服务器，从团队后台下发 |

普通本地聊天没有单独的 Memories 产品说明。Memories 只出现在 Automations：同一次自动化的多次运行之间，默认记在名为 `MEMORIES.md` 的条目里，位于 Agent 工作区文件系统之外，可在自动化的工具配置里查看和编辑。

Cloud Agent 最佳实践把 Agent 描述成聪明但上下文很少的开发者，并建议用仓库里的 skills、`AGENTS.md` 和规则把约定交给它。规则建议控制在 500 行以内，需要的文件用路径引用，不要把整份资料复制进规则。规则正文里的 `@某文件` 不会把该文件内容内联进提示；Agent 在需要时用工具去读。必须每次都在上下文里的句子，要直接写在规则正文里。

## Cloud Agent 与 Project

Cloud Agent 在 Cursor 云上的隔离虚拟机里跑。状态分成几块：

| 状态 | 存在哪里 | 文档中的保留 |
| --- | --- | --- |
| 当次运行的工作区（检出的仓库、构建产物、工具上下文） | 这台虚拟机 | 运行空闲后按生命周期回收。再发后续消息会刷新这个计时 |
| 虚拟机磁盘快照（含克隆下来的代码） | 虚拟机之外的加密快照层 | 连续 30 天没有启动或恢复就删除。不是一台磁盘能闲置一年还在的机器 |
| 对话状态（提示、回复、工具调用、diff、演示产物） | Cursor 后端，按 Agent 单独加密 | 默认可一直保留，以便以后打开并继续。可用 Delete Agent API 删掉。Enterprise 可把保留设为无限期或 90 天（自定义窗口为早期功能） |
| 密钥 | Cursor 后端 | 加密保存，直到你删除 |

隐私概览还写：Cloud Agents 是唯一需要 Cursor 存储代码的功能。仓库加密副本在运行期间临时保存，Agent 完成后删除。上面这张表是安全文档里的具体保留规则。

Projects 跑在 Cloud Agents 上。文档写它可以在数月的工作里维持上下文：协调者把研究和约定放进一组会同步到该 Project 各 Agent 的文件。项目共享存储在 `/cursor/stores/self`，个人存储在 `/cursor/stores/user`。文档写它适合活过单次聊天的工作（一个功能、一次迁移），没有写「一个 Project 保证记满一年」。

## 对「做一年」文档实际说明了什么

文档没有讨论「一个会话连续做一年」，也没有给出一年期保留承诺。能直接对应的事实是：

- 模型窗口装不下一年原始聊天。接近满时，较早内容变成摘要。
- 本地记录留在本机数据库，可搜索。检查点只用于回滚 Agent 改过的文件。
- 跨聊天还能用的，是仓库里的规则、`AGENTS.md`、skills，以及 `@` 引用文件或 `@Chats` 引用另一段对话。
- 更大、更长的一块工作，文档指向 Cloud Agent 上的 Project：上下文在 Project 的共享文件里积累。
- Cloud Agent 的对话默认可长期保留并继续。虚拟机磁盘快照在 30 天不活动后删除。

## 隐私

使用 AI 时，提示和代码上下文会发给模型提供商。Privacy Mode 打开时，Cursor 和模型提供商都不会拿这些代码做训练。多数模型还有零数据保留协议，提供商不存储输入输出。少数模型（文档点名 Claude Fable 系列）要求提供商保留数据，默认关闭，需管理员批准。用自己的 API key 时，按该提供商的政策处理。

本地聊天的记录在本机数据库。文档没有写这些本地全文会再存一份到 Cursor 服务器。你主动分享时，才会生成一份只读副本（团队或公开链接）。其中包含完整对话、代码片段和工具结果。Teams / Enterprise 可用，且不适用于 “No Storage” Privacy Mode。Enterprise 还可给已分享记录设 30 到 365 天保留，默认关闭。

删除 Cursor 账号会删除该账号及其关联数据，包括账号上的聊天记录和设置，30 天内移除。本地项目文件不受影响。文档没有把「账号上的聊天记录」等同于本机那份数据库。

## 依据

- [Agent 概览](https://cursor.com/docs/agent/overview)
- [对话搜索](https://cursor.com/help/ai-features/conversation-search)
- [本机聊天数据库与清理](https://cursor.com/help/troubleshooting/install-issues)
- [上下文窗口与摘要](https://cursor.com/docs/agent/prompting)
- [@ 引用](https://cursor.com/help/customization/context)
- [Max Mode](https://cursor.com/help/ai-features/max-mode)
- [规则](https://cursor.com/docs/rules)
- [规则存放位置](https://cursor.com/help/customization/rules)
- [Projects](https://cursor.com/docs/agent/projects)
- [Cloud Agents](https://cursor.com/docs/cloud-agent)
- [Cloud Agent 安全与保留](https://cursor.com/docs/cloud-agent/security)
- [快照与对话保留细则](https://cursor.com/docs/cloud-agent/security-network)
- [Cloud Agent 最佳实践](https://cursor.com/docs/cloud-agent/best-practices)
- [Automations Memories](https://cursor.com/docs/cloud-agent/automations)
- [隐私与数据治理](https://cursor.com/docs/enterprise/privacy-and-data-governance)
- [隐私帮助](https://cursor.com/help/security-and-privacy/privacy)
- [分享记录](https://cursor.com/help/ai-features/shared-transcripts)
- [保留与删除汇总](https://cursor.com/docs/enterprise/security-hardening)
- [删除账号](https://cursor.com/help/account-and-billing/delete-account)

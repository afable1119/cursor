# 仓库与会话

**仓库（repo）是代码所在的 Git 仓库。会话是你和 Agent 的一条对话。** 一个仓库上可以有很多条会话。官方文档没有把 Session 定义成和仓库并列的一级对象。编辑器里日常说法是对话（chat、conversation、thread）。CLI 和 Hooks 会直接把它叫 chat session 或 session。

## 仓库

本地用法是打开一个项目文件夹：`File > Open Folder`。这个打开的文件夹叫 **workspace（工作区）**。它如果是 Git 仓库，Agent 就在这里读代码、改文件、跑命令。

和仓库相关的几层：

- **一个窗口里的工作区有自己的 Agent 和 Git 上下文。** 另一个窗口里的 Agent 不会改这个工作区里的文件。
- **多根工作区可以有多个根目录。** Hooks 里的 `workspace_roots` 通常只有一个路径，多根工作区会有多个。
- **Agents Window 可以同时管理多个项目、多个仓库上的 Agent**，包括本地、云端和远程 SSH。
- **Worktree** 是同一个仓库上的隔离 Git checkout。多个 Agent 可以各改各的文件，主工作区保持不动。
- **Cloud Agent 的环境会克隆仓库。** 可以是一个仓库，也可以是一组仓库（frontend、backend 等分开的仓库）。

Agents Window 左侧还有一个产品功能叫 **Project**。创建时在 **Workspace** 里选择这个 Project 使用的仓库。Project 用来长期协调很多 Agent，本身不是 Git 仓库。需要另一个仓库时，会先申请访问，再在那个仓库上启动 Agent。

## 会话

一条会话就是你和 Agent 来回的消息。

- Agent 文档把这一次工作叫 **Agent session**。检查点记下这次会话里的代码快照，存在本地，和 Git 分开。恢复检查点只回滚文件，不删除对话消息。长期版本管理用 Git。
- **CLI 把它叫 chat session。** `/clear` 新开一条，`/rename` 改当前这条的名字，`--resume [chatId]` 恢复某一条，`--continue` 继续上一条，`/fork` 分叉成新的 session。`--resume` 的参数也叫 thread id。
- **Hooks 里，`session_id` 就是 `conversation_id`。** `sessionStart` 在新建一条 composer 对话时触发。`sessionEnd` 在这条对话结束时触发（完成、中止、关窗口或用户关闭）。这是编辑器里这条对话的生命周期。
- **Side chat** 是挂在父对话上的子对话，留在原来的父对话和工作区上。
- Agents Window 里用 `Cmd/Ctrl+K` 搜索过去的 agent transcripts，索引建在本地。当前对话里用 `Cmd/Ctrl+F`。

Agent SDK 里的 session 是另一套：一条 session 保存一段对话、它的 workspace，以及消息和工具调用记录。那是给自建 Agent 用的，不是编辑器侧边栏里的那条聊天。

## 和仓库的关系

一个仓库上可以有很多条对话、很多个 Agent。Worktree 就是为了让它们同时改同一个仓库时不互相覆盖。

本地这条对话跟当时打开的工作区绑在一起：每次 Hook 都会带上 `workspace_roots`；每个打开的工作区有自己的 Agent 和 Git 上下文；side chat 也留在原来的父对话和工作区上。Agents Window 把这些对话放在一起，可以跨仓库查看和搜索。文档没有写「换一个项目文件夹，旧对话会被删掉」。

搬到云端时，**Move to Cloud 会带走对话历史和上下文**。云端从远程仓库的干净 Git 状态开始，不带本地未提交的修改。

文档没有写 Agents 侧边栏里并排两个叫 Repo 和 Session 的标签。能对上的界面是：

- 按位置搜项目。位置包括 This Computer、Cloud、远程机器。
- 创建 Project 时，在 Workspace 下面选仓库。
- **Cloud Agent 页面顶部是仓库名。** 悬停可以看到这次运行用的环境。
- CLI 用 chat session、thread、conversation id 指同一条聊天。

## Cloud Agent：仓库、对话、run

一次 Cloud Agent 是一个持久对象，地址类似 `cursor.com/agents/bc-...`。创建时带上一个或多个仓库；也可以不带仓库，从零开始，这时会在后台建一个 Origin 草稿仓库。

- 虚拟机克隆环境里的仓库，在单独分支上改。默认是新的 `cursor/...` 分支，再推回仓库，并可以开 PR。
- **对话和工作区状态跟这次 Agent 走，不跟某一次 Git 提交走。** 后面的追问会新开一次 **run**，并沿用这条 Agent 已有的对话和工作区。Git 分支和 PR 记在这条 Agent 上，同一次 Agent 的每个 run 看到的是同一份 git 快照。
- 把链接发给队友，对方能看对话、代码改动和产物；对方自己也要有这个仓库的权限。

本地 Agent 直接改你打开的工作区。Cloud Agent 在隔离虚拟机里改克隆出来的仓库，合上电脑也会继续跑。

托管 Cloud Agent 没有编辑器那种 `sessionEnd`：文档写明它没有 editor-lifetime session boundary。`sessionEnd` 绑的是 IDE 里的对话，不是云端这条聊天。自托管机器上，`sessionStart` / `sessionEnd` 则是在一次会话占用 worker 和释放占用时触发。

## 依据

- [Git](https://cursor.com/help/integrations/git)
- [Hooks](https://cursor.com/docs/hooks)
- [Agents Window](https://cursor.com/docs/agent/agents-window)
- [Worktrees](https://cursor.com/docs/configuration/worktrees)
- [Cloud Agents](https://cursor.com/docs/cloud-agent)
- [环境配置](https://cursor.com/docs/cloud-agent/setup)
- [Projects](https://cursor.com/docs/agent/projects)
- [Access Other Repositories](https://cursor.com/docs/cloud-agent/repo-access)
- [Agent 概览](https://cursor.com/docs/agent/overview)
- [CLI 使用](https://cursor.com/docs/cli/using)
- [斜杠命令](https://cursor.com/docs/cli/reference/slash-commands)
- [参数](https://cursor.com/docs/cli/reference/parameters)
- [Side chats](https://cursor.com/help/ai-features/side-chats)
- [Conversation search](https://cursor.com/help/ai-features/conversation-search)
- [Cloud Agents 帮助](https://cursor.com/help/ai-features/cloud-agents)
- [Cloud Agents API](https://cursor.com/docs/cloud-agent/api/endpoints)
- [Sessions（Agent SDK）](https://cursor.com/docs/agent-sdk/reference/sessions)

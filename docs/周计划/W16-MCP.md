# W16 周计划：MCP

- **所属阶段**：阶段五 AI 应用开发（W12-W18）
- **起止日期**：
- **计划用时**：工作日 5 × 约 1.5 小时 + 周末约 3 小时，共约 10–11 小时
- **硬件要求**：无
- **主计划**：[AI 学习计划（23 周）](../AI学习计划-23周.md)
- **上一周**：[W15](W15-Agent.md)
- **笔记位置**：`notes/W16/`，每天两个文件：学习笔记用 [学习笔记模板](../../notes/学习笔记模板.md)，自我总结用 [自我总结模板](../../notes/自我总结模板.md)

## 1. 本周目标

学完这一周，你应该能做到：

1. 讲清 MCP 的定位，以及 Host / Client / Server 与 Tools / Resources / Prompts。
2. 理解 JSON-RPC、stdio 与 Streamable HTTP，能画出一次工具调用时序。
3. 用 Python MCP SDK 写出可被发现的 Server（列文件、读文件）。
4. 把 W14 的代码检索封装成 MCP 工具。
5. 在 Cursor 里实际调用自己的 Server，并处理好错误与权限。

## 2. 本周产出

| 产出 | 位置 | 对应日 |
|---|---|---|
| MCP 架构图 | `notes/W16/architecture.md` | 周一 |
| 工具调用时序图 | `notes/W16/sequence.md` | 周二 |
| 文件工具 MCP Server | `labs/w16-mcp/file_server/` | 周三 |
| 代码检索 MCP Server | `labs/w16-mcp/code_search/` | 周四 |
| Cursor 接入记录 | `labs/w16-mcp/cursor-setup.md` | 周五 |
| 完善后的项目与 README | `labs/w16-mcp/` | 周末 |

## 3. 开始前的准备

- [ ] 新建目录 `notes/W16/` 和 `labs/w16-mcp/`。
- [ ] 阅读入口：[MCP 官方文档](https://modelcontextprotocol.io/)。
- [ ] 准备 Python 环境；安装官方文档推荐的 MCP Python SDK。
- [ ] 确认 W14 代码检索可以命令行调用（本周做封装）。
- [ ] Cursor 可安装/配置 MCP Server（周五用）。

---

## 4. 每日安排

### 周一：MCP 定位与核心概念

**学习目标**

1. 理解 MCP 要解决的问题：AI 应用与外部工具/数据的标准连接方式。
2. 分清 Host、Client、Server。
3. 分清 Tools、Resources、Prompts。

**时间安排**：读文档 50 分钟，画图 25 分钟，自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [MCP 官方文档首页与介绍](https://modelcontextprotocol.io/) | 英文 | 必读 | 是什么、为什么 |
| [Architecture](https://modelcontextprotocol.io/docs/concepts/architecture) | 英文 | 必读 | Host/Client/Server |
| Tools / Resources / Prompts 概念页 | 英文 | 必读 | 三类原语 |

**动手任务**

1. 画架构图：Cursor（Host）里的 Client 如何连到你的 Server。
2. 表格对比 Tools vs Resources vs Prompts：谁主动、有无副作用、典型例子。
3. 对照 W15：你手写的工具注册表，和 MCP Tool 抽象如何对应。

**验收标准**

- [ ] 架构图能讲 5 分钟。
- [ ] 三类原语能各举一例。

**自测题**

1. MCP Server 是模型本身吗？
2. 为什么需要标准协议，而不是每个 IDE 私有插件协议？
3. Resource 和 Tool 在“读文件”场景下如何取舍？

---

### 周二：JSON-RPC、传输与能力发现

**学习目标**

1. 理解 MCP 消息如何基于 JSON-RPC 表达。
2. 知道 stdio 与 Streamable HTTP 两种传输的适用场景。
3. 梳理 initialize → 能力发现 → tools/call 的完整时序。

**时间安排**：读文档 40 分钟，画时序图 30 分钟，自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [MCP Transport 文档](https://modelcontextprotocol.io/docs/concepts/transports) | 英文 | 必读 | stdio / HTTP |
| Lifecycle / Messages 相关文档 | 英文 | 必读 | 初始化与能力协商 |
| [JSON-RPC 2.0 规范](https://www.jsonrpc.org/specification) | 英文 | 选读 | id / method / params / error |

**动手任务**

1. 画时序图：用户在 Cursor 发问 → Client 列工具 → 模型选工具 → `tools/call` → Server 返回 → 模型继续。
2. 笔记：本地 IDE 插件为何常偏爱 stdio；远程/多客户端何时考虑 HTTP。
3. 记录能力发现里你最关心的字段（tools 列表、参数 schema）。

**验收标准**

- [ ] 时序图步骤完整。
- [ ] 能解释“能力发现”发生在什么时机。

**自测题**

1. JSON-RPC 的 `id` 有什么用？
2. stdio Server 的生命周期通常由谁拉起？
3. 工具 schema 变更后，Client 侧常见要做什么？

---

### 周三：第一个 Python MCP Server

**学习目标**

1. 用官方 SDK 写出最小 Server。
2. 实现 `list_files`、`read_file`，并能被 MCP 客户端发现。

**时间安排**：读快速开始 20 分钟，编码 55 分钟，联调 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [MCP 快速开始 / Python SDK 文档](https://modelcontextprotocol.io/) | 英文 | 必读 | 从官网进入 SDK 与示例 |
| 本周架构笔记 | — | 必读 | 对照实现 |

**动手任务**

1. 创建 `file_server`：工作根目录可配置。
2. 工具：
   - `list_files(path?)`
   - `read_file(path, max_bytes?)`
3. 路径沙箱：禁止 `..` 逃逸。
4. 用文档推荐的检查方式 / Inspector / 最小 client 确认工具可被列出并调用。

**验收标准**

- [ ] 工具可被发现并返回正确内容。
- [ ] 非法路径返回明确错误而不是崩溃。

**自测题**

1. 工具的输入 schema 为什么要写清楚类型与必填？
2. 错误信息怎样写才对模型后续修复有帮助？
3. 大文件读取为什么必须截断？

---

### 周四：封装 W14 代码检索为 MCP 工具

**学习目标**

1. 把代码搜索能力暴露为稳定 MCP 工具。
2. 设计对 Agent 友好的返回格式（短、结构化、带来源）。

**时间安排**：接口设计 20 分钟，封装 55 分钟，测试 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| W14 `labs/w14-code-rag` | — | 必读 | 检索入口 |
| MCP Tools 文档 | 英文 | 必读 | 返回内容类型 |

**动手任务**

1. 工具建议：
   - `search_code(query, top_k?)` → 片段、路径、符号、行号、分数
   - `lookup_symbol(name)` → 精确符号查询（有则做）
2. Server 启动时加载索引（或首次调用惰性加载）；路径可配置到 raylib 知识库。
3. 返回值控制长度：Top-K 默认小；每条片段截断。
4. 写 5 个调用样例与期望结果。

**验收标准**

- [ ] MCP 调用检索结果与 CLI 检索一致或可解释地接近。
- [ ] 返回格式适合直接塞进模型上下文。

**自测题**

1. 为什么 MCP 工具返回“全文 dump”往往有害？
2. 索引未建好时，工具应返回什么？
3. 检索工具要不要在 Server 内直接调用 LLM 生成答案？本周建议怎么做？

---

### 周五：接入 Cursor，错误处理与权限

**学习目标**

1. 让 Cursor 能配置并调用你的 Server。
2. 补齐错误处理、日志与权限控制。

**时间安排**：配置联调 50 分钟，加固 25 分钟，记录 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Cursor 文档中的 MCP 说明](https://cursor.com/docs) | 英文 | 必读 | 如何配置 MCP Server |
| 你的 `cursor-setup.md` 草稿 | — | 必读 | 边做边写 |

**动手任务**

1. 按 Cursor 要求配置启动命令、工作目录、环境变量。
2. 在对话里实际触发 `search_code` / `read_file`，保存截图或日志片段。
3. 加固：
   - 权限：只读检索默认开；写操作本周可不暴露
   - 超时与异常捕获
   - 结构化错误码/错误信息
4. 记录常见失败：Python 路径不对、依赖未装、stdio 缓冲、工作目录错误。

**验收标准**

- [ ] Cursor 中至少成功调用一次自写工具并得到正确结果。
- [ ] `cursor-setup.md` 别人照做能接上。

**自测题**

1. Host 崩溃重启后，stdio Server 进程应怎样管理？
2. 为什么生产里要区分“只读工具”和“有副作用工具”？
3. 工具在 Cursor 里“能列出来但调用失败”，优先查哪几层？

---

### 周末：完善与 README

**学习目标**

1. 把 Server 整理到可提交、可演示状态。
2. 为综合项目预留“知识库工具 MCP 化”的接口。

**时间安排**（约 3 小时）：打磨 90 分钟，文档 40 分钟，复盘 30 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| MCP 官方示例与文档 | 英文 | 选读 | 对照最佳实践 |
| 本周全部产出 | — | 必读 | 查漏 |

**动手任务**

1. 统一配置：索引路径、沙箱根目录、日志级别。
2. README：安装、运行、Cursor 配置 JSON/示例、工具列表、安全说明。
3. 补最小自动化测试：schema 存在、沙箱拒绝、检索 Top-1 含期望路径（可用 fixture）。
4. 更新 [进度追踪](../进度追踪.md)。

**验收标准**

- [ ] 自己写的 MCP Server 能在 Cursor 里被调用并返回正确结果。
- [ ] README 完整，工具权限说明清楚。

---

## 5. 本周验收

对应主计划 W16 的验收：

- [ ] 自己写的 MCP Server 能在 Cursor 里被调用并返回正确结果。

## 6. 周复盘

- **实际用时**：
- **验收通过情况**：
- **本周最重要的三个收获**：
- **哪些自测题答不上来**：
- **遗留问题**（带到下周）：
- **哪种学习方式效果好，哪种不好**：
- **下周要调整的地方**：

## 7. 容易踩的坑

- **只读文档不接 Cursor**：验收过不了，周五必须真接。
- **在 MCP 里塞进完整 Agent**：本周焦点是协议与工具暴露。
- **返回巨量文本**：直接打爆上下文。
- **写权限默认打开**：综合项目前先养成最小权限习惯。
- **硬编码绝对路径**：换机器就挂，改成配置。

## 8. 下周预告

[W17](W17-应用框架.md) 用 LangGraph 重写 W15 编码 Agent，并横向了解 LangChain、LlamaIndex、Dify、AutoGen 的定位。

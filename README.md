# Claude Code 项目结构解析

这个仓库主要存放从 `@anthropic-ai/claude-code@2.1.88` 的 npm 发布包中还原出来的源码结构，以及我基于 `cli.js.map` 做的项目结构解析。

这份 README 不再重点讲“如何解包”，而是直接作为这个项目的结构导览。

---

## 一、这个项目里有什么

仓库中的核心内容主要有三部分：

- `unpacked/package/`：npm 包解包后的原始内容
- `extracted/`：根据 `cli.js.map` 还原出来的源码文件
- `map-analysis/`：对还原结果做的结构统计和清单分析

如果你的目标是理解 Claude Code 的实现结构，重点看：

- `extracted/src/`
- `map-analysis/src-summary.txt`
- `map-analysis/tools-tree.txt`
- `map-analysis/services-tree.txt`
- `map-analysis/commands-tree.txt`

---

## 二、整体还原结果

通过解析 `cli.js.map`，总共还原出 **4756** 个文件。

顶层分布如下：

- `src`：**1902** 个文件
- `node_modules`：**2850** 个文件
- `vendor`：**4** 个文件

这说明 `cli.js.map` 中包含了非常完整的源码映射，不只是少量入口文件，而是相当大规模的项目源代码结构。

---

## 三、Claude Code 的整体架构判断

从源码结构来看，Claude Code 不是一个“薄薄一层的 CLI 包装器”，而是一个相当完整的终端 Agent 系统，大致可以拆成下面几层：

1. **CLI 启动层**
2. **命令系统层**
3. **工具运行时（Tool Runtime）**
4. **任务 / Agent 编排层**
5. **对话与查询主循环**
6. **终端 UI 层**
7. **MCP / Remote / Bridge 层**
8. **记忆 / 技能 / 配置层**

也就是说，它本质上更像一个：

- 有完整交互界面
- 有工具调用机制
- 有权限控制
- 有任务系统
- 有多 Agent / 子任务能力
- 有 MCP 和远程能力

的终端智能体平台，而不仅仅是“把 prompt 发给模型”的命令行程序。

---

## 四、各模块结构解析

### 1. CLI 启动层

核心文件：

- `extracted/src/main.tsx`
- `extracted/src/entrypoints/*`
- `extracted/src/cli/*`

这一层负责：

- 程序启动
- 命令行参数处理
- 初始化上下文
- 启动终端界面
- 连接后续的查询主循环和工具系统

从 `main.tsx` 可以看出，这个项目在启动阶段就做了很多预处理，比如：

- 启动 profile / profiler
- 预读取配置
- 预取 secure storage / keychain 信息
- 初始化 telemetry
- 初始化上下文与入口逻辑

这说明 Claude Code 很重视：

- 启动性能
- 配置加载顺序
- 安全存储
- 终端应用级别的初始化流程

---

### 2. 命令系统层

目录：

- `extracted/src/commands/`

还原出的命令相关文件共有 **207** 个。

从目录统计看，里面包含很多命令子系统，例如：

- `plugin`
- `install-github-app`
- `mcp`
- `extra-usage`
- `clear`
- `review`
- `remote-setup`
- `context`
- `add-dir`
- `rename`

这说明 Claude Code 不只是单会话聊天工具，而是内建了比较完整的命令生态。它支持：

- 插件管理
- GitHub 集成
- MCP 相关操作
- 会话上下文管理
- 目录管理
- review 流程
- 远程连接/远程配置能力

这层更像传统 CLI 的“命令入口层”，但后面连的是完整 agent runtime。

---

### 3. 工具运行时（Tool Runtime）

目录：

- `extracted/src/tools/`

这是整个项目最关键的部分之一，还原出的文件共有 **178** 个。

里面能看到大量内建工具，例如：

- `BashTool`
- `FileReadTool`
- `FileWriteTool`
- `FileEditTool`
- `GlobTool`
- `GrepTool`
- `WebFetchTool`
- `WebSearchTool`
- `MCPTool`
- `LSPTool`
- `AgentTool`
- `TodoWriteTool`
- `ConfigTool`
- `SkillTool`
- `ScheduleCronTool`
- `TaskCreateTool / TaskListTool / TaskUpdateTool / TaskOutputTool / TaskStopTool`

这说明 Claude Code 的能力不是硬编码在某几个 if/else 里，而是被组织成一套统一的“工具系统”。

从文件命名还能看出每个工具通常带有：

- 具体实现文件
- `prompt.ts`
- `constants.ts`
- `UI.tsx`

这意味着工具不仅有执行逻辑，还有：

- 提示词定义
- 常量/协议定义
- 独立的交互展示 UI

这是一个非常典型的 agent tool runtime 设计。

---

### 4. 任务 / Agent 编排层

核心文件和目录：

- `extracted/src/Task.ts`
- `extracted/src/tools/AgentTool/*`
- `extracted/src/tools/shared/*`

这里能看出 Claude Code 有完整的任务模型。

`Task.ts` 中可以看到一些明确的任务类型：

- `local_bash`
- `local_agent`
- `remote_agent`
- `in_process_teammate`
- `local_workflow`
- `monitor_mcp`
- `dream`

这非常关键，它说明系统内部不仅仅是“调用一个 tool 然后返回”，而是存在正式的 task lifecycle：

- pending
- running
- completed
- failed
- killed

也就是说，Claude Code 把很多操作都抽象成任务，并允许任务被追踪、终止、恢复、输出到文件、通知 UI。

在 `AgentTool` 下还能看到一组内建 agent：

- `planAgent`
- `verificationAgent`
- `exploreAgent`
- `claudeCodeGuideAgent`

这说明它原生支持“让一个 agent 去做特定角色工作”的模式，而不是单线程单人格的 assistant。

---

### 5. 对话与查询主循环

核心文件：

- `extracted/src/QueryEngine.ts`
- `extracted/src/query.ts`
- `extracted/src/utils/processUserInput/*`
- `extracted/src/services/api/*`

`QueryEngine.ts` 基本可以看作整个系统的大脑之一。

从导入关系就能看出它负责连接：

- system prompt
- memory prompt
- 工具系统
- 权限系统
- 消息结构
- 文件历史
- session storage
- API 调用
- model 选择
- hooks
- query context
- 消息归一化

它不是一个简单的“发请求”模块，而更像是：

- 接收用户输入
- 组织上下文
- 调度模型
- 插入工具调用
- 处理权限与状态
- 记录 session / transcript
- 更新 UI 与内部状态

这基本就是 Claude Code 的主循环控制器。

---

### 6. 终端 UI 层

关键目录：

- `extracted/src/components/`
- `extracted/src/ink/`
- `extracted/src/screens/`
- `extracted/src/hooks/`

相关规模：

- `components`：**389** 个文件
- `hooks`：**104** 个文件
- `ink`：**96** 个文件

这几个数字已经足够说明问题：Claude Code 的终端界面非常重，不是简单打印几行文本。

它很可能是一个基于 React / Ink 的复杂 TUI（Terminal UI）应用，具备：

- 组件化界面
- 状态驱动渲染
- 权限弹层 / 交互面板
- 会话与消息渲染组件
- agent/task 相关 UI
- 输入框与选择器
- loading / spinner / overlay / modal 等终端交互体验

尤其值得注意的是 `components/permissions/*` 规模很大，说明权限提示和用户确认是重要的一层产品设计，而不是附属逻辑。

---

### 7. MCP / Remote / Bridge 层

关键目录：

- `extracted/src/services/mcp/`
- `extracted/src/remote/`
- `extracted/src/bridge/`
- `extracted/src/server/`

这部分说明 Claude Code 不只跑在本地单机模式。

它至少具备这些方向的能力：

- MCP server / client 交互
- bridge 通道
- remote session 管理
- server 侧会话创建或连接

也就是说，它的设计目标更像一个可以扩展、可以远程连接、可以挂接外部工具协议的 Agent 平台。

---

### 8. 记忆 / 技能 / 配置层

关键目录：

- `extracted/src/memdir/`
- `extracted/src/skills/`
- `extracted/src/utils/settings/`
- `extracted/src/migrations/`

这部分说明 Claude Code 把“长期使用”考虑得比较完整。

能看到的方向包括：

- memory prompt / memory scan
- skills 加载
- settings 校验与变更
- 历史配置迁移

这说明它不是一次性 demo，而是一个持续演进、版本迁移、可扩展的产品级系统。

---

## 五、`src/` 目录的主要规模

根据还原结果，`src/` 里的主要模块规模大致如下：

- `commands`：**207**
- `tools`：**178**
- `services`：**130**
- `components`：**389**
- `hooks`：**104**
- `ink`：**96**
- `bridge`：**31**
- `cli`：**19**
- `skills`：**20**
- `memdir`：**8**
- `remote`：**4**
- `server`：**3**

从这个分布可以得出一个很直观的结论：

- 它有很强的 **UI 层**
- 它有很强的 **工具系统**
- 它有比较完整的 **命令和服务层**
- 它已经具备了 **多能力集成型 Agent 产品** 的典型结构

---

## 六、一些特别值得注意的实现方向

### 1. 权限系统很重

在工具层和组件层中都能看到大量和权限有关的内容，例如：

- Bash / PowerShell 的危险命令识别
- 路径校验
- 只读校验
- destructive command warning
- permissions UI
- permission mode / permission rule / denial tracking

这说明 Claude Code 在“模型能不能执行这个动作”这件事上做了很多系统化设计。

---

### 2. Agent / teammate / swarm 能力很明显

从这些目录和文件名里可以看出明显的多 Agent 痕迹：

- `AgentTool`
- `spawnMultiAgent`
- `swarm/*`
- `teammate*`
- `in_process_teammate`

这说明它的架构并不是只为单助手会话设计，而是支持更复杂的 agent collaboration 模式。

---

### 3. 插件和扩展能力很强

从这些目录能看出来：

- `plugins/*`
- `skills/*`
- `services/mcp/*`
- `utils/plugins/*`

说明 Claude Code 是按照“可扩展产品”来设计的，插件、技能、MCP 都是核心扩展机制，而不是边缘能力。

---

### 4. 终端应用工程化程度很高

从 `startupProfiler`、`telemetry`、`migrations`、`secureStorage`、`settings`、`sessionStorage` 这些模块看，Claude Code 的工程化程度是很高的。

它不是一个纯实验脚本，而是一套相当成熟的终端应用系统。

---

## 七、建议从哪里开始读源码

如果你想真正理解它的运行方式，建议按这个顺序读：

### 第一层：入口和总控

- `extracted/src/main.tsx`
- `extracted/src/QueryEngine.ts`
- `extracted/src/query.ts`
- `extracted/src/Tool.ts`
- `extracted/src/Task.ts`

先建立对“主循环 + 工具 + 任务”的整体认识。

### 第二层：工具系统

- `extracted/src/tools/BashTool/*`
- `extracted/src/tools/FileReadTool/*`
- `extracted/src/tools/FileEditTool/*`
- `extracted/src/tools/AgentTool/*`
- `extracted/src/tools/MCPTool/*`

这层能帮助理解 Claude Code 实际是怎么和外部世界交互的。

### 第三层：服务和上下文

- `extracted/src/services/api/*`
- `extracted/src/services/mcp/*`
- `extracted/src/utils/processUserInput/*`
- `extracted/src/utils/permissions/*`

这层能帮助理解请求调度、权限判定、上下文处理。

### 第四层：UI

- `extracted/src/components/*`
- `extracted/src/ink/*`
- `extracted/src/screens/*`

如果你想理解产品交互和终端体验，这部分值得单独看。

---

## 八、仓库里的分析文件说明

在 `map-analysis/` 目录里，我额外整理了这些文件：

- `sources.json`：source map 的基础信息
- `file-list.txt`：还原出来的完整文件列表
- `top-level-summary.txt`：顶层文件统计
- `src-summary.txt`：`src/` 目录模块统计
- `tools-tree.txt`：工具目录清单
- `services-tree.txt`：服务目录清单
- `commands-tree.txt`：命令目录清单
- `core-snippets.txt`：几个核心入口文件的摘录

这些文件适合做进一步逆向分析时快速定位。

---

## 九、结论

基于这次从 source map 还原出的文件结构，可以把 Claude Code 粗略理解为：

> 一个以终端为主要交互界面、以工具系统为执行核心、以 QueryEngine 为主循环中枢、同时具备多 Agent、权限系统、MCP 扩展、技能系统和远程能力的工程化 AI Agent 平台。

如果只是把它理解为一个“命令行版 Claude”，其实是低估了它的复杂度。

从结构上看，它更像是一个：

- 终端产品
- Agent runtime
- Tool orchestration system
- 可扩展插件/技能平台
- 支持远程与协作的智能体框架

的组合体。

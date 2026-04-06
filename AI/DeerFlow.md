## 一、项目定位

**DeerFlow**（**D**eep **E**xploration and **E**fficient **R**esearch **Flow**）是字节跳动开源的超级智能体运行时（Super Agent Harness）。它不是"又一个 ChatBot"，而是一个**可编排、可扩展、可嵌入**的 Agent 基础设施。

**v1 → v2 的跃迁**——2026 年初，DeerFlow 团队做了一个大胆的决定：将整个项目从头重写，v1 和 v2 之间零代码共享：

![](https://cdn.nlark.com/yuque/__mermaid_v3/b5bd06f0b41d775859e1ce16f04a876c.svg)

为什么要做这个跃迁？<font style="background-color:#C1E77E;">v1 的问题在于它把"深度研究"写死在了工作流里——Coordinator → Planner → Researcher/Coder → Reporter 是一条固定流水线。而现实世界的 Agent 需求千变万化：有时用户只想写一段代码，有时需要深度调研，有时需要操作浏览器。v2 的答案是</font>**<font style="background-color:#C1E77E;">把工作流变成配置，把固定角色变成可组合的中间件</font>**。2026 年 2 月 28 日发布后，DeerFlow 登顶 GitHub Trending #1，证明了这个方向的正确性。

---

## 二、整体架构

### 2.1 四进程运行模型

当你执行 `make dev` 启动 DeerFlow 时，背后实际上启动了**四个独立进程**，通过 Nginx 统一对外暴露在 2026 端口。这不是过度设计——每个进程承担着截然不同的职责，且可以独立伸缩和重启：

![](https://cdn.nlark.com/yuque/__mermaid_v3/495957478715f94de4a506f1d571b3e4.svg)

这里有一个关键的架构决策值得展开讨论——**为什么要双后端进程？** 答案在<font style="background-color:#C1E77E;">于</font>**<font style="background-color:#C1E77E;">有状态 vs 无状态</font>**<font style="background-color:#C1E77E;">的本质差异。LangGraph Server 是天然有状态的：它维护着对话的 Checkpoint、管理着流式输出的长连接、懒加载着 MCP 工具实例。而 Gateway 是纯粹的无状态 REST 服务：模型列表查询、配置读写、文件上传。把这两者放在同一个进程里会导致一个尴尬的问题——当你重启 Gateway 更新配置时，所有正在进行的 Agent 对话都会被打断。分离后，两者独立伸缩、独立重启，互不干扰。</font>

还有一个精妙的细节：**MCP 工具只在 LangGraph 侧懒加载**。Gateway 的 `app.py` 中有一条注释明确说明了这一点——它只负责配置的 CRUD，不消耗额外的 MCP 连接资源。这意味着即使你配置了 50 个 MCP 服务器，Gateway 的内存占用也几乎不变。

### 2.2 代码分层架构

接下来这张图是理解整个项目"可嵌入"能力的关键。DeerFlow 的代码被严格切分为两层——应用层 `app.*` 和框架层 `deerflow.*`，两者之间有一条**单向依赖的硬边界**：

![](https://cdn.nlark.com/yuque/__mermaid_v3/55e22ee28b002e9b483a841e68ac909c.svg)

这里的规则很简单但很严格：`app`** 可以导入 **`deerflow`**，但 **`deerflow`** 绝不导入 **`app`。这不是靠"团队约定"来保证的——项目中有一个 `test_harness_boundary.py` 测试文件，在每次 CI 运行时自动扫描所有 `deerflow.*` 模块的 import 语句，一旦发现反向依赖就会立即报错。

<font style="background-color:#C1E77E;">这条边界带来了一个巨大的好处：</font>`<font style="background-color:#C1E77E;">deerflow</font>`<font style="background-color:#C1E77E;"> 包可以</font>**<font style="background-color:#C1E77E;">独立发布为 pip 包</font>**<font style="background-color:#C1E77E;">。任何 Python 应用——不管是 Django、Flask 还是一个简单的脚本——都可以通过 </font>`<font style="background-color:#C1E77E;">DeerFlowClient</font>`<font style="background-color:#C1E77E;"> 直接嵌入使用，完全不需要 FastAPI、不需要 Nginx、不需要前端。对于想把 Agent 能力集成到现有系统的团队来说，这是一个零摩擦的入口。</font>`<font style="background-color:#C1E77E;">DeerFlowClient</font>`<font style="background-color:#C1E77E;"> 提供了与 Gateway REST API 完全一致的方法签名（</font>`<font style="background-color:#C1E77E;">stream</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">chat</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">list_models</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">get_memory</font>`<font style="background-color:#C1E77E;"> 等），并通过 77 个单测（</font>`<font style="background-color:#C1E77E;">TestGatewayConformance</font>`<font style="background-color:#C1E77E;">）持续验证这种一致性。</font>

---

## 三、核心智能体系统

### 3.1 Lead Agent 构建流程

<font style="background-color:#C1E77E;">DeerFlow 没有 v1 时代的多个固定角色（Coordinator → Planner → Researcher → Reporter）。取而代之的是</font>**<font style="background-color:#C1E77E;">一个 Lead Agent + 可选的子代理委派</font>**。Lead Agent 的构建过程由 `make_lead_agent()` 函数完成，它从 `RunnableConfig` 中读取运行时参数，经过四个并行步骤，最终组装出一个完整的 Agent 实例：

![](https://cdn.nlark.com/yuque/__mermaid_v3/6fb2d3cc024bffb791ecac3d6723c7b6.svg)

注意第④步的系统提示生成——`apply_prompt_template()` 不是简单地拼接字符串。它会根据当前配置**条件性地注入**多个段落：如果启用了子代理，就注入 `<subagent_system>` 段落和并发分解的思考指令；如果有已加载的 Skills，就注入技能说明；如果有记忆数据，就注入用户偏好和历史事实。这意味着**同一个 Agent 在不同配置下拿到的系统提示是完全不同的**——这正是"可编排"的体现。

每个 Agent 实例都绑定一个 `ThreadState`——这是 Agent 在每个对话线程中维护的完整状态视图：

![](https://cdn.nlark.com/yuque/__mermaid_v3/92d95cb70349a64b8292f41a6db40616.svg)

### 3.2 中间件链：12 层洋葱模型

如果只能选一个概念来解释 DeerFlow 的设计哲学，那就是**中间件链**。

类比 Express.js 或 Koa 的中间件，<font style="background-color:#C1E77E;">DeerFlow 在 Agent 和 LLM 之间插入了一条严格有序的处理链。每个中间件通过 </font>`<font style="background-color:#C1E77E;">before_model</font>`<font style="background-color:#C1E77E;">（LLM 调用前）和 </font>`<font style="background-color:#C1E77E;">after_model</font>`<font style="background-color:#C1E77E;">（LLM 调用后）两个钩子参与处理，形成了一个经典的"洋葱模型"——请求从外层穿入，响应从内层穿出：</font>

![](https://cdn.nlark.com/yuque/__mermaid_v3/f36ad394e7f0198e5904e738f25b85d2.svg)

让我逐一解读几个关键中间件，帮助大家理解它们为什么要存在、为什么要按这个顺序排列：

+ **① ThreadData** 必须排在最前面——因为后续所有中间件可能需要读写线程目录（workspace/uploads/outputs），如果目录还不存在就会报错。

+ **④ DanglingToolCall** 解决了一个 LLM 调用的边缘情况：当 Agent 上一轮调用了工具但因超时或异常中断时，消息列表中会留下一个没有对应 `ToolMessage` 的 `AIMessage`。如果不补全这条消息，LLM 在下一轮调用时会直接报错。

+ **<font style="background-color:#C1E77E;">⑤ Guardrail</font>**<font style="background-color:#C1E77E;"> 是安全门卫——它拦截 LLM 返回的工具调用请求，交给外部 Provider 评估（比如检查 bash 命令是否危险），不允许的操作直接返回拒绝消息，而不是执行。</font>

+ **⑫ LoopDetection** 防止 Agent 陷入死循环。它对最近 20 条消息的 tool_calls 做 MD5 哈希，如果同一模式重复 3 次则注入警告，5 次则强制剥掉 `tool_calls` 迫使 Agent 输出文本。

+ **<font style="background-color:#C1E77E;">⑬ Clarification 永远排在最后</font>**<font style="background-color:#C1E77E;">——因为它会通过 </font>`<font style="background-color:#C1E77E;">Command(goto=END)</font>`<font style="background-color:#C1E77E;"> 直接中断整个图的执行，把控制权交还给用户。任何排在它后面的中间件都不会被执行。</font>

**设计原则**：

| 原则 | 说明 |
| --- | --- |
| **单一职责** | 每个中间件只做一件事，不互相耦合 |
| **顺序敏感** | ThreadData 在 Sandbox 之前（确保目录存在）；Clarification 在最后（它会中断流程） |
| **可选组合** | Guardrail、Summarization、TodoList、ViewImage、SubagentLimit 按配置条件启用 |
| **自定义注入** | 支持在 Clarification 之前插入自定义中间件（`custom_middlewares` 参数） |

这个设计的优雅之处在于：<font style="background-color:#C1E77E;">如果你想给 Agent 增加一个新的能力（比如自动翻译、日志审计），你只需要实现一个 </font>`<font style="background-color:#C1E77E;">AgentMiddleware</font>`<font style="background-color:#C1E77E;"> 子类，然后通过 </font>`<font style="background-color:#C1E77E;">custom_middlewares</font>`<font style="background-color:#C1E77E;"> 参数注入即可——不需要修改 Agent 本身的任何代码。</font>

### 3.3 子智能体委派模型

DeerFlow 的多智能体不是对等协商，而是**主从委派**——Lead Agent 是唯一的决策者，子代理只是执行者。

这是一个有意识的设计取舍。<font style="background-color:#C1E77E;">业界很多多 Agent 框架（如 AutoGen、CrewAI）采用"对等协商"模式，让多个 Agent 互相讨论达成共识</font>。<font style="background-color:#C1E77E;">DeerFlow 认为这种模式在实际应用中有两个问题：一是对话轮次难以控制，容易陷入无限讨论；二是最终结果的质量取决于"协商能力最弱的那个 Agent"。主从委派模式则简单直接——Lead Agent 负责理解需求、分解任务、综合结果，子代理只需要专注执行：</font>

![](https://cdn.nlark.com/yuque/__mermaid_v3/b0339c0b1d63d5f22446f273081907d9.svg)

<font style="background-color:#C1E77E;">这里有几个值得深挖的实现细节。首先是 </font>**<font style="background-color:#C1E77E;">SubagentLimitMiddleware</font>**<font style="background-color:#C1E77E;">——它在 LLM 输出后立即检查，如果 Lead Agent 一次性发起了过多的 </font>`<font style="background-color:#C1E77E;">task</font>`<font style="background-color:#C1E77E;"> 工具调用（比如 5 个），中间件会</font>**<font style="background-color:#C1E77E;">静默截断</font>**<font style="background-color:#C1E77E;">到最大并发数（默认 3，硬限制为 2~4）。这避免了 LLM "过于热心"地并行化导致资源耗尽。</font>

然后是执行层的**双线程池架构**：

![](https://cdn.nlark.com/yuque/__mermaid_v3/87382514ccaf891066e552811de3508d.svg)

为什么需要两个线程池？因为<font style="background-color:#C1E77E;"> Scheduler Pool 负责</font>**<font style="background-color:#C1E77E;">调度和超时监控</font>**<font style="background-color:#C1E77E;">（15 分钟）</font>，而<font style="background-color:#C1E77E;"> Execution Pool 负责</font>**<font style="background-color:#C1E77E;">实际的 Agent 执行</font>**。如果把这两个职责混在一起，一个执行缓慢的子代理就可能占满所有 worker，导致其他子代理无法被调度。分离后，Scheduler 的 worker 只做轻量的"提交 + 等待"，执行时间几乎为零。

目前内置了两种子代理类型：`general-purpose`（通用型，拥有父代理除 `task` 外的全部工具）和 `bash`（命令执行专家，仅限 `bash`、`ls`、`read_file`、`write_file`、`str_replace`）。它们通过 `SubagentConfig` 数据类配置，支持在 `config.yaml` 中覆盖超时时间等参数。

<font style="background-color:#C1E77E;">最关键的安全设计：</font>**<font style="background-color:#C1E77E;">子代理不能再派生子代理</font>**<font style="background-color:#C1E77E;">。</font>`<font style="background-color:#C1E77E;">general-purpose</font>`<font style="background-color:#C1E77E;"> 的 </font>`<font style="background-color:#C1E77E;">disallowed_tools</font>`<font style="background-color:#C1E77E;"> 列表包含 </font>`<font style="background-color:#C1E77E;">task</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">ask_clarification</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">present_files</font>`<font style="background-color:#C1E77E;">，从根本上杜绝了无限递归的可能性。同时，子代理继承父代理的 </font>`<font style="background-color:#C1E77E;">sandbox_state</font>`<font style="background-color:#C1E77E;"> 和 </font>`<font style="background-color:#C1E77E;">thread_data</font>`<font style="background-color:#C1E77E;">，确保它们在同一个沙箱环境中工作，共享同一套文件系统视图。</font>

---

## 四、工具生态与扩展

### 4.1 四大工具来源

Agent 能力的强弱，很大程度上取决于它手里有什么工具。<font style="background-color:#C1E77E;">DeerFlow 的工具系统设计了</font>**<font style="background-color:#C1E77E;">四个来源层次</font>**<font style="background-color:#C1E77E;">，从静态配置到动态发现，</font>层层递进。最终通过 `get_available_tools()` 函数统一组装，这个函数内部使用 `resolve_variable()` 反射加载——`config.yaml` 中的 `use: deerflow.sandbox.tools:bash_tool` 这样的字符串会被解析为实际的 Python 对象：

![](https://cdn.nlark.com/yuque/__mermaid_v3/1d0accb299cd85c7e506ead9e039bc8f.svg)

每一层的设计意图不同：

+ **Config 声明工具**是基本盘——在 `config.yaml` 的 `tools` 列表中声明，通过 `tool_groups`（`web`、`file:read`、`file:write`、`bash`）分组管理。不同的 Agent 人格可以通过引用不同的 `tool_groups` 来获得不同的工具子集。

+ **MCP 动态工具**是扩展性的核心——支持 stdio、SSE、HTTP 三种传输协议，甚至支持 OAuth 认证。MCP 工具的加载是**懒惰的**：首次使用时才建立连接，之后通过 `extensions_config.json` 文件的 mtime 判断是否需要重新加载。这意味着你可以随时通过 Gateway API 更新 MCP 配置，下一次 Agent 调用就会自动感知变化。

+ **内置工具**是框架的"器官"——`present_files` 将生成的文件呈现给前端（校验路径必须在 `outputs` 目录内），`ask_clarification` 触发人在回路中断，`view_image` 把图片转为 base64 注入视觉模型，`task` 触发子代理委派。

+ **ACP 代理工具**是 Agent-to-Agent 通信——允许 DeerFlow 调用外部 Agent（如 Claude Code、Codex），这些外部 Agent 的工作空间被映射到只读的 `/mnt/acp-workspace`。

### 4.2 Tool Search：延迟加载机制

这是一个非常实用的优化。当 MCP 服务器暴露大量工具时（50+），如果把所有工具的 schema 全部塞入系统提示，会产生两个问题：一是浪费大量 token（每个工具描述可能占 200~500 token），二是 LLM 在选择工具时会"目不暇接"，准确率显著下降。

DeerFlow 的解决方案是**Tool Search 模式**——只给 Agent 一个"搜索工具"，让它按需检索：

![](https://cdn.nlark.com/yuque/__mermaid_v3/67680137bbe4f67ca3593d05b912397c.svg)

<font style="background-color:#C1E77E;">这里有一个精妙的协作：</font>`<font style="background-color:#C1E77E;">tool_search</font>`<font style="background-color:#C1E77E;"> 只是一个"搜索入口"，它从 </font>`<font style="background-color:#C1E77E;">DeferredToolRegistry</font>`<font style="background-color:#C1E77E;"> 中检索匹配的工具并注入 Agent 上下文。但当 Agent 实际调用这些"按需检索到的工具"时，</font>`<font style="background-color:#C1E77E;">DeferredToolFilterMiddleware</font>`<font style="background-color:#C1E77E;"> 负责在运行时</font>**<font style="background-color:#C1E77E;">将延迟工具还原为可执行的实际工具</font>**<font style="background-color:#C1E77E;">。整个过程对 Agent 完全透明——它不知道也不需要知道工具是预加载的还是延迟加载的。通过 </font>`<font style="background-color:#C1E77E;">config.yaml</font>`<font style="background-color:#C1E77E;"> 中 </font>`<font style="background-color:#C1E77E;">tool_search.enabled: true</font>`<font style="background-color:#C1E77E;"> 即可开启此模式。</font>

---

## 五、沙箱与虚拟路径系统

### 5.1 虚拟路径映射 沙箱系统要解决的核心问题是：**<font style="background-color:#C1E77E;">如何让 Agent 安全地操作文件系统，同时在本地开发、Docker 容器和 K8s Pod 三种环境下保持一致的行为</font>****？** DeerFlow 的答案是引入一层**虚拟路径抽象**。Agent 永远只看到统一的虚拟路径（比如 `/mnt/user-data/workspace`），工具层通过 `replace_virtual_path()` 函数做透明翻译，将虚拟路径映射到每个对话线程独立的物理目录： ![](https://cdn.nlark.com/yuque/__mermaid_v3/34e87fb0e27a1f59c6892b069f45ccf3.svg) 这个设计有三个好处。第一，**线程隔离**——每个对话线程有自己独立的 workspace、uploads、outputs 目录，Agent A 的文件操作不会影响 Agent B。第二，**环境无关**——无论底层是本地目录还是 Docker 卷挂载还是 K8s PV，Agent 代码都不需要改动。第三，**安全边界**——`present_files` 工具在呈现文件前会校验路径必须落在 `outputs` 目录内，防止 Agent 将系统文件暴露给用户。系统提示中也明确告知了 Agent 这些虚拟路径的用途和约束。

### 5.2 三种沙箱模式对比 在实际部署中，不同场景对隔离级别的要求差异很大。本地开发时你可能完全不需要隔离，标准部署时需要容器级隔离，而多租户生产环境需要每个用户会话独享一个 Pod。DeerFlow 通过 **Provider 模式**统一了这三种场景——<font style="background-color:#C1E77E;">所有沙箱都实现同一个抽象接口（</font>`<font style="background-color:#C1E77E;">execute_command</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">read_file</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">write_file</font>`<font style="background-color:#C1E77E;">、</font>`<font style="background-color:#C1E77E;">list_dir</font>`<font style="background-color:#C1E77E;">），通过 </font>`<font style="background-color:#C1E77E;">config.yaml</font>`<font style="background-color:#C1E77E;"> 中的 </font>`<font style="background-color:#C1E77E;">sandbox.use</font>`<font style="background-color:#C1E77E;"> 字段切换：</font> ![](https://cdn.nlark.com/yuque/__mermaid_v3/d120e5ccd8c087eae75fda88dcc8f61d.svg) 一个有趣的实现细节：`LocalSandboxProvider` 的 `acquire()` 返回一个跨轮次复用的单例，其 `release()` 是空操作（no-op）。而 `AioSandboxProvider` 在 K8s 模式下，`acquire()` 会通过 `provisioner_url` 向外部调度器请求一个独立 Pod，`release()` 则归还 Pod。这种设计确保了 **Agent 代码完全不关心底层是哪种模式**——`SandboxMiddleware` 在 `before_agent` 时获取沙箱，`after_agent` 时释放，中间的所有工具调用都通过同一套抽象接口操作。

---

## 六、Context Engineering：长对话工程化 Agent 最容易被忽视但最影响用户体验的问题是**上下文管理**。当对话超过 20 轮，context window 接近上限时，直接截断会丢失关键信息，全量保留又会超出 token 限制。DeerFlow 用三个机制组成了一套完整的"Context Engineering"方案：短期靠自动摘要，长期靠结构化记忆，专业能力靠 Skills 渐进加载。

### 6.1 Summarization 自动摘要 自动摘要是 DeerFlow 处理长对话的第一道防线。`SummarizationMiddleware` 使用 LangChain 内置的摘要机制，在每次 LLM 调用前检测对话长度是否超过阈值： ![](https://cdn.nlark.com/yuque/__mermaid_v3/7bc39d23b51da502b04085eb7010cce5.svg) 三个触发条件是**OR 逻辑**——任何一个满足就会启动摘要。这个设计考虑了不同 LLM 的差异：有些模型的 context window 很大但推理变慢，用 token 数触发；有些模型的 window 较小，用消息条数或比例触发更及时。摘要完成后，较早的消息被替换为一段摘要文本，最近 10 条消息原样保留——因为用户最可能回溯的是近期对话。

### 6.2 长期记忆系统 摘要解决的是"单次对话不超限"的问题，但 Agent 还需要**跨对话**的记忆<font style="background-color:#C1E77E;">——记住用户是谁、喜欢什么风格、最近在做什么项目</font>。DeerFlow 的<font style="background-color:#C1E77E;">长期记忆系统是完全异步、不阻塞对话的</font>，让我们看看它的完整数据流： ![](https://cdn.nlark.com/yuque/__mermaid_v3/cfa92589fe75b1a0541d538505b9d178.svg) 这里有几个值得关注的工程决策： **<font style="background-color:#C1E77E;">消息过滤</font>**<font style="background-color:#C1E77E;">——</font>`<font style="background-color:#C1E77E;">MemoryMiddleware</font>`<font style="background-color:#C1E77E;"> 不会把所有消息都送进记忆系统。它只保留用户的原始输入和 Agent 的最终文本回复，剥掉了工具调用消息、</font>`<font style="background-color:#C1E77E;"><uploaded_files></font>`<font style="background-color:#C1E77E;"> 标签等"技术噪音"。这大幅减少了 LLM 提取事实时的干扰。</font> **30 秒防抖**——用户在快速连续发送多条消息时（比如"帮我写个函数"→"参数名改成 x"→"加个 docstring"），`MemoryQueue` 会用 `threading.Timer` 做 30 秒防抖，同一线程只保留最新一条，避免频繁触发昂贵的 LLM 调用。 **原子写入**——`memory.json` 的更新采用 `temp file → rename` 模式，确保即使进程崩溃也不会写入损坏的文件。写入后立即使缓存失效，下次读取时重新从文件加载。 **<font style="background-color:#C1E77E;">记忆数据结构</font>**<font style="background-color:#C1E77E;">——这是整个记忆系统最精妙的部分，它不是存聊天记录，而是</font>**<font style="background-color:#C1E77E;">结构化事实提取</font>**<font style="background-color:#C1E77E;">：</font> ![](https://cdn.nlark.com/yuque/__mermaid_v3/604ab7641bd9a00b907bdf08371589a2.svg) <font style="background-color:#C1E77E;">每个 Fact 有五个类别（preference/knowledge/context/behavior/goal）和一个 confidence 分数。LLM 在提取时如果对某个事实不确定（confidence < 0.7），这条事实就不会被存储。当事实总数超过 </font>`<font style="background-color:#C1E77E;">max_facts</font>`<font style="background-color:#C1E77E;">（默认 100），会按 confidence 从低到高淘汰。注入时，</font>`<font style="background-color:#C1E77E;">format_memory_for_injection()</font>`<font style="background-color:#C1E77E;"> 会按 confidence 降序排列，用 tiktoken 精确计算 token 预算（最多 2000 tokens），确保最重要的事实优先被 Agent 看到。</font>

### 6.3 Skills 渐进加载 Skills 是 DeerFlow 实现"专业能力可插拔"的机制。每个 Skill 本质上是一个 `SKILL.md` 文件，包含 YAML frontmatter（声明名称、描述、允许的工具列表）和 Markdown 正文（详细的领域知识和行为指令）。Agent 在启用某个 Skill 后，其正文会被注入系统提示，相当于给 Agent 临时"植入"了一段专业知识： ![](https://cdn.nlark.com/yuque/__mermaid_v3/e3b5d6cc8f69a2b0ca4c0ea5a736299b.svg) Skills 分为两类目录：`public/` 是官方提供的公共技能，`custom/` 是用户自定义的私有技能（被 gitignore）。开关状态存储在 `extensions_config.json` 中，Gateway API 提供了 `PUT /api/skills/{name}` 端点供前端动态切换。默认情况下，`public` 和 `custom` 类别的技能自动启用，无需手动配置。 子代理也可以继承 Skills——当 Lead Agent 派发子任务时，可以在 `task` 工具的参数中指定需要的 Skills，子代理的系统提示会相应地拼接技能内容。

---

## 七、配置系统

### 7.1 双配置 + 热加载 DeerFlow 的配置系统面临一个矛盾：模型、工具、沙箱这些"部署级"配置相对稳定，而 MCP 服务器和 Skills 开关则需要在运行时频繁调整。把它们放在同一个文件里会导致频繁的冲突和误改。DeerFlow 的解决方案是**双配置文件 + 基于文件修改时间的热加载**： ![](https://cdn.nlark.com/yuque/__mermaid_v3/73ad5c390841d5599851daf438339e9b.svg) <font style="background-color:#C1E77E;">这里的热加载机制非常轻量：</font>`<font style="background-color:#C1E77E;">get_app_config()</font>`<font style="background-color:#C1E77E;"> 和 </font>`<font style="background-color:#C1E77E;">ExtensionsConfig.from_file()</font>`<font style="background-color:#C1E77E;"> 在每次调用时对比文件的 </font>**<font style="background-color:#C1E77E;">mtime</font>**<font style="background-color:#C1E77E;">（修改时间戳），只有文件真正被修改过才会重新解析。这意味着即使每秒调用 1000 次 </font>`<font style="background-color:#C1E77E;">get_app_config()</font>`<font style="background-color:#C1E77E;">，开销也只是一次 </font>`<font style="background-color:#C1E77E;">os.stat()</font>`<font style="background-color:#C1E77E;"> 系统调用。</font> 一个值得注意的差异：`config.yaml` 中的 `$VAR` 环境变量如果缺失会**直接报错**（这是部署级配置，缺失意味着配置错误），而 `extensions_config.json` 中的 `$VAR` 缺失会**静默替换为空字符串**（MCP 服务器可能在某些环境中不可用，这是合理的）。

### 7.2 配置版本管理 开源项目中一个常见的痛点：项目更新后 `config.yaml` 新增了字段，用户却还在用旧版本的配置文件，导致运行时报错或行为异常。DeerFlow 用一个简单但有效的 **config_version** 机制解决了这个问题： ![](https://cdn.nlark.com/yuque/__mermaid_v3/3c22a2dd0e9314bf66a2aff8868b20fa.svg) `_check_config_version()` 在每次 `AppConfig.from_file()` 时自动执行：读取 `config.example.yaml` 中的最新版本号，与用户的 `config.yaml` 对比。版本落后时输出警告并提示运行 `make config-upgrade`。升级过程是**非破坏性的**——它会合并新字段（使用默认值），同时保留用户已修改的自定义值。

---

## 八、多端接入 <font style="background-color:#C1E77E;">DeerFlow 的"可嵌入"不只是一句口号——它真正支持从 Web 前端到 IM 机器人到纯 Python SDK 的全渠道接入。每种接入方式使用</font>**<font style="background-color:#C1E77E;">最适合自身的通信模式</font>**<font style="background-color:#C1E77E;">，但最终都汇聚到同一个 LangGraph Server 执行 Agent 逻辑：</font> ![](https://cdn.nlark.com/yuque/__mermaid_v3/11494796fdecbbab86655329e594d386.svg) 注意图中三种不同的连接方式：Web 前端通过 LangGraph SDK **直连** LangGraph Server（绕过 Gateway），获得最低延迟的流式体验；三个 IM 通道（飞书/Slack/Telegram）通过 **MessageBus 消息总线**解耦，由 `ChannelManager` 统一调度；嵌入式 `DeerFlowClient` 则**直接调用同一份 Agent 代码**，零 HTTP 开销。 让我们用飞书场景展开看看 IM 消息的完整生命周期——这是一个典型的**流式卡片更新**模式： ![](https://cdn.nlark.com/yuque/__mermaid_v3/336502f666437424784b10c2eff98888.svg) 飞书使用 `runs.stream()` 获得流式输出，每收到一段文本就 **patch 同一张卡片**（而不是发送新消息），用户看到的效果就像 ChatGPT 那样逐字出现。Slack 和 Telegram 因为平台限制使用 `runs.wait()` 阻塞式等待完整结果。这种差异被 `MessageBus` 的 pub/sub 模型完美抽象——`ChannelManager` 发布 `is_final=false` 的中间结果和 `is_final=true` 的最终结果，各通道根据自身能力选择消费。 **嵌入式客户端**则是完全不同的路径。`DeerFlowClient` 直接在进程内创建 Agent 实例，通过 `stream()` 和 `chat()` 方法交互，产出与 LangGraph SSE 对齐的 `StreamEvent`。它的 API 签名与 Gateway 的 Pydantic 模型一一对应，通过 `TestGatewayConformance`（77 个单测）持续验证一致性——确保用 SDK 写的代码可以无缝迁移到 HTTP API，反之亦然。

---

## 九、工程质量

### 可借鉴的设计模式一览 ![](https://cdn.nlark.com/yuque/__mermaid_v3/48c11fad66dcd9729b0a95f5a2b043a5.svg) 这张思维导图中的每一项都不是"锦上添花"的 nice-to-have，而是在真实生产环境中踩过坑后沉淀出来的 must-have。几个特别值得展开的：

+ **反射加载 **`resolve_variable()`——`config.yaml` 中的 `use: deerflow.sandbox.tools:bash_tool` 不是硬编码，而是运行时解析为 Python 对象。这意味着你可以在配置文件中引用任何符合接口的第三方实现，不需要修改框架代码。

+ **Provider 模式**——沙箱有 `SandboxProvider`，模型有 `create_chat_model()`，护栏有 `GuardrailProvider`，所有"可替换"的组件都通过 Provider 接口解耦。

+ **HTML/SVG 强制下载**——这是一个安全细节：当 Agent 生成了 HTML 或 SVG 文件时，Gateway 在返回文件产物时强制设置 `Content-Disposition: attachment`，防止浏览器直接渲染可能包含恶意脚本的内容。

---

## 十、总结 回顾全文，DeerFlow 2.0 不是一个对话应用，而是一个**超级智能体运行时**。如果用一句话概括它的设计哲学：**把 Agent 开发从"手写逻辑"变成"配置 + 组合"**。 以下是我认为最值得借鉴的五个架构决策，每一个都解决了 Agent 开发中的一个真实痛点： ![](https://cdn.nlark.com/yuque/__mermaid_v3/eb00b6b00021ec51dc6d9ffae0334ae2.svg)

+ **中间件链模式**解决了"Agent 行为难以扩展"的问题——新增能力只需插入一个中间件，不修改任何现有代码。

+ **虚拟路径沙箱**解决了"Agent 代码在不同环境表现不一致"的问题——从本地开发到 K8s 生产环境，Agent 代码一行不改。

+ **Harness/App 硬边界**解决了"框架难以复用"的问题——`deerflow-harness` 可以作为独立 pip 包发布，任何 Python 应用都能嵌入。

+ **双进程分离**解决了"有状态服务和无状态服务混合部署"的问题——独立伸缩、独立重启、互不干扰。

+ **渐进加载**解决了"上下文膨胀导致 Agent 准确率下降"的问题——Tool Search 和 Skills 按需注入，精确控制 token 预算。 **适用场景**：深度研究、自动化工作流、内容生成、代码编写、IM Bot，以及任何需要"可编程 Agent 基础设施"的场景。如果你的团队正在构建 Agent 产品，DeerFlow 的架构设计——尤其是中间件链、虚拟路径沙箱和双配置热加载——值得深入研究和借鉴。

---

> 基于 DeerFlow 2.0 源码深度分析生成 | 仓库：github.com/bytedance/deer-flow >

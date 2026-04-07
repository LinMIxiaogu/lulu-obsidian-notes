## 整体对比

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773975330171-1ef18ac3-82cb-4d26-9c75-28c1f7477993.png" width="717" title="" crop="0,0,1,1" id="ua553eeb3" class="ne-image">

  

- Deep Agents 是给开发者**构建**自己的智能体用的 SDK。
    
- OpenCode 是给开发者**直接用**的编码工具。
    
- OpenClaw 是给普通用户**委托日常任务**的个人 AI 助手——定位上根本就不在同一个赛道。
    

## Agent Loop：最核心的区别

|   |   |   |   |   |
|---|---|---|---|---|
|维度|Claude Code|OpenCode|DeepAgent|OpenClaw|
|Loop 基础|专有单线程（`nO`）|Vercel AI SDK `streamText`|LangGraph StateGraph|Pi `agentLoop`|
|工具执行|并行 + BatchTool|并行（事件总线）|顺序（Middleware）|顺序（Pi 设计）|
|循环控制|模型 + `h2A` 侧信道|模型 + 事件驱动|图节点转移|模型决定 + 外层重试|
|上下文注入|CLAUDE.md 每次重注入|运行时读取 Markdown|Middleware 动态拼装|每次 attempt 动态构建|
|压缩策略|自动摘要旧历史|自动摘要|SummarizationMiddleware|压缩前先提取事实到 MEMORY.md|
|停止条件|模型无工具调用|模型无工具调用|图终止节点|模型无工具调用|

  

  

实现方式的对比

  

**1. OpenCode：基于文件系统权限手动plan-and-execute切换**

  

- 硬性状态切换：通过 `plan.txt`（规划）与 `build-switch.txt`（执行）两套独立的系统提示词，在运行时强制切换 Agent 的“人格”。
    
- 权限物理隔离：在规划阶段，系统从底层关闭写工具权限，严禁 Agent “边想边乱动”；执行阶段则通过 Bun 运行时与 LSP 诊断回路，将代码纠错形成闭环。
    

**2. DeepAgents：分层递归plan-and-execute**

  

- 任务递归分治：采用层次化任务图（HTDAG）结构，每个节点都配备专属 Planner，非原子任务会像细胞分裂一样，递归触发新的子规划逻辑。
    
- 带“眼”的校验闭环：执行器内置 Validator 组件，每步操作后通过“截图快照”进行视觉比对。一旦结果偏离预期，立即熔断下游任务并指挥 Planner 重新画图。
    

**3. OpenClaw：心跳驱动reAct**

  

- 规划即执行：彻底打破独立的规划阶段，将推理链条直接转化为工具调用序列，实现“所思即所做”的流式响应。
    
- 心跳与数字记忆：系统运行由 Gateway 进程的心跳信号驱动，并将执行过程以 Markdown 文件形式本地持久化，赋予 Agent 跨会话的记忆继承能力。
    

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774256609548-247ed637-9b8e-4e77-8499-af04f83f6add.png" width="704" title="" crop="0,0,1,1" id="ua5b5cb97" class="ne-image">
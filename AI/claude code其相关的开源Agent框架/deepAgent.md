## 系统架构

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773977390563-a3ba8b1b-b81e-4b0c-9ccb-3aa841918a8e.png" width="551" title="" crop="0,0,1,1" id="u4d17b4f6" class="ne-image">

  

Deep Agents 是 LangChain 推出的 agent harness，构建在 LangChain 和 LangGraph 之上，内置 Planning 工具、Filesystem 后端和 Subagent 派生能力，最初的设计灵感来自 Claude Code。

  

## 核心能力

- **任务****规划 (Planning)：** 内置“待办清单”管理工具，允许 Agent 将复杂长任务拆解为不同状态（进行中/已完成）的子步骤，确保持久化执行不迷失方向。
    
- **虚拟文件系统 (Virtual Filesystem)：** 充当 Agent 的“外部缓存卸载区”，通过文件读写工具处理中间结果以节省上下文空间，其读取功能原生支持多模态内容。
    
- **子智能体派生 (Subagent)：** 主 Agent 可作为“项目经理”外包任务，派生出的子 Agent 拥有独立上下文并支持并行工作，执行完毕后仅回传统一的结果报告。
    
- **上下文压缩机制：** 具备“自动瘦身”功能，当单次调用过载时会将结果转存为文件并仅保留预览（Offloading），在窗口即将溢出时则自动进行全文摘要（Summarization）。
    

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773977624155-50a07cc4-1cd4-421e-a21a-02d542491ef6.png" width="704" title="" crop="0,0,1,1" id="u6b6681aa" class="ne-image">

  

## 上下文管理

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773977430989-fef3c0af-5fb0-4d11-8045-ef13d4f4970f.png" width="558" title="" crop="0,0,1,1" id="LG7ma" class="ne-image">

  

任务执行

  

- Planning：内置 `write_todos` 工具，Agent 可以把复杂任务拆解为多个步骤，追踪状态（pending / in_progress / completed），适合长时间运行的任务。
    

上下文压缩自动触发机制：

  

- Offloading——当工具调用结果超过 20,000 token 时，自动写入 backend，在 context 里替换为文件路径和前 10 行预览，Agent 后续可按需重新读取；
    
- Summarization——当 context 超过模型窗口 85% 时，对消息历史进行 in-context summary 压缩。
    

## 子agent派生

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774233491066-154dcc3c-76ed-4f30-ac13-fd1167caa6b2.png" width="562" title="" crop="0,0,1,1" id="u16af9663" class="ne-image">

  

- 五种可插拔 Backend：`StateBackend`（默认，存于 LangGraph state，单线程内持久）；`FilesystemBackend`（本地磁盘，仅适合 CLI 和开发场景，有安全风险）；`StoreBackend`（基于 LangGraph Store，跨线程持久化）；Sandbox（Modal / Daytona / Deno 隔离环境，支持 `execute` 工具）；`CompositeBackend`（路由不同路径到不同 backend）。
    
- Virtual Filesystem：提供 `ls/read_file/write_file/edit_file/glob/grep` 工具，让 Agent 把大量中间结果卸载到存储层，避免 context 窗口溢出。`read_file` 原生支持图片，返回多模态内容块。
    
- Subagent 派生：主 Agent 通过 `task` 工具创建子 Agent，子 Agent 有独立的 context，执行完毕后返回单一结果报告。子 Agent 是无状态的（不能多轮回传），多个子 Agent 可以并行运行。
    

## Skill

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774233835030-50093685-0908-435e-b795-8f84a22afaad.png" width="548" title="" crop="0,0,1,1" id="u1e348c08" class="ne-image">

  

Skills 的核心机制是 Progressive Disclosure（渐进式披露）：启动时只读取每个 `SKILL.md` 的 frontmatter；收到 prompt 后才匹配 description；命中时才读取完整文件注入 prompt。这样避免了大量 skill 文件把 context 撑爆。`SKILL.md` 文件限制 10MB，description 字段截断至 1024 字符。

  

  

  

## Memory（双层存储）

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774233894121-e71ebb77-cb03-44ac-b4f1-b5406bad7b7f.png" width="546" title="" crop="0,0,1,1" id="u46c93791" class="ne-image">

  

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774234392460-81a37a00-442d-4366-aa4e-33763b1fc88e.png" width="558" title="" crop="0,0,1,1" id="ufe186d45" class="ne-image">

  

## prompt的加载

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774253028893-aee53209-7d8f-4a0b-98ae-ec8fe40b1c11.png" width="554" title="" crop="0,0,1,1" id="uf54238ff" class="ne-image">
#结合仓库实际结构，下面是一套快速摸清项目的梳理顺序（README.md 目前是空的，所以要以代码和目录为主）。

## 1. 先定「系统边界」

- 入口：app/main.py — FastAPI，注释写明面向「与 Qunar 聊天 LLM 的 API」；SSE、风控、工作流调用都在这里串起来。

- 依赖画像：requirements.txt 里 LangChain / LangGraph / FastAPI / OpenAI / MCP / Postgres checkpoint 等，说明是「图状态机 + LLM + 可选持久化」的后端服务。

## 2. 抓「主业务流程」一条线

- 工作流从哪进：在 app/main.py 里跟 run_agent_workflow（app/services/workflow_service）相关的调用链，看一次请求如何进图、如何流式返回。

- 图怎么拼：graph/builder.py 是总览级文件：

- 父图：START → parallel（子图）+ 条件边是否进 thinker。

- 子图 build_parallel_graph()：注册大量业务节点（接待、行程规划、交通/酒店咨询、小红书导入、国际 OTA 等），以及 START 上的条件路由（例如 _router_to_reception 决定走哪条接待/路由路径）。

把 builder.py 里 节点名 和 graph/nodes*.py 文件名 对上号，你就有了「能力地图」。

## 3. 按「三层」拆目录（比按字母扫更快）

|层次|目录|看什么|
|---|---|---|
|编排|graph/|builder.py、types.py（State 字段即全局上下文）、各 nodes_*.py（每个节点干什么）|
|能力|tools/|可调用的外部能力（搜索、爬取、航班、REPL 等）及 function_tools_register.py|
|提示词|prompts/*.md|每个场景的系统提示；文件名往往和节点/业务一一对应|
|数据与路由|store/、workflows/routing/|会话/路线等存储；独立路由子图（如 route_graph）|

## 4. 选一个「最小竖切」深读

不要先通读所有 nodes_*.py。任选一条你最关心的链路，例如：

- 普通对话 / 接待：graph/nodes_by_reception.py、trip_react_node.py、对应 prompts/reception*.md

- 行程规划：graph/nodes_by_travel_plan.py + prompts/planner.md 等

- 爬虫/正文：crawler/ + tools/crawl.py

每条链路：HTTP → workflow_service → 图入口节点 → 用到的 tools → 改写的 State 字段。

## 5. 配置与运行环境

- app/qconfig、环境变量、远程配置拉取（main.py 里可见）决定开关（例如 builder.py 里 reception v2/v3/v5、白名单等）。

- deploy_scripts/：各环境启动脚本，便于对照真实部署参数。

- conf/（若存在）：日志等本地配置。

## 6. 可选：生成「图」辅助记忆

graph/builder.py 在 import/编译时会 print Mermaid 图（draw_mermaid）。本地跑起来或单独 import 该模块，可以把打印出的 Mermaid 贴到支持 Mermaid 的编辑器里，一眼看清节点与边（注意依赖是否齐全、是否会连外部服务）。

---

一句话总结：从 app/main.py 的请求路径进 workflow_service，用 graph/builder.py 当目录索引，用 graph/types.py 的 State 当「数据契约」，用 tools/ + prompts/ 当「能力与话术」，再选一条业务竖切到底，比平铺扫 200+ 个文件快得多。

如果你说下更关心「接入层」「行程规划」还是「接待路由」，我可以按那条链路标出更具体的文件阅读顺序（仍不展开改代码）。# 

# builder.py 是干什么的？

它是 LangGraph 的「接线图」：不负责具体业务（那些在各 nodes*.py、trip_react_node.py 里），只负责：

1. 创建 StateGraph(State)

2. 注册节点（每个节点是一个函数）

3. 连边（从谁走到谁、条件分支走哪条）

4. 编译成可执行的 CompiledStateGraph

也就是说：把「行程规划 Agent」拆成很多小步骤，在这里拼成一张有向图；运行时 LangGraph 按边和条件在节点间跳。

---

## 图分两层，更好记

1. 父图 build_graph()（约 238–250 行）

- 从 START 出来两条逻辑：

- 一边进 parallel：跑子图（下面第 2 点）。

- 一边由 start_router_to_thinker 决定：若不需要思考且未 skip_thinker，就去 thinker，否则直接 END。

- 所以父图做的是：「并行子工作流」+「可选 thinker」 这一层编排。

2. 子图 build_parallel_graph()（约 125–225 行）

- 这才是 业务节点大集合：接待、意图、路线/酒店/交通规划、小红书导入、咨询、错误处理、trip_react 等，都在这里 add_node。

- _router_to_reception（96–121 行）：从 START 做 条件路由——根据配置开关、白名单、cache_state 等，决定走 route_graph / trip_react_reception / reception_by_bert / 多套 reception 等不同接待入口。你看到的 reception_v5_switch、reception_v5_whitelist 就是在选「用哪一套接待链路」。

- 同时还有 START → transcriptor，和上面条件边一起，表示 一进子图就会按图里定义的方式启动相应节点（具体并行语义以 LangGraph 对该图的组合规则为准）。

parallel_node（231–234 行）：父图里的一个节点，内部 invoke 子图，把同一份 state / config 传进去，相当于「整包并行工作流跑一遍」。

---

## 和 types.py 的关系

- builder.py 里 StateGraph(State) 的 State 来自 types.py。

- types = 状态长什么样；builder = 这些状态在哪些节点之间流转。

---

## 文件末尾的 print + draw_mermaid

227–228、249 行在 构建/导入时打印 Mermaid 图，方便在文档或工具里 可视化整张图；若你觉得启动时刷屏，一般是调试用的，可再考虑是否改成按需调用。

---

一句话：builder.py = 用 LangGraph 把行程相关所有节点注册好，并从入口（START）按配置和缓存决定走哪条接待/规划分支；具体每个节点里干什么，要去对应的 nodes / trip_react_node 文件里看。
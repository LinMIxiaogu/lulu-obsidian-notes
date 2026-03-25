# libai 主链路时序笔记

## 目标

这篇笔记回答一个问题：

当用户在前端发出一句话后，`libai` 内部到底会怎么流转？

这里重点看文本聊天主链路，也顺带标出酒店问一问这种特殊分支。

## 总体时序

可以先把主链路记成下面这条大路径：

1. 前端调用 `/aiChat/sendMessage` 或 `/chat/sendMessage`
2. `app/main.py` 做请求接入和前置校验
3. 通过 `_start_sse_stream()` 建立统一 SSE 输出通道
4. 后台启动 `run_agent_workflow()`
5. 加载历史消息、用户信息、定位、会话上下文
6. 将状态送入 `graph.astream_events()`
7. LangGraph 节点开始执行
8. `process_events()` 把图事件转成前端可消费的输出
9. `consume_output_queue()` 持续把输出队列里的内容推回前端
10. 工作流结束后统一落库、补结束事件、补推荐问题

可以理解成：

`HTTP入口 -> 预处理 -> Graph执行 -> 输出分发 -> 收尾落库`

## 第一段：HTTP 入口接入

主入口在 `app/main.py`：

- `/aiChat/hotelInquire`
- `/aiChat/sendMessage`
- `/chat/sendMessage`

对应函数是 `send_message()`。

这一段的职责不是生成答案，而是把请求变成一个“可以安全进入工作流”的输入。

### 这里做了哪些事

1. 读取请求体、cookie、IP
2. 记录监控日志
3. 判断是否命中 IP 黑名单
4. 调 `chat_check` 获取登录和用户信息
5. 判断是否属于酒店问一问
6. 做反爬检查
7. 对普通聊天继续做黑名单、风控、cookie 记录等校验
8. 最终进入 `_start_sse_stream()`

### 这里最重要的业务判断

`send_message()` 会先判断当前请求是不是酒店问一问：

- 如果是酒店问一问，某些校验会被跳过或走特殊流程
- 如果不是，则走完整的认证、风控和普通聊天流程

所以主链路从入口开始就已经在做业务分流了。

## 第二段：前置校验与请求准入

这一段仍然发生在 `send_message()` 中。

### 主要校验点

#### 1. IP 黑名单

如果请求 IP 在黑名单里，直接返回 SSE `error` 事件，不进入 Graph。

#### 2. chat_check

通过 `get_check_data()` 调 `chat_check`，拿到：

- 是否允许继续
- 用户信息 `qunarInfo`
- 错误提示文案

这是后续所有用户身份相关逻辑的起点。

#### 3. 反爬检查

通过 `http_request_service.is_anti_craw()` 判断当前请求是否允许继续。

#### 4. 用户黑名单

如果命中黑名单用户 ID，直接返回错误事件。

#### 5. 文本风控

调用 `risk_content_check()` 检查当前输入内容是否通过文本风控。

### 特殊处理：酒店问一问

如果当前请求属于 `HOTEL_INQUIRE_BIZ_SOURCE`，会出现特殊分支：

- `hotelInquireType == 1`：直接返回问题推荐
- `hotelInquireType == 2`：跳过部分认证检查，直接进入统一 SSE 流程

所以酒店问一问虽然复用了统一流式输出，但前置准入逻辑和普通聊天并不完全一样。

## 第三段：建立 SSE 输出通道

当前置校验通过后，会进入 `_start_sse_stream()`。

这一层的职责是：

- 为前端建立统一 SSE 输出
- 在后台异步执行实际工作流
- 把工作流产出的事件实时转发给前端

### 它做了什么

1. 把当前用户输入组装成 LangGraph 需要的 message 格式
2. 创建内部 `event_queue`
3. 启动后台任务 `background_workflow()`
4. 在后台任务里调用 `run_agent_workflow()`
5. 将 `run_agent_workflow()` 产生的事件放入队列
6. 外层 `event_generator()` 从队列不断取事件并 yield 给 `EventSourceResponse`

所以这里本质上是：

`Graph工作流` 和 `SSE输出` 被解耦了，中间通过一个事件队列衔接。

## 第四段：启动工作流

真正进入 AI 业务逻辑的是 `run_agent_workflow()`。

这是整个主链路最核心的总控函数。

它负责：

- 初始化当前工作流上下文
- 加载历史消息和会话状态
- 创建输出处理器
- 启动 Graph
- 管理输出事件和输出队列
- 在结尾统一做保存和收尾

## 第五段：工作流初始化

`run_agent_workflow()` 一开始会做几件关键事情。

### 1. 生成会话级 ID

包括：

- `conversation_id`
- `message_id`
- `workflow_id`

其中 `workflow_id` 是这次工作流执行的唯一标识。

### 2. 初始化输出处理器

它会创建一组 handler，例如：

- `ThinkingOutputStreamHandler`
- `OutputStreamHandler`
- `OutputUnstreamHandler`
- `AssistantAgentCardHandler`
- `DrawerChatHandler`
- `ErrorHandler`
- `ReActFinalStreamHandler`
- `ComponentOutHandler`
- `FunctionCallToolOutputHandler`
- `ModelOutputStreamHandler`
- `StreamChunkHandler`

这说明从一开始，系统就假设输出不是单一文本，而是多种事件类型混合。

### 3. 加载历史消息

普通聊天会调用：

- `get_history_messages(conversation_id, user_input_messages, c, qunar_info, biz_source)`

酒店问一问某些分支也会单独加载历史消息。

### 4. 查询 transcript context

如果没有关闭转录上下文，就会调用：

- `query_transcript_context(conversation_id)`

这一步的意义是补充多轮会话理解。

### 5. 先保存当前用户消息

在真正运行 Graph 之前，当前这条 human message 会先落库。

也就是说：

- 用户消息先入库
- 再触发 AI 工作流

## 第六段：补齐运行时状态

在把数据送入 Graph 前，`run_agent_workflow()` 还会构建很多运行状态。

### 主要内容包括

- `common_param`
- `user_location`
- `qunar_info`
- `messages`
- `current_messages`
- `history_message_converters`
- `transcript_context`
- `route_state`
- `hotel_state`
- `transport_state`
- `cache_state`

### 特别值得注意的两点

#### 1. 会查用户定位

调用 `http_request_service.get_location_info()`，根据 `c` 参数里的经纬度补出用户位置。

这意味着后面很多业务模块都能用到：

- 用户所在城市
- 用户国家
- 用户详细地址

#### 2. 会尝试读取路线缓存

如果开启 `route_state` 缓存能力，会先查缓存中的路线状态。

这会影响后面 Graph 入口是走：

- 正常 reception 路由
- 还是直接走 `route_graph`

所以 `libai` 并不是每次都从零开始理解用户，而是会尝试复用上次路线状态。

## 第七段：进入 Graph

准备好状态后，会调用：

- `graph.astream_events(...)`

这里的 `graph` 来自 `build_graph()`。

### Graph 的入口结构

最外层图很简单，只有两个起点分支：

- `parallel`
- `thinker`

其中：

- `parallel` 负责真正业务工作流
- `thinker` 负责“思考流”输出

`start_router_to_thinker()` 决定是否跳过 thinker。

所以从最外层看，主链路是：

- 一条业务流
- 一条思考流

并行运行。

## 第八段：业务流起点怎么分流

`parallel` 内部会调用 `build_parallel_graph()`。

这里真正决定“用户这句话会进哪个业务模块”的是：

- `_router_to_reception()`

### 它可能把请求导向哪些入口

- `trip_react_reception`
- `route_graph`
- `reception`
- `reception_by_domain`
- `reception_by_sector`
- `reception_by_bert`

### 这层的本质

这一层做的是“入口路由策略选择”，也就是决定：

- 用旧版 reception
- 用新版 trip_react reception
- 用 domain / sector / bert 并行 reception
- 还是直接走缓存路线图

所以并不是每个请求都走同一套 reception 逻辑，而是按配置和用户白名单动态选择入口版本。

## 第九段：transcriptor 会并行启动

在 `build_parallel_graph()` 里有一条很关键的边：

- `START -> transcriptor`

也就是说，除了 reception 分流之外，`transcriptor` 会并行启动。

它的职责是：

- 压缩和分类用户意图
- 维护会话级 transcript context

这说明系统会一边做业务分流，一边做会话语义抽取。

## 第十段：进入具体业务模块

经过 reception 之后，用户请求会跳转到某个具体业务模块。

常见去向包括：

- `chatbot`
- `travel_plan_trigger`
- `transport_consult_sector`
- `hotel_consult_sector`
- `adjust_route_sector`
- `travel_xhs_import_sector`
- `hotel_inquire`
- `trip_react`

### 可以把这些跳转理解成

#### 1. 普通问答

走 `chatbot`

#### 2. 行程规划

走 `travel_plan_trigger -> travel_route_planner -> travel_transport_planner -> travel_hotel_planner -> travel_weather_planner -> travel_planner`

#### 3. 交通咨询

走 `transport_consult_sector -> by_list -> by_recommend`

#### 4. 酒店咨询

走 `hotel_consult_sector -> by_list -> by_recommend`

#### 5. 路线调整

走 `adjust_route_sector`

#### 6. 小红书导入

走 `travel_xhs_import_sector -> route_planner -> transport/hotel/weather -> planner`

#### 7. 酒店问一问

走 `hotel_inquire`

所以 Graph 运行的实质，就是把请求送入一条条明确的业务场景链路。

## 第十一段：Graph 事件如何变成前端输出

Graph 执行过程中，`run_agent_workflow()` 不会直接把节点结果原样返回给前端。

中间会经过：

- `process_events()`

它会消费 `graph.astream_events()` 产生的 LangGraph 事件，并根据事件类型交给不同 handler。

### 它做的关键事

#### 1. 记录节点执行轨迹

通过 `_track_node_execution()` 等逻辑，记录哪个节点跑过。

#### 2. 按输出类型分发

例如：

- `thinking`
- `stream`
- `unstream`
- `drawer_chat`
- `error_handler`
- `agent_card`
- `component_out`
- `react_final`
- `trip_function_call_output`
- `model_output_stream`
- `stream_chunk`

#### 3. 把输出写入输出队列

真正给前端看的结果，大多不是在节点里直接返回，而是被加工后放进 `output_queue_service`。

所以 `process_events()` 是“图事件 -> 前端事件”的翻译层。

## 第十二段：输出队列如何推给前端

除了 `process_events()`，还会并行跑：

- `consume_output_queue()`

它会不断从 `output_queue_service` 拉取已经准备好的输出，并产出给外层。

然后 `run_agent_workflow()` 会同时启动两个并行任务：

- 一个跑事件处理
- 一个跑输出队列消费

这两个任务共同把结果塞进统一 `output_queue`。

主循环再从 `output_queue` 中取出结果，控制节奏后 `yield` 给上层 `_start_sse_stream()`。

所以从执行机制上看，主链路是：

`LangGraph事件流 + 输出队列流 -> 合并 -> SSE输出`

## 第十三段：前端看到的输出类型

前端最终看到的不是单一答案，而可能包括：

- 思考内容
- 普通文本流
- 非流式文本
- Agent 卡片
- Trip Card
- Drawer Chat
- 组件化内容
- 错误事件
- Stream start / chunk / end
- 会话结束事件

这也是为什么 `libai` 前端体验更像“过程型交互”，而不是一次性返回整块文本。

## 第十四段：工作流结束后的收尾

当 Graph 执行和输出消费结束后，`run_agent_workflow()` 还会做统一收尾。

### 主要包括

#### 1. 追加会话结束事件

会生成 `CONVERSATION_ENDING` 事件，并发送给前端。

#### 2. 统一保存消息

通过：

- `get_save_messages_and_clear()`
- `collect_and_save_messages()`

把这次流程中的消息统一顺序落库。

#### 3. 处理 reception 结果

通过 `_handle_reception_goto_result()` 处理本轮 reception 路由结果。

#### 4. 构建通用输出结果

通过 `common_output_service.build_common_output_results()` 生成：

- 数据库存储结果
- 前端额外输出结果

#### 5. 生成问题推荐

最后才会调用：

- `handle_question_recommend()`

或酒店问一问场景下的：

- `handle_hotel_inquire_question_recommend()`

也就是说，推荐问题是在主答案处理完成后追加的，不是主流程中途生成的。

## 第十五段：特殊链路和普通链路的差别

### 普通聊天链路

典型流程是：

`send_message -> 校验 -> _start_sse_stream -> run_agent_workflow -> reception分流 -> 业务节点 -> 输出 -> 落库 -> 推荐问题`

### 酒店问一问链路

酒店问一问会有几个不同点：

- 前置校验不完全一样
- 可能直接返回推荐问题
- 可能跳过部分普通聊天流程
- `hotel_inquire` 节点内部是对外部 SSE 的再封装

所以它虽然复用了统一框架，但业务上是一条特殊纵向链路。

## 第十六段：从源码阅读角度怎么抓主链路

如果下次你要继续读这条链路，建议按这个顺序：

1. `app/main.py`
2. `_start_sse_stream()`
3. `app/services/workflow_service.py` 里的 `run_agent_workflow()`
4. `graph/builder.py`
5. `graph/nodes.py` 和各 `nodes_by_*`
6. `app/services/output_service/*`
7. `store/*`

这样读会比直接从 Graph 里乱跳更容易建立整体感。

## 当前结论

`libai` 的主链路并不是“请求进来 -> LLM 回答 -> 返回结果”这么简单。

更准确地说，它是一条分层流水线：

- 入口负责校验和准入
- 工作流负责状态装载和 Graph 调度
- Graph 决定具体业务模块
- 输出层把内部事件翻译成前端交互流
- 收尾阶段负责落库、结束事件和问题推荐

所以后续理解 `libai`，最好一直带着“事件驱动工作流”这个视角，而不是传统 MVC 接口视角。

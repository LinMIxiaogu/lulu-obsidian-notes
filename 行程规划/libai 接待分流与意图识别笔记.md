# libai 接待分流与意图识别笔记

## 这篇笔记解决什么问题

这篇笔记不是只讲“意图识别”四个字，而是回答一个更完整的问题：

> 一个用户问题进入 `libai` 以后，到底是怎么一步步变成最终输出的？

如果把 `libai` 当成一个面向旅游场景的 AI 系统，那么它并不是“一个大模型直接回答所有问题”，而是一个分层系统：

1. API 接口层先接住请求
2. 入口层做鉴权、风控、黑名单、代理、SSE 建链
3. 工作流层组装上下文、历史消息、用户信息
4. 接待分流层判断这到底是什么问题
5. 具体业务节点执行酒店、交通、行程规划、导入、小问答等能力
6. 输出层把过程事件、卡片、文本、推荐问题统一包装后流式返回前端

所以，“接待分流与意图识别”不是孤立的一层，而是整个产品主链路里的中枢路由层。

---

## 先用一句话理解这个产品

`libai` 本质上是一个旅游场景的多能力 AI 编排系统。

它面向的不是单一问答，而是多种旅游相关场景，例如：

- 普通聊天问答
- 行程规划
- 路线生成
- 路线调整
- 酒店搜索与推荐
- 酒店对比
- 酒店“问一问”
- 交通查询与推荐
- 低价机票/价格趋势/翻页等细分交通能力
- 小红书笔记导入并转成行程
- 图片输入后的视觉聊天
- 抽屉式二级交互
- 问题推荐、联想问题

从代码结构看，它不是一个“单 Agent”，而是：

- `app/main.py` 提供 HTTP/SSE 接口
- `app/services/workflow_service.py` 负责把请求送进工作流
- `graph/builder.py` 定义 LangGraph 主图
- `graph/nodes*.py` 和 `graph/node*.py` 是各类业务节点
- `app/services/output_service/*` 负责把图执行事件转成前端能消费的输出

---

## 一张总图：从用户进入到输出

可以把主链路理解成下面这条线：

`用户请求 -> FastAPI 接口 -> 请求校验/鉴权/风控 -> run_agent_workflow -> LangGraph 分流 -> 业务节点执行 -> 输出事件整理 -> SSE 返回前端`

再展开一点：

`/chat/sendMessage`
-> `app/main.py::send_message`
-> 校验 cookie / 用户 / 黑名单 / 风控 / 是否代理
-> `_start_sse_stream`
-> `run_agent_workflow`
-> `build_graph().astream_events(...)`
-> `parallel + thinker`
-> `reception / reception_by_* / trip_react_reception / route_graph / 各业务节点`
-> `process_events + output_queue`
-> `conversation_ending + 附加卡片 + 问题推荐`
-> SSE 流式回前端

这里最重要的认识是：

- 前半段解决“这条请求能不能进系统”
- 中间段解决“这条请求该去哪个业务模块”
- 后半段解决“结果怎么按前端需要的格式发出去”

---

## 第一层：HTTP 入口层做了什么

主入口在 [app/main.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/app/main.py)。

核心接口是：

- `/aiChat/sendMessage`
- `/chat/sendMessage`
- `/aiChat/hotelInquire`

这说明从产品视角看，至少有两类主入口：

- 通用 AI 聊天入口
- 酒店问一问入口

### 1. 请求进来后先做基础校验

`send_message()` 不是一上来就调模型，而是先做外围治理：

- 读取 cookie，例如 `_s`
- 获取用户 IP
- 判断 IP 黑名单
- 调用 `chat_check` 做登录/用户校验
- 做反爬校验
- 做文本风控 `risk_content_check`
- 特定环境下可能走代理转发

这一步的意义是：

- 不是所有请求都能进 AI 主链路
- 系统先保证“请求合法、用户合法、内容不过线”

### 2. 酒店问一问是特殊入口

代码里对 `invokeSource in HOTEL_INQUIRE_BIZ_SOURCE` 做了单独处理。

也就是说，酒店问一问不是普通聊天的一个 prompt 分支，而是一个单独的业务入口，拥有自己的处理逻辑，包括：

- 可直接绕过部分通用鉴权检查
- 可走问题推荐分支
- 可进入专门的 `hotel_inquire` 节点

### 3. 所有正常请求最终都走 SSE 流

入口层最后会进入 `_start_sse_stream()`：

- 先把用户输入转成 LangGraph 使用的 `messages`
- 创建事件队列 `event_queue`
- 后台启动 `run_agent_workflow(...)`
- 一边执行工作流，一边把事件持续放进队列
- 前端通过 `EventSourceResponse` 持续收到消息

所以这个系统从产品表现上看，是“流式聊天”，从架构上看，是“后台工作流持续产出事件，前台持续消费事件”。

---

## 第二层：工作流层在进入图之前做了什么

核心在 [app/services/workflow_service.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/app/services/workflow_service.py)。

这层的职责不是分流本身，而是把“请求”变成“图可以运行的状态”。

### 1. 组装会话上下文

它会把这些信息整理成状态：

- `conversation_id`
- `message_id`
- `workflow_id`
- 当前用户输入
- 历史消息
- 历史消息转换器
- `extMap`
- `c` 参数和解析后的 `common_param`
- 用户地理位置
- qunar 用户信息
- transcript 上下文
- route 缓存状态

这些最终会被放进 LangGraph 的 `State`。

这意味着：

- 后续节点做判断时，不只是看这一句用户输入
- 还会看历史对话、客户端传参、地理位置、用户身份、上文缓存

### 2. 存储用户消息

在真正进入图之前，它会先保存一条 human message。

所以这个系统有很明显的“会话消息落库”能力，而不是只做内存态回答。

### 3. 可能使用 RouteState 缓存

它会尝试根据首轮输入生成 route cache key，例如某些固定模式的规划问题。

如果命中缓存，就可以跳过部分前置识别，直接使用已有 route state。

这说明：

- `libai` 不是每次都从零开始
- 某些规划类问题有缓存加速机制

### 4. thinker 不一定每次都跑

`skip_thinker()` 会在某些情况下跳过 thinker，例如：

- 点击标签进入
- 某些固定首轮触发语
- 酒店问一问
- 路由状态来自缓存

所以“思考流”只是可选层，不是所有请求的必经层。

---

## 第三层：LangGraph 主图到底长什么样

主图定义在 [graph/builder.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/builder.py)。

这个图最关键的设计是：

- 一条是 `parallel`
- 一条是 `thinker`

也就是一进图，系统会同时考虑两件事：

1. 真正的业务分流和执行
2. 是否给前端展示“思考中”的流式内容

### 1. `parallel` 里装的才是主业务流

`parallel` 子图里包含了几乎所有业务节点：

- `reception`
- `reception_by_domain`
- `reception_by_sector`
- `reception_by_bert`
- `reception_finally`
- `trip_react_reception`
- `trip_react`
- `route_graph`
- `travel_intent_sector`
- `travel_plan_trigger`
- `hotel_consult_sector`
- `transport_consult_sector`
- `hotel_inquire`
- `chatbot`
- `chatbot_vl`
- `drawer_chat`
- `travel_xhs_import_*`
- `adjust_route_sector`
- `question_association`

### 2. 主图一开始先决定走哪套接待策略

`_router_to_reception(state)` 会决定从哪一套入口进：

- `route_graph`
- `trip_react_reception`
- `reception`
- `reception_by_domain + reception_by_sector + reception_by_bert`

这取决于：

- 配置开关
- 白名单
- 是否已有 route cache

所以接待层本身也是多版本并存的，不是只有一套逻辑。

---

## 第四层：接待分流层到底在做什么

这是这篇笔记最核心的部分。

“接待分流”不是回答问题，而是判断：

> 这句话应该被送到哪个业务模块处理？

### 可以先把它理解成“分诊台”

用户说一句话，系统先不急着答，而是先判断：

- 这是酒店问题吗？
- 这是交通问题吗？
- 这是路线规划吗？
- 这是路线调整吗？
- 这是导入路线吗？
- 这是普通闲聊吗？
- 这是图片问答吗？
- 这是酒店问一问吗？

只有路由对了，后面的回答才可能对。

---

## 第五层：最重要的快速直达规则 `pre_router_sector`

核心在 [graph/nodes_by_reception.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/nodes_by_reception.py)。

`pre_router_sector(state)` 是整个接待层里最重要的一段逻辑。

它的原则是：

> 只要能明确判断，就不要再浪费一次模型分类。

也就是说，它优先处理“强信号场景”。

### 它会直接命中的场景

#### 1. 酒店问一问

如果 `invokeSource` 属于酒店问一问来源：

- 直接 `goto = hotel_inquire`

这是一个非常强的业务直达规则。

#### 2. 前端显式指定了下一个节点

如果 `extMap` 里带有公共事件标记：

- 直接跳到指定节点

这说明前端不是只能“发一句自然语言”，还可以显式控制后续流程。

#### 3. 图片输入

如果请求带图片：

- 直接去 `chatbot_vl`

也就是视觉聊天。

#### 4. 抽屉式聊天

如果 `extMap.transportType` 不为空：

- 进入 `drawer_chat`

这类通常不是普通自由聊天，而是某个 UI 操作带起的二级交互。

#### 5. 行程规划显式触发

如果 `extMap.planTrigger == true`：

- 开启 route workflow 时走 `route_graph`
- 否则走 `travel_plan_trigger`

意思是：前端已经明确告诉后端，“这不是普通聊天，这是要开始规划”。

#### 6. 补参数场景

如果 `extMap.skipFlag == true`：

- 开启 route workflow 时走 `route_graph`
- 否则走 `travel_intent_sector`

这通常表示上一轮系统问了补充问题，这一轮用户是在补参数。

#### 7. 小红书导入首轮

命中特定 XHS 首轮内容时：

- `goto = travel_xhs_import_repair`

#### 8. 高分酒店首轮引导

命中特定首轮触发语时：

- `goto = high_score_hotel_repair`

#### 9. 低价交通首轮引导

命中特定首轮触发语时：

- `goto = low_price_transport_repair`

#### 10. 自然语言调整路线

如果 `extMap.llm_adjust_route_flag == true`：

- 开启 route workflow 时走 `route_graph`
- 否则走 `adjust_route_sector`

#### 11. AI 语音生成路线

如果 `extMap.ai_voice_generate_route == true`：

- 开启 route workflow 时走 `route_graph`
- 否则走 `travel_plan_trigger`

### 这层的意义

`pre_router_sector` 本质上是在处理：

- 前端显式控制
- 明显业务特征
- 固定产品入口
- 上一轮追问后的续接

所以它像是一个“业务高速路入口”。

能在这里直接判断的请求，没必要再让 LLM/BERT 分一次。

---

## 第六层：接待分流不是一套，而是三套

从当前代码看，`libai` 的接待分流至少有三种模式。

### 模式 A：老版 `reception`

定义在 [graph/nodes.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/nodes.py) 的 `reception_node()`。

流程很简单：

1. 先跑 `pre_router_sector`
2. 命不中再用 `reception` prompt + LLM 分类
3. 得到 `goto`
4. 直接跳到具体业务节点

它适合简单直接的场景，但可控性相对弱。

### 模式 B：分层版 `reception_by_*`

也就是：

- `reception_by_domain`
- `reception_by_sector`
- `reception_by_bert`
- `reception_finally`

这是一套更典型的企业级分类链路。

它不是一步定终局，而是：

1. 先分大类 domain
2. 再分细类 sector
3. BERT 作为辅助分类
4. 最后统一裁决

### 模式 C：新版 `trip_react_reception`

定义在 [graph/trip_react_node.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/trip_react_node.py)。

这套逻辑更偏“统一入口 + 工具编排”：

1. 先尝试 `pre_router_sector`
2. 首轮优先做关键词路由
3. 再尝试 BERT v2 分类
4. 最后才用 LLM 分类
5. 得到的不是具体节点名，而是更高层的业务领域
6. 后续交给 `trip_react` 决定用哪些工具处理

它更像“智能调度入口”，而不是传统节点跳转入口。

---

## 第七层：分层版接待链路怎么理解

这部分对应 [graph/nodes_by_reception.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/nodes_by_reception.py)。

### 1. `reception_by_domain` 先分大类

它先判断用户属于哪个大领域，例如：

- 旅游规划类
- 交通类
- 酒店类
- 普通聊天类

它的处理顺序是：

1. 酒店问一问直接去 `hotel_inquire`
2. 先尝试 `pre_router_sector`
3. `INTER_OTA` 特殊处理
4. 最后才用 LLM 做 domain 分类

这一步的价值是把问题先放进“大盘子”。

### 2. `reception_by_sector` 再分细类

大类确定后，再从对应候选 sector 里选一个具体业务。

它不是在全量空间里乱猜，而是：

1. 根据 `invokeSource` 先拿 domain 列表
2. 再从配置里拿这个 domain 下允许的 sector 列表
3. 用 prompt + 候选列表让 LLM 选

这比“开放式分类”稳定得多。

### 3. `reception_by_bert` 是辅助裁判

它不是最终业务节点，而是给出一个额外分类结果。

它的价值在于：

- 更快
- 更便宜
- 某些场景更稳

所以它常常是 LLM 分类的辅助参照。

### 4. `reception_finally` 负责最终裁决

它会整合三类结果：

- domain_result
- sector_result
- bert_result

然后做这些判断：

- 如果 `pre_router_sector` 已经能直达，优先直达
- 某些配置下直接采用 BERT
- 如果 domain 或 sector 为空，走默认兜底
- 如果 domain 与 sector 反推的大类一致，说明分类自洽，直接使用
- 如果不一致，就带着 domain 约束重新跑一次 `reception_by_sector`

所以它其实是一个“分类仲裁器”。

它防止出现这种情况：

- 一级分类说是酒店
- 二级分类却跳到了交通

---

## 第八层：`trip_react_reception` 和传统分流最大的区别

传统 `reception` / `reception_by_*` 的输出更像：

- `hotel_consult_sector`
- `transport_consult_sector`
- `travel_plan_trigger`

也就是直接给出“该跳哪个业务节点”。

而 `trip_react_reception` 的输出更像：

- `route`
- `transportation`
- `hotel`
- `combined`
- `other_clear`
- `ambiguous`

也就是说，它先给出“业务领域”，然后再由 `trip_react` 结合工具 schema、prompt、函数调用结果继续往下做。

所以它本质上更像：

- 一个领域识别器
- 一个工具编排入口
- 一个 ReAct 调度器前置路由

---

## 第九层：分流后会落到哪些具体业务能力

这一部分是给完全不了解产品的人最重要的内容。

从代码上看，`libai` 支持的主要能力如下。

### 1. 普通聊天问答

节点：

- `chatbot`

能力特点：

- 普通自然语言问答
- 可接知识库检索结果
- 是通用兜底能力

### 2. 图片问答 / 视觉聊天

节点：

- `chatbot_vl`

触发方式：

- 用户请求里带 `images`

### 3. 行程规划

相关节点：

- `travel_plan_trigger`
- `travel_intent_inquiry_sector`
- `travel_intent_sector`
- `travel_route_planner`
- `travel_hotel_planner`
- `travel_transport_planner`
- `travel_weather_planner`
- `travel_planner`

能力特点：

- 抽取出发地、目的地、日期、天数等旅行意图
- 先追问补参数，再正式规划
- 可以生成路线、酒店、交通、天气等组合结果
- 最后形成完整行程输出

### 4. Route Workflow 版路线规划

相关节点：

- `route_graph`

定义在：

- [workflows/routing/route/route_graph.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/workflows/routing/route/route_graph.py)

特点：

- 这是另一套更结构化的路线工作流
- 它不是直接让某个节点一次做完，而是按 pipeline 走：
  - `build_param`
  - `select_processor`
  - `get_route_ref`
  - `route_ref_to_json`
  - `standardization`
  - `summary`
  - `after_summary`
- 支持 fallback 处理器
- 支持缓存 route state

所以“行程规划”在 `libai` 里其实有两种实现路线：

- 传统 planner 节点路线
- route workflow 结构化流水线

### 5. 旅行意图识别与追问

相关节点：

- `travel_intent_inquiry_sector`
- `travel_intent_sector`

能力特点：

- 判断信息够不够
- 不够则反问用户
- 够了以后抽取旅行意图
- 再进入规划或路线生成

### 6. 酒店搜索与推荐

相关节点：

- `hotel_consult_sector`
- `hotel_consult_sector_by_list`
- `inter_hotel_consult_sector_by_list`
- `hotel_consult_sector_by_recommend`

能力特点：

- 先用 LLM 抽取酒店搜索参数
- 参数不够时追问
- 支持国内酒店与国际酒店两套逻辑
- 先查酒店列表
- 再从列表中推荐更合适的酒店

### 7. 高分酒店首轮引导

节点：

- `high_score_hotel_repair`

它更像一个产品化入口提示，不是完整酒店查询本身。

### 8. 酒店对比

节点：

- `hotel_comparison`

能力特点：

- 根据 `ext_map` 中的对比参数获取酒店对比列表
- 流式生成对比文案
- 支持替换式输出

### 9. 酒店问一问

节点：

- `hotel_inquire`

定义在：

- [graph/node_by_hotel_inquire.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/node_by_hotel_inquire.py)

能力特点：

- 不是普通酒店搜索
- 是调用外部酒店咨询接口
- 自己处理流式 chunk
- 把外部流式结果再转发成系统内部的流式输出事件

这是一个非常典型的“垂直业务专线”。

### 10. 交通咨询

相关节点：

- `transport_consult_sector`
- `transport_consult_sector_by_list`
- `transport_consult_sector_by_recommend`

能力特点：

- 抽取出发地、目的地、日期、交通类型
- 参数不够先追问
- 查交通列表
- 再从列表里做推荐
- 同时支持航班和火车

### 11. 航班精搜和低价相关能力

相关节点：

- `flight_precise_search`
- `flight_lowprice_destination`
- `flight_price_trend`
- `flight_bug_price`
- `flight_turn_page`
- `transport_prompt_for_details`

这说明交通模块不只是“查票”，还包括：

- 精确航班搜索
- 低价目的地
- 价格趋势
- bug 价
- 翻页查看更多结果
- 便宜去哪类提示与详情

### 12. 路线调整

节点：

- `adjust_route_sector`

触发方式：

- 自然语言调路线
- `extMap.llm_adjust_route_flag`

### 13. 小红书笔记导入

相关节点：

- `travel_xhs_import_repair`
- `travel_xhs_import_sector`
- `travel_xhs_import_route_planner`
- `travel_xhs_import_transport_planner`
- `travel_xhs_import_hotel_planner`
- `travel_xhs_import_weather_planner`
- `travel_xhs_import_planner`

能力特点：

- 不是普通聊天
- 是把外部笔记内容导入后转成旅行规划流程

### 14. 抽屉式聊天

节点：

- `drawer_chat`

它更像某个 UI 子面板里的专属交互，不是主聊天入口。

### 15. 问题推荐与联想问题

相关节点：

- `question_recommend_groups`
- `question_association`

以及工作流结束后的：

- `handle_question_recommend`
- `handle_hotel_inquire_question_recommend`

也就是说，系统最终输出不只有“回答内容”，还可能附带：

- 推荐追问
- 相关问题
- 标签分组问题

---

## 第十层：为什么说“输出”也是一层，而不只是 return 文本

这是很多新人一开始最容易忽略的地方。

在 `libai` 里，业务节点产出的并不总是最终给前端的字符串。

真正的输出整理是在 [app/services/workflow_service.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/app/services/workflow_service.py) 的 `process_events()` 和后续队列消费逻辑里完成的。

### 它会把图中的事件分成很多类

例如：

- thinker 流式输出
- 普通文本流式输出
- 非流式文本输出
- 抽屉聊天输出
- 错误输出
- Agent 卡片输出
- function call 工具输出
- 模型 chunk 输出
- component 化输出

对应的 handler 包括：

- `ThinkingOutputStreamHandler`
- `OutputStreamHandler`
- `OutputUnstreamHandler`
- `DrawerChatHandler`
- `ErrorHandler`
- `AssistantAgentCardHandler`
- `FunctionCallToolOutputHandler`
- `ModelOutputStreamHandler`
- `ComponentOutHandler`
- `StreamChunkHandler`

所以从工程实现上看：

- 图节点只负责产生业务结果或中间事件
- 输出层负责决定这些东西怎么变成前端能消费的 event/data

### 最终返回给前端的并不只有文本

根据 `graph/types.py` 里的 `EventType`，前端能接收到的事件包括：

- `thinking`
- `message`
- `assistantAgentCard`
- `error`
- `question_recommend`
- `hotel_consult_sector_by_recommend`
- `transport_consult_sector_by_recommend`
- `transport_consult_sector_by_list`
- `flight_precise_search`
- `flight_bug_price`
- `markdown`
- `trip_card`
- `hotel_card`
- `conversation_ending`

所以这个系统的输出天然就是“多模态前端事件流”，不是一个简单字符串。

---

## 第十一层：一次请求结束时，还会做哪些收尾动作

在 `run_agent_workflow()` 的尾部，还会做几件很关键的事情。

### 1. 发送 `conversation_ending`

表示这轮会话工作流完成。

### 2. 统一保存消息

包括：

- 用户消息
- 过程消息
- 最终结果

### 3. 构造通用附加输出

当前看 [graph/service/common_output_service.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/service/common_output_service.py)，至少会尝试补：

- 行程卡片引用笔记等附加结果

### 4. 追加问题推荐

普通链路和酒店问一问链路，会分别走自己的推荐问题逻辑。

这说明最终一个请求的完整输出可能包含：

1. 中间流式文本
2. 卡片/组件
3. 收尾事件
4. 附加卡片
5. 推荐问题

---

## 第十二层：从产品视角看，用户进入到输出的关系可以怎么理解

如果从产品经理或新人视角，可以把它理解成 5 个阶段。

### 阶段 1：请求接入

用户在前端输入一句话，或者点一个标签、传一组参数、上传一张图。

### 阶段 2：入口治理

系统判断：

- 请求是否合法
- 用户是否合法
- 是否命中黑名单/风控
- 是否属于特殊业务入口

### 阶段 3：接待分流

系统判断：

- 这是不是酒店问一问
- 这是不是酒店搜索
- 这是不是交通咨询
- 这是不是行程规划
- 这是不是路线调整
- 这是不是导入路线
- 这是不是图片问答
- 如果都不是，就当普通问答

### 阶段 4：业务执行

对应业务节点去调用：

- LLM
- 外部酒店/交通/路线服务
- route workflow processor
- function calling tools
- 知识库检索

### 阶段 5：输出编排

系统把结果整理成：

- 流式文本
- 卡片
- 组件
- 推荐问题
- 收尾事件

再通过 SSE 连续推给前端。

---

## 第十三层：为什么这套设计对新人很重要

因为如果只盯着某个 prompt 或某个节点，很容易误解这个系统。

`libai` 的关键不在于“某个模型怎么回答”，而在于：

- 入口是否治理住了
- 分流是否分对了
- 业务节点是否命中了正确能力
- 输出是否被正确包装给前端

换句话说，真正决定用户体验的，往往不是某个单点模型效果，而是：

> 用户有没有被送进正确的业务链路。

这也是为什么“接待分流与意图识别”是整个系统里非常核心的一层。

---

## 最后给一个新人版总结

如果你完全不了解这个产品，可以先记住下面这段话：

`libai` 不是一个单纯的大模型聊天机器人，而是一个面向旅游场景的 AI 编排系统。用户请求先经过接口层的校验和风控，再进入工作流。工作流先结合历史消息、用户信息和前端参数建立上下文，然后由接待分流层判断请求属于哪类业务。能被规则直接命中的请求会直接进入对应模块，不能命中的再通过 BERT、LLM 或 ReAct 入口做分类。分流完成后，请求会进入酒店、交通、行程规划、路线调整、小红书导入、视觉聊天、酒店问一问等具体业务节点执行。最后，系统把文本、卡片、组件和推荐问题统一包装成 SSE 事件流返回前端。`

如果后面继续读代码，建议优先按下面顺序看：

1. [app/main.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/app/main.py)
2. [app/services/workflow_service.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/app/services/workflow_service.py)
3. [graph/builder.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/builder.py)
4. [graph/nodes_by_reception.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/nodes_by_reception.py)
5. [graph/trip_react_node.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/trip_react_node.py)
6. [graph/nodes.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/graph/nodes.py)
7. [workflows/routing/route/route_graph.py](D:/Users/lulut.tang/Desktop/study/行程规划/libai/workflows/routing/route/route_graph.py)

这样看，会比一上来钻某个业务节点清楚很多。

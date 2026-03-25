# libai 业务模块总结

## 总体判断

从业务逻辑角度看，`libai` 不是一个“统一聊天机器人”，而是一套围绕旅游场景拆分好的 AI 业务中枢。

它做的核心事情可以概括成三步：

1. 接住用户自然语言输入
2. 判断当前属于哪类旅游场景
3. 调用对应业务模块生成结果，并以流式形式返回

所以更适合把它理解成“旅游场景 AI 分发与编排系统”，而不是单一聊天服务。

## 业务主链路

用户请求进入 `app/main.py` 后，主流程大致如下：

1. 接收聊天请求 `/aiChat/sendMessage` 或 `/chat/sendMessage`
2. 做用户校验、风控、反爬、cookie 和用户信息处理
3. 进入 `workflow_service`
4. 由 `graph/builder.py` 组织的 LangGraph 工作流开始运行
5. 先做接待分流 / 意图识别
6. 跳转到具体业务模块
7. 输出文本流、卡片流、组件流或结构化事件

这意味着 `libai` 的第一职责不是“回答问题”，而是“把问题分给正确的业务模块”。

## 核心业务分层

从业务视角，`libai` 大致可以拆成下面几层。

### 1. 接待分流层

这一层的目标是回答一个问题：

当前用户是在问什么类型的旅游问题？

对应的核心节点包括：

- `reception`
- `trip_react_reception`
- `reception_by_domain`
- `reception_by_sector`
- `reception_by_bert`
- `pre_router_sector`

这一层会综合使用：

- 规则预判
- 关键词路由
- BERT 分类
- LLM 分类

最终把用户问题归入类似这些场景：

- 普通聊天
- 路线规划
- 行程调整
- 交通咨询
- 酒店咨询
- 酒店问一问
- 小红书导入路线
- 图文/视觉类能力

所以 `reception` 本质上就是业务分诊台。

## 业务模块一：通用聊天与兜底问答

当用户问题不进入明确的旅游业务链路时，会落到 `chatbot`。

这个模块的职责是：

- 处理泛问答
- 使用旅游客服知识
- 在没有复杂业务调用时直接给出文本回复

它不是主要的“路线规划器”，更像旅游场景下的基础问答层和兜底回复层。

从代码看，这一层还支持：

- 检索 chatbot 知识库
- 用流式方式输出文本
- 作为业务链路失败时的保底交互

## 业务模块二：行程规划主链路

这是 `libai` 最核心的业务模块。

对应节点主要包括：

- `travel_plan_trigger`
- `travel_route_planner`
- `travel_hotel_planner`
- `travel_transport_planner`
- `travel_weather_planner`
- `travel_planner`
- `after_route_out`

### 它在业务上负责什么

当用户说“帮我规划一个行程”时，这条链路会：

- 提取用户的出发地、目的地、时间、天数、偏好
- 先做路线推荐或路线生成
- 再补交通推荐
- 再补酒店推荐
- 再补天气信息
- 最后把这些结果汇总成可展示的行程结果

### 业务上的特点

这条链路不是只产出一段文字，而是会同时处理：

- 路线文本
- 路线卡片
- 酒店卡片
- 交通卡片
- 后处理内容

`after_route_out` 还会继续追加一些后置动作，例如：

- 生成标准化路线结果
- 追加外部跳转内容
- 补充参考内容
- 统一前端组件输出

所以“行程规划”在 `libai` 里并不是一个节点，而是一条完整流水线。

## 业务模块三：路线生成与路线资产处理

`route_service.py` 是路线能力的核心服务层。

从函数名看，它承担了几类重要职责：

- 推荐路线获取：`get_recommend_route_list`
- 用户导入路线：`get_user_import_route`
- 路线生成：`generate_route`
- 基于小红书笔记生成路线：`generate_route_by_xhs_note`
- 基于 embedding 笔记生成路线：`generate_route_use_embedding_xhs_note`
- 生成并保存路线：`generate_route_and_save_route`
- 路线风格生成：`generate_route_style_list`
- 行程标准化：`travel_plan_standardization`

这说明 `libai` 自己并不只是“把 dufu 返回什么就原样输出”，而是对路线结果做了明显的二次加工与生成。

更准确地说，路线模块负责：

- 路线召回
- 路线文本生成
- 路线结构化
- 路线落库前处理
- 路线展示后处理

## 业务模块四：行程调整

这部分对应 `adjust_route_sector`。

这是一个非常典型的“已有结果二次编辑”模块。

### 它解决的问题

用户不是第一次创建行程，而是在已有路线基础上提出修改，例如：

- 加一个景点
- 换一个酒店
- 调整某一天顺序
- 删除某个点位
- 改整体目的地或行程安排

### 当前实现特点

代码里能看到两套模式：

- 智能调整模式
- 传统 tool 调用模式

其中传统模式已经显式定义了 function tools，例如：

- `add_poi_item`
- `replace_poi_item`
- `move_poi_item`
- `delete_poi_item`
- `change_travel`

智能模式则会：

- 先取已有路线详情
- 调用 `intelligent_adjust_route`
- 做 POI 挂靠和结果修正
- 调用 route store 存回结果
- 输出新的路线内容和行程卡片

所以这部分本质上是“AI 驱动的路线编辑器”。

## 业务模块五：交通咨询

这一块对应：

- `transport_consult_sector`
- `transport_consult_sector_by_list`
- `transport_consult_sector_by_recommend`
- `flight_precise_search`
- `flight_price_trend`
- `flight_turn_page`
- `low_price_transport_repair`

### 它在业务上的工作方式

交通模块不是直接查票，而是先用 LLM 把自然语言解析成交通搜索参数，例如：

- 出发城市
- 到达城市
- 出发日期
- 交通类型

然后再去拉数据，并分两段输出：

1. 先给列表
2. 再从列表中挑推荐

### 它解决的业务问题

它覆盖的不是单一“查机票”，而是多种交通咨询场景：

- 国内机票
- 火车票
- 国际机票
- 低价票推荐
- 价格趋势
- 翻页换一批结果

`transport_service.py` 也说明了这一点，里面有：

- `get_transport_list_data`
- `get_transport_list_data_for_ai`
- `get_recommend_flights_and_trains`
- `get_low_price_flight_search`
- `get_flight_price_trend`

所以交通模块本质上是“自然语言交通搜索代理”。

## 业务模块六：酒店咨询

这一块对应：

- `hotel_consult_sector`
- `hotel_consult_sector_by_list`
- `hotel_consult_sector_by_recommend`
- `inter_hotel_consult_sector_by_list`
- `hotel_comparison`
- `high_score_hotel_repair`

### 业务上的处理过程

酒店模块会先把用户自然语言解析成酒店筛选条件，例如：

- 城市
- 入住日期
- 离店日期
- 酒店名称关键词
- 补充描述

然后走酒店搜索，再做推荐总结。

### 这部分的特点

它同时支持：

- 国内酒店
- 国际酒店
- 酒店列表结果展示
- 从列表中做 AI 推荐
- 酒店比较

`hotel_service.py` 也能佐证这一点，里面有：

- `nlp_search_hotel_list`
- `nlp_search_hotel_list_for_chat`
- `nlp_search_inter_hotel_list`
- `get_recommend_hotel_list`

所以酒店模块本质上是“自然语言酒店检索与推荐器”。

## 业务模块七：酒店问一问

这一块和普通酒店咨询不同，是单独的业务模块。

对应节点是：

- `hotel_inquire`

### 它解决的问题

不是“推荐我住哪里”，而是“围绕某个具体酒店继续问问题”，例如：

- 某酒店房型如何
- 某酒店的某个设施怎样
- 某酒店房间相关问题

### 它的实现特点

这一模块不是本地完整推理，而是：

- 组装请求参数
- 调外部酒店咨询 API
- 接收流式 SSE 响应
- 把 chunk 实时回推给前端

所以这更像“酒店垂直问答代理层”。

## 业务模块八：小红书导入生成路线

这是 `libai` 很有特色的一块。

对应节点包括：

- `travel_xhs_import_repair`
- `travel_xhs_import_sector`
- `travel_xhs_import_route_planner`
- `travel_xhs_import_transport_planner`
- `travel_xhs_import_hotel_planner`
- `travel_xhs_import_weather_planner`
- `travel_xhs_import_planner`

### 它解决的问题

用户不是直接输入结构化旅游需求，而是输入一段小红书笔记内容、链接或攻略文本，希望系统把它转成可执行行程。

### 业务上的处理方式

这条链路会：

- 先判断用户给的内容是否足够用于导入
- 提取攻略里的目的地、天数、出发地等信息
- 生成路线
- 补齐交通、酒店、天气
- 形成完整行程结果

所以它其实是“内容转行程”的专项工作流。

## 业务模块九：问题推荐与追问引导

这部分对应：

- `question_recommend_node`
- `question_association`
- `question_recommend_groups`

### 业务意义

它不是生成主答案，而是为下一轮交互补问题。

比如在交通场景下，它可能继续推荐：

- 换一批结果
- 看低价趋势
- 看最佳购买时间

也可能根据工具调用结果生成下一轮推荐问题。

所以这部分本质上是“会话增长与引导层”，让用户能继续点选或追问，而不是只停在一轮回答。

## 业务模块十：转录与上下文理解

`transcriptor` 是一个经常被忽略，但业务上很关键的模块。

它负责：

- 提取当前轮的意图
- 维护会话级上下文
- 为后续节点提供用户意图摘要

换句话说，它不是直接对用户可见的模块，但它影响后面所有模块的判断质量。

它让 `libai` 不只是“看这一句话”，而是能在多轮会话里继续理解用户在说什么。

## 业务模块十一：语音对话

`app/services/voice/ai_voice_pipeline.py` 说明语音是独立模块。

### 它的业务作用

让用户可以通过实时语音和“旅行规划师”对话，而不是只通过文本。

### 它做的事

- 建立 WebSocket 连接
- 做用户校验和位置获取
- 加载历史消息
- 调 LLM 进行语音式聊天
- 对整段语音对话做旅行意图总结

所以它是“语音版旅行规划入口”。

## 业务模块十二：攻略图 / 打卡图生成

这一部分对应：

- `guide_shot_api`
- `guide_shot_service`
- `graph/shot/*`

### 业务上的用途

它不是聊天主链路的一部分，而是内容生产能力：

- 选图
- 重写图片描述
- 重新增强图片
- 生成打卡机位或攻略图内容

这说明 `libai` 不只是对话系统，它还承担一部分旅游内容生成能力。

## 业务模块十三：输出编排层

这部分虽然偏技术，但业务上也很重要。

`workflow_service` 和 `output_service` 负责把不同模块结果转换成前端可消费的形态，例如：

- 文本流
- 思考流
- 卡片
- 组件
- markdown
- trip card
- 自定义事件

所以 `libai` 不只是“会算”，还负责“把业务结果包装成用户看得懂的交互形式”。

## 业务模块之间的关系

从业务链路看，可以把这些模块分成三类。

### 1. 分流模块

- `reception`
- `trip_react_reception`
- `transcriptor`

职责：识别用户在说什么。

### 2. 核心业务模块

- 行程规划
- 路线调整
- 交通咨询
- 酒店咨询
- 酒店问一问
- 小红书导入
- 通用聊天

职责：真正解决旅游问题。

### 3. 增强模块

- 问题推荐
- 语音
- 攻略图
- 输出组件层

职责：增强体验、扩展场景、改善交互。

## 从用户视角看 libai 在做什么

如果不用技术语言，可以把 `libai` 的业务职责理解为：

- 先听懂用户到底是在规划、调整、查交通、查酒店，还是在问某个具体酒店
- 再决定要调用哪套旅游能力
- 再把结果整理成适合前端展示的文本、卡片和推荐动作
- 最后把交互持续推进下去

所以它不是简单的 LLM 封装，而是一个“旅游业务会话操作系统”。

## 和 dufu 的业务分工再落一层

从这次业务模块视角看，两者关系可以再准确一点：

- `libai` 负责“理解问题、选择业务模块、组织交互结果”
- `dufu` 负责“提供路线、酒店、交通、聊天存储、风控等底层业务能力”

也就是：

- `libai` 偏业务交互编排
- `dufu` 偏业务能力承载

## 当前最值得记住的结论

`libai` 的业务不是单模块，而是多模块协作：

- 前面有接待分流
- 中间有规划、咨询、调整、导入等核心旅游模块
- 后面有推荐、语音、攻略图和流式输出模块

因此后面读源码时，最好不要按“文件夹”理解，而要按“业务场景”理解：用户在问哪类问题，就去找对应的节点链路和 service 实现。

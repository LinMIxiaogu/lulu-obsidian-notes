# dufu 项目框架总结

## 一句话定位

`dufu` 是“行程规划”产品里的旅游业务主后端，负责承接路线、交通、酒店、聊天记录、攻略图、风控、素材与运营后台等核心业务能力，并向上层 AI 服务或前端提供接口。

它不是一个轻量 API 壳，而是一个业务域很多、企业基础设施接入较深的综合型 Spring Boot 服务。

## 代码仓库整体结构

仓库根目录是一个 Maven 父工程，包含两个模块：

- `qal_dufu_web`：主业务模块，`war` 打包，绝大多数代码都在这里
- `qal_dufu_web-api`：`jar` 模块，目前内容很轻，主要是公共依赖占位

当前理解上，真正需要重点阅读的是 `qal_dufu_web`。

## 技术栈与运行方式

### 基础框架

- `Java 17`
- `Spring Boot 2.6.6`
- `Spring Cloud`
- `MyBatis`
- `OpenFeign`
- `Dubbo`
- `QMQ`
- `Redis`
- `Elasticsearch`

### AI / LLM 相关依赖

除了传统业务栈，它还直接引入了：

- `langchain4j`
- `langchain4j-open-ai`
- `langchain4j-qunar`

这说明 `dufu` 自身也并非完全“不懂 AI”，而是已经内嵌了一部分 LLM/代理相关能力，只是从整体角色看，它仍然更偏业务主后端，而不是主要的对话编排入口。

### 启动方式

启动类是 `qal_dufu_web/src/main/java/com/qunar/corp/qal/dufu/Application.java`。

从启动类可以看出：

- 它是标准 `SpringBootApplication`
- 使用 `@MapperScan` 扫描大量业务域 mapper
- 开启 `Feign`
- 开启异步能力
- 通过 `@ImportResource("classpath:appContext.xml")` 引入 XML 配置

这意味着它是“Spring Boot + XML 配置并存”的企业项目风格，不是纯注解式简洁工程。

## 配置与基础设施特征

### 多环境配置

`src/main` 下存在多套环境目录：

- `resources`
- `resources.beta`
- `resources.simulation`
- `resources.proda`
- `resources.prodb`

说明它对不同部署环境做了显式区分。

### XML 配置承担的职责

从 `appContext.xml` 与 `appContext-dubbo.xml` 可以看到：

- 使用 `property-placeholder` 加载多类 properties
- 开启 `qconfig`
- 开启 `qschedule`
- 引入 `dubbo` provider / consumer 配置

这说明项目对公司内部基础设施依赖很深，运行时不仅依赖数据库，还依赖内部配置中心、任务调度、Dubbo 服务体系等。

## 项目分层理解

可以先按下面四层理解：

### 1. application 层：对外接口入口

`application/` 下集中放 Controller，负责暴露 HTTP 接口。

从现有目录看，入口被分成多类：

- `trip`：行程主业务接口
- `transport`：交通相关接口
- `hotel`：酒店相关接口
- `common`：天气、风控、cookie、公共查询等
- `proxy`：LLM / OSS / 认证回调等代理能力
- `xhs`：小红书相关
- `ugc`：UGC 相关
- `sushi`：运营/标注/内部管理相关的一大组接口
- `backdoor` / `hack`：测试、运维、补数、调试入口

其中 `trip` 和 `sushi` 是目前体量最明显的两个入口域。

### 2. biz 层：核心业务实现

`biz/` 下按业务域拆分，看到的目录包括：

- `trip`
- `route`
- `transport`
- `hotel`
- `poi`
- `xhs`
- `ugc`
- `traceability`
- `materials`
- `guideshot`
- `miejue`
- `hotelInquire`
- `proxy`
- `oss`
- `task`
- `mq`

这层可以理解为真正做业务编排、规则处理、数据转换和领域实现的地方。

### 3. data 层：Mapper + XML

项目大量使用 MyBatis。

从 `resources/mapper` 可以看到很多核心表映射文件，例如：

- `RouteMapper.xml`
- `TripRouteDoMapper.xml`
- `TripUserConversationDoMapper.xml`
- `TripUserMessageDoMapper.xml`
- `HotelInquireMessageMapper.xml`
- `PublicRouteDoMapper.xml`
- `LLMProxyRequestLogMapper.xml`

说明系统包含：

- 路线数据
- 用户会话与消息数据
- 酒店问答数据
- 公共路线库
- LLM 请求日志

这进一步证明它不仅提供即时接口，还承担持久化与业务状态管理。

### 4. common 层：通用能力与外部调用

`common/` 里能看到：

- `feign/`：外部服务客户端
- `utils/`：天气、地理、HTTP、用户 cookie、JSON 等工具
- `executor/`：线程池与监控
- `dataset/`：数据集导出/处理能力
- `exception/`：全局异常处理

`common/feign` 中明确存在：

- `BaiduMapClient`
- `FlightGatewayClient`
- `HotelBaseInfoClient`
- `TicketClient`
- `TrainClient`
- `SightClient`
- `PoiAggregateClient`
- `MaterialsClient`

说明 `dufu` 会主动聚合多个旅游与地图相关下游服务。

## 对外接口分层

`UriRoots.java` 很关键，它定义了接口分层方式：

- `ROOT = /dufu`
- `CLIENT = /dufu/client`
- `CORP = /dufu/corp`
- `BACKDOOR = /dufu/corp/backdoor`
- `SUSHI = /dufu/corp/sushi`
- `QAL_SUSHI = /dufu/corp/qal/sushi`
- `UGC = /dufu/client/ugc`
- 前端直接接口使用 `/trip`

这个设计说明它面对的调用方不止一种：

- 面向前端的业务接口
- 面向客户端/上层服务的 client 接口
- 面向公司内部后台或运营体系的 corp / sushi 接口
- 面向测试和运维的 backdoor 接口

所以 `dufu` 本质上是一个多入口业务平台，而不是单一场景的 API 服务。

## 当前最值得记住的业务域

### 1. trip / route：行程主链路

这是“行程规划”最核心的域。

从 `TripRouteController` 及相关 request/dto 命名可以看到，它覆盖：

- 路线详情查询
- 按天查看路线
- 路线更新
- 出发日期更新
- 自动修正路径
- 刷新距离信息
- 编辑行程
- 行程列表
- 行程收藏
- 行程调整工具执行

这是整个产品的主业务骨架。

### 2. common：公共工具能力

`CommonController` 中已经能直接看到你之前提过的那些接口：

- `queryWeather`
- `riskContentCheck`
- `recordUserCookie`
- `riskImageCheck`
- `getHotPoi`
- `getRelatedQuestions`
- `getRestaurantRank`
- `getDistrictStandardInfo`

这一层很明显就是被上层服务反复复用的“公共旅游能力接口层”。

### 3. transport / hotel：交易与商品能力

结合 `TransportController`、酒店相关 controller 以及 Feign 客户端命名，可以推断这里承接：

- 机票
- 火车
- 低价票
- 酒店基础信息
- 酒店问答

也就是 AI 在生成建议后，真正查询和落地商品能力的后端基础。

### 4. conversation / message：聊天落库与会话能力

从 mapper 与 request 命名可以看到：

- `TripUserConversationDoMapper`
- `TripUserMessageDoMapper`
- `TripUserTranscriptDoMapper`
- `ConversationCreateRequest`
- `ConversationListRequest`
- `MessageSaveRequest`
- `TranscriptSaveRequest`

这说明 `dufu` 不只是纯业务查询接口，它还承担聊天上下文、用户消息、转录记录等持久化职责。

### 5. sushi：内部运营/标注/管理体系

`application/sushi` 下 controller 数量很多，是一个完整子系统，涉及：

- 路线审核
- POI 管理
- Prompt 管理
- 消息查询
- 酒店问答管理
- 收藏夹
- 仪表盘
- 追溯
- 评测
- 小红书数据查询

这更像是内部运营平台 / 标注后台 / 数据治理后台，而不是直接面向终端用户的业务接口。

### 6. xhs / ugc / guideshot / traceability

这些域说明项目还承担内容生产和内容治理相关工作，例如：

- 小红书内容召回与聚合
- 攻略图任务
- UGC 内容相关接口
- 水印追溯 / 溯源能力

这表示“行程规划”并不只是 itinerary planning，还连接了内容消费、素材生产与质量追踪。

## 对 dufu 的更准确认知

以前可以简单说：`dufu` 是旅游业务主后端。

现在更准确的描述应该是：

`dufu` 是“行程规划”产品里的综合业务平台，既承接核心旅游业务，也承接聊天数据、内容数据、内部运营后台、素材与审核流程，并对接大量公司内部基础设施与下游服务。

## 和 libai 的关系再补充一层

结合当前阅读结果，`libai -> dufu` 仍然成立，但可以补充得更具体：

- `libai` 更像 AI 交互入口、Prompt/Graph/Tool 编排层
- `dufu` 更像旅游业务能力平台 + 数据持久化平台 + 内部运营支撑平台
- `libai` 会把天气、风控、路线、交通、酒店等请求下发到 `dufu`
- `dufu` 不只是“被调用接口集合”，它本身还管理会话、路线、内容、后台审核和运营数据

所以它们关系更接近：

- `libai` 负责“理解用户并决定调用什么”
- `dufu` 负责“把旅游业务真正做出来，并把业务状态存下来”

## 后续阅读建议

如果后面继续读 `dufu`，推荐按这个顺序：

1. `application/trip`：先抓主链路接口长什么样
2. `biz/trip/service`：看核心行程生成、编辑、调整逻辑
3. `application/common/CommonController`：看上层 AI 常调用的公共能力
4. `application/transport` 与 `application/hotel`：看商品查询链路
5. `TripUserConversation / Message / Transcript` 相关 mapper 和 service：看聊天数据如何存储
6. `application/sushi`：最后再看内部后台体系

## 当前笔记结论

`dufu` 不是单纯的“旅游接口集合”，而是一个以行程规划为核心、同时覆盖内容、聊天、运营和治理能力的综合型后端平台；在整套系统里，它是承接实际旅游业务和业务数据状态的核心底座。

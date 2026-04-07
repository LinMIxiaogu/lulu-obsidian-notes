## 口述框架：机票查询 MCP 封装开发

### 开场定位（30秒）

> "我们要做的事是：把去哪儿内部的机票查询能力，封装成一个标准的 MCP 工具，让 AI Agent（比如 Cursor、Claude）可以直接调用它来查国内直飞航班。"

---

### 第一部分：MCP 协议接入层

这是整个服务的"门面"，负责把 AI 客户端发来的请求解析成 MCP 标准协议。

需要做的事：

1. 暴露一个 HTTP 接口：POST /qunar/mcp，这是所有 AI 客户端的统一入口

2. 接入官方 MCP Java SDK：用 McpStatelessSyncServer 来处理 initialize、tools/list、tools/call 三个标准方法，不需要自己手写协议分发

3. 写一个传输桥接类（McpStatelessTransportBridge）：把 HTTP 请求体转给 SDK，再把 SDK 的响应写回 HTTP

4. 写服务注册配置类（McpSdkServerConfig）：在这里声明服务名、版本，并把 flight_search 工具注册进去，定义它的入参 schema（出发城市、到达城市、日期，三个必填项）

---

### 第二部分：鉴权与开关

在请求进入工具逻辑之前，需要做两层拦截：

1. Token 鉴权：支持 X-MCP-Auth Header 或 Authorization: Bearer 两种方式，Token 固定配置在服务端，失败返回 JSON-RPC -32001 unauthorized

2. 服务开关：通过配置项 mcp.external.access.switch 控制，关闭时直接返回 -32002 mcp service disabled

---

### 第三部分：flight_search 工具的核心服务层

这是业务逻辑最重的部分，核心类是 FlightMcpService：

参数校验

- 三个入参不能为空

- 日期必须符合 yyyy-MM-dd 格式

- 调 FlightGatewayService 解析城市别名，判断是否为国内站点（非中国/China/CN 的都拒掉），只支持国内航线

请求下游

- 调内部的 third_channel/search 接口（地址和超时都通过配置项控制），传出发城市、到达城市、日期，以及固定的业务参数（离线请求标识、来源标签等）

结果过滤

- 只保留"直飞"分组（DIRECT_FLIGHT），中转分组整体丢掉

- 过滤掉 PTrip 航班

- 过滤掉共享航班（codeshare）

---

### 第四部分：字段映射与转换

这部分由 FlightMcpConverter 负责，把下游复杂的数据结构"压平"成 AI 可读的简洁结构：

- 顶层：出发城市、到达城市、日期、航班总数

- 每条航班：航班号、出发/到达机场（含航站楼）、出发/到达时间、飞行时长、价格、航司名称、航司 logo URL、是否经停、跨天描述

- 最关键的：每条航班要生成一个 jumpUrl（去哪儿 App 跳转链接），格式是 qunarphone://flight/ota?...，里面包含城市、日期、航班号、价格等，供用户点击直接购票

---

### 第五部分：返回格式

成功时同时返回两份内容（兼容不同客户端）：

- content[0].text：结果的 JSON 字符串（Cursor 等偏文本的客户端读这个）

- structuredContent：同一份结果的结构化对象（支持结构化输出的客户端读这个）

错误时区分两类：

- 参数错误（城市不合法、日期格式错等）：isError: true + 明确错误文案

- 运行时错误（下游挂了、数据解析失败）：isError: true + internal error

---

### 一句话总结给对方

> "整体分五块：协议接入层用官方 SDK 搞定 MCP 握手，鉴权开关做安全卡口，服务层负责参数校验、调下游、过滤直飞，转换层把复杂数据压平并生成购票跳转链接，最后按 MCP 规范返回文本+结构化双份结果。"
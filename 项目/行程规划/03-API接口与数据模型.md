# API 接口与数据模型

---

## 主要 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/agent/chat` | POST | 核心对话接口，SSE 流式响应 |
| `/agent/image-process` | POST | 图片处理/分析 |
| `/api/guide_shot` | POST | 打卡机位生成 |
| `/eval` | POST | 评估相关（内部使用）|
| `/agent/getenv` | GET | 获取环境变量（仅开发）|

---

## 核心对话接口详解

### 请求结构

```
POST /agent/chat
Content-Type: application/json
```

```python
class ChatRequest(BaseModel):
    conversationId: Optional[str]          # 会话ID（多轮对话用）
    messages: Optional[List[ChatMessage]]  # 消息历史列表
    content: Optional[str]                 # 本次用户输入文本
    images: Optional[List[str]]            # 图片 URL 列表（多模态）
    messageId: Optional[str]               # 本条消息的唯一ID
    route_id: Optional[str]                # 已有行程ID（用于路线调整）
    c: Optional[str]                       # Qunar 用户参数（含用户信息）
    drawerChatType: Optional[str]          # 抽屉聊天类型
    invokeSource: Optional[str]            # 调用来源标识
    bizSource: Optional[str]               # 业务来源标识
    debug: Optional[bool] = False          # 调试模式
    extMap: Optional[Dict[str, Any]]       # 扩展参数（自定义业务参数）
    # 风控相关
    bmagic: Optional[str]
    betime: Optional[str]
    bver: Optional[str]
    becheck: Optional[str]

class ChatMessage(BaseModel):
    role: str      # "user" 或 "assistant"
    content: str   # 消息内容
```

### 响应：SSE 事件流

响应头：`Content-Type: text/event-stream`

前端通过 `EventSource` 或 `fetch` + `ReadableStream` 接收。

```
event: MESSAGE
data: {"text": "根据您的需求，"}

event: MESSAGE
data: {"text": "我为您规划了5天三亚行程："}

event: THINKING
data: {"text": "用户想去三亚5天，需要考虑海滩景点..."}

event: TRIP_CARD
data: {"routeId": "route_xxx", "days": 5, "title": "三亚5日精华游", "dayRouteList": [...]}

event: HOTEL_CARD
data: {"hotels": [...]}

event: QUESTION_RECOMMEND
data: {"questions": ["需要安排接送机吗？", "推荐亲子餐厅吗？"]}

event: END_MESSAGE
data: {}
```

### SSE 事件类型完整列表

| 事件名 | 触发节点 | 内容 |
|--------|---------|------|
| `START_MESSAGE` | 任意文本节点 | 开始输出文本 |
| `MESSAGE` | 任意文本节点 | 文本块（流式） |
| `END_MESSAGE` | 任意文本节点 | 文本输出结束 |
| `START_THINKING` | thinker_node | 开始输出思考过程 |
| `THINKING` | thinker_node | 思考内容块 |
| `END_THINKING` | thinker_node | 思考结束 |
| `TRIP_CARD` | travel_planner_node | 行程卡片数据 |
| `HOTEL_CARD` | hotel_planner_node | 酒店推荐卡片 |
| `ASSISTANT_AGENT_CARD` | reception_node | 专家团展示卡片 |
| `QUESTION_RECOMMEND` | question_recommend_node | 推荐问题列表 |
| `TRANSPORT_CONSULT_SECTOR_BY_RECOMMEND` | transport_consult_node | 交通推荐数据 |
| `HOTEL_CONSULT_SECTOR_BY_RECOMMEND` | hotel_consult_node | 酒店咨询结果 |
| `ERROR` | 任意节点 | 错误信息 |

---

## 核心数据模型

### 旅行意向

```python
class TravelIntention(BaseModel):
    departure: str               # 出发地
    arrival: str                 # 目的地
    start_date: str              # "2026-04-01"
    end_date: str                # "2026-04-05"
    days: int                    # 5
    adult_count: int             # 成人数
    child_count: int             # 儿童数
    select_tags: List[str]       # ["海滩", "美食", "亲子"]
    trip_style_dict: Optional[dict]  # {"亲子": True, "探险": False}
    select_pois: List[str]       # 用户钦点的景点 ID 列表
```

### 路线规划结果

```python
class RoutePlanResult(BaseModel):
    ready: bool                         # 是否规划完整
    message: str                        # 失败时的原因
    path: str                           # 路线摘要路径
    title: str                          # 行程标题
    days: int                           # 天数
    day_route_list: List[DayRouteResult]

class DayRouteResult(BaseModel):
    day: int                            # 第几天
    theme: str                          # 当天主题
    route_text: str                     # "景点A → 景点B → 景点C"
    highlights: str                     # 亮点描述
    full_desc: str                      # 完整介绍
    spots: List[PoiSpot]               # 景点列表
```

### POI 景点模型

```python
class PoiSpot(BaseModel):
    poi_id: str
    name: str
    visit_time: str      # 建议游览时间，如 "2小时"
    tips: str            # 小贴士

class PoiDetail(BaseModel):
    title: str
    subtitle: str
    image_url_list: List[str]
    description: str
    tips: List[PoiTips]
    reference_list: List[PoiReferences]
```

### 打卡机位请求/响应

```python
class GuideShotRequest(BaseModel):
    taskId: Optional[str]   # 任务ID
    noteId: Optional[str]   # 小红书笔记ID

class GuideShotResponse(BaseModel):
    success: bool
    error: Optional[str]
    data: Optional[Dict[str, Any]]
```

---

## 数据存储层

### Route Store（路线落库）

行程规划完成后，通过 HTTP 接口保存到专门的行程数据库：

```python
# route_store.py
params = {
    "c": "用户c参",
    "userName": "用户名",
    "userIntent": TravelIntention.dict(),
    "routeList": [DayRouteResult.dict(), ...],
    "hotelRecommendList": [{酒店数据}, ...],
    "goTripFlightRecommendData": {机票信息},
    "goTripTrainRecommendData": {火车信息},
    "metaRouteId": "召回路线的原始ID（如有）"
}
response = http_post("trip_route 服务 URL", params)
route_id = response["routeId"]   # 返回行程ID，供前端展示
```

### Conversation Message Store（对话存储）

- 保存每轮对话的 `HumanMessage` 和 `AIMessage`
- 按 `conversationId` 组织
- 用于跨会话的历史回溯

### Transcript Context Store（上下文存储）

- 存储用户的旅行意图（不想每次都重新解析）
- 存储当前咨询记录
- 支持从 Redis 快速读取

---

## 配置层说明

### 多环境配置

```
config/profiles/
├── beta.yaml        # 测试环境（连接 beta 数据库）
├── prod.yaml        # 生产环境
└── simulation.yaml  # 仿真环境
```

### 关键配置项（ai_account_config.py）

```python
# LLM 连接配置（可通过环境变量覆盖）
LLM_BASE_URL = "..."        # API 基础 URL
LLM_API_KEY = "..."         # API Key
LLM_MODEL = "deepseek-..."  # 模型名称
LLM_TEMPERATURE = 0.7       # 生成温度
LLM_MAX_TOKENS = 4096       # 最大输出 token
ENABLE_THINKING = True      # 是否开启思考模式
STREAM_OUTPUT = True        # 是否流式输出
```

### 远程配置（remote_config.py）

支持运行时热更新配置，不重启服务就可以修改：
- LLM 模型切换
- 提示词更新（通过 Langfuse）
- 功能开关（Feature Flag）

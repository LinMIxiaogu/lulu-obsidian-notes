好的，我来用图示把 MCP 的核心原理讲清楚，分三部分：**配置层（mcp.json）→ 协议层（工具注册与调用）→ 模型决策层（如何选工具）**。

---

## 一、MCP 是什么，解决什么问题

MCP（Model Context Protocol）本质是一个**标准化插件协议**，让 AI 模型可以调用外部能力，就像浏览器可以安装扩展一样。

没有 MCP 之前：每个 AI 客户端（Cursor、Claude）要对接一个业务接口，都得定制开发。有了 MCP：业务方按标准封装一次，所有支持 MCP 的客户端都能直接用。

**什么时候用 MCP？** 当你有实时数据、私有系统、或复杂业务逻辑需要给 AI 调用时——比如你们的机票查询、订单系统、内部数据库，这类东西 AI 自己不知道，必须通过 MCP 工具去查。

---

## 二、mcp.json 是什么
![[Pasted image 20260420203046.png]]
`mcp.json` 是 AI 客户端（如 Cursor、Claude Desktop）的**服务发现配置文件**。它告诉客户端：有哪些 MCP 服务可以用，怎么连接它们。每个 MCP 服务可以注册**多个工具**，客户端启动时会向每个服务发 `tools/list` 请求，把所有工具的 schema 全部拿回来。

---

## 三、一次完整的调用流程---

## 四、模型是怎么"知道"用哪个工具的

这是最核心的问题。答案是：**模型不靠代码逻辑选工具，靠读工具的 description（自然语言描述）来决策**。

你注册工具时写的 schema 大概长这样：

```json
{
  "name": "flight_search",
  "description": "查询国内直飞航班信息，输入出发城市、到达城市和日期，返回航班列表和价格",
  "inputSchema": {
    "type": "object",
    "properties": {
      "dep": { "type": "string", "description": "出发城市，如北京" },
      "arr": { "type": "string", "description": "到达城市，如上海" },
      "date": { "type": "string", "description": "出行日期，格式 yyyy-MM-dd" }
    },
    "required": ["dep", "arr", "date"]
  }
}
```

客户端启动时，这段 schema 会被放进模型的 system prompt 里。用户说"北京飞上海"，模型看到 `flight_search` 的描述，判断相关，于是输出一段结构化 JSON（而不是普通文字），客户端拦截这段 JSON，发起 `tools/call`。

**所以工具 description 写得越清晰，模型选对工具的概率越高。** 你们的机票接口需要特别写清楚"只支持国内直飞"，否则用户问国际航班，模型可能也会调这个工具，结果返回错误。

---

## 五、总结

|概念|一句话|
|---|---|
|`mcp.json`|客户端的服务目录，配置去哪找 MCP 服务|
|`tools/list`|启动时握手，模型把所有工具 schema 读进来|
|`tools/call`|模型决策后发出调用，等同于一次 HTTP 请求|
|工具 description|模型选工具的唯一依据，写好它比写代码更重要|
|什么时候用 MCP|实时数据、私有系统、业务逻辑——凡是模型自己不知道的东西|

你的机票封装项目五层结构（协议接入 → 鉴权 → 业务层 → 转换层 → 返回格式）是标准的 MCP 服务端架构，完全正确。最值得关注的细节是：`flight_search` 的 description 要把"国内直飞"限制描述清楚，以及 `jumpUrl` 的生成逻辑——这是让 AI 回答直接可购票的关键。
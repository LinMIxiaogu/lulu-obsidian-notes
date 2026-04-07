# OpenClaw 完整笔记

## 一、什么是 OpenClaw

OpenClaw（曾用名 Clawdbot、Moltbot）是由 Peter Steinberger 开发的自由开源自主 AI 代理，以大型语言模型为驱动，以消息平台为主要用户界面。2026年2月，Steinberger 加入 OpenAI，项目移交开源基金会。

**核心特点：**

- 本地优先（Local-first），数据不上传云端
    
- 自托管，运行在用户自己的设备上（Mac、VPS、树莓派）
    
- 模块化、插件优先的架构设计
    
- 支持接入 Claude、GPT、DeepSeek 等多种 LLM
    

---

  

## 二、整体架构（六层）

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773908440468-0dec677b-7bb3-4acb-a8af-e9d4bc27b96f.png" width="603" title="" crop="0,0,1,1" id="uf9287292" class="ne-image">

  

```Plain
消息平台接口（用户层）
       ↓
  Gateway（控制平面）
       ↓
  Agent 运行时（核心逻辑层）
       ↓
  技能与插件系统
       ↓
  Monorepo 代码结构
       ↓
  本地持久化存储
```

  

### 1. 消息平台接口

支持 Signal、Telegram、Discord、WhatsApp、iMessage、Slack 等，OpenClaw 不提供独立 UI，寄生在现有平台上。

  

### 2. Gateway（控制平面）

- 默认监听 **18789 端口**的 WebSocket 服务器
    
- 负责：会话管理、消息路由、多 Channel 并发
    
- 只要本地设备开机，Gateway 持续运行
    

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773908462857-44c3894c-0657-4f79-a42a-f821ac76cd00.png" width="629" title="" crop="0,0,1,1" id="u58bdaa9c" class="ne-image">

  

### 3. Agent 运行时

- LLM 接入（Claude / GPT / DeepSeek）
    
- 上下文引擎（ContextEngine）
    
- 工具调用 · RPC 模式 · 流式传输
    

### 4. 技能与插件系统

- 技能优先级：工作区技能 > 全局安装技能 > 内置捆绑技能
    
- 扩展工具：浏览器自动化（CDP）、本地设备 Nodes、Cron 定时任务、ClawHub 注册中心
    

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976254009-eeaf006b-213f-4673-bf5c-4c2062929dc6.png" width="532" title="" crop="0,0,1,1" id="u6b9ab8c1" class="ne-image">

  

### 5. Monorepo 代码结构（pnpm 管理）

|   |   |
|---|---|
|包|职责|
|`packages/core`|框架基础类，约 8MB|
|`packages/gateway`|控制平面，WebSocket 路由|
|`packages/agent`|推理运行时，工具调用|
|`packages/cli`|命令行工具|
|`packages/sdk`|插件开发包|
|`packages/ui`|WebChat 界面与控制台|

  

  

### 6. 本地持久化存储

- 配置数据（config.yaml）
    
- 会话历史（跨 session 记忆）
    
- 工作区文件系统（`~/.openclaw/workspace/`）
    
- 向量数据库（RAG 检索，可选）
    

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976281598-00cdf10f-3884-4f29-a3a3-be263aee9dd2.png" width="528" title="" crop="0,0,1,1" id="ud439c6b8" class="ne-image">

  

---

  

## 三、即时通讯（IM）集成详解

### 工作原理

OpenClaw 不提供自己的聊天界面，而是作为"机器人"接入你已有的 IM 平台。用户在平台内发消息，Gateway 监听并接收，处理后原路返回响应。

  

### 支持的平台

|   |   |   |
|---|---|---|
|平台|接入方式|特点|
|Signal|Signal CLI / 链接设备|隐私性强，端对端加密|
|Telegram|Bot API Token|配置最简单，功能最丰富|
|Discord|Bot Token + OAuth|支持服务器频道、私信|
|WhatsApp|WhatsApp Business API|需要官方 API 授权|
|iMessage|macOS 本地运行|仅限 Apple 设备|
|Slack|Slack App + OAuth|适合团队/企业场景|

  

  

### 消息流转过程

```Plain
用户在 IM 平台发送消息
       ↓
Gateway WebSocket 接收（port 18789）
       ↓
解析平台格式 → 统一消息结构
       ↓
路由给对应 Agent
       ↓
Agent 处理 → 生成响应
       ↓
Gateway 将响应转换回平台格式
       ↓
推送回 IM 平台显示给用户
```

  

### Channel 配置

每个 Agent 可以绑定一个或多个 Channel，同一 Agent 可同时响应多个平台的消息：

  

```YAML
agents:
  - name: my-agent
    channels:
      - type: telegram
        token: "BOT_TOKEN"
      - type: discord
        token: "BOT_TOKEN"
```

  

### 多 Agent 路由

不同平台或不同群组可以路由给不同的 Agent，实现场景隔离（如：工作 Slack 用严肃 Agent，私人 Telegram 用随意 Agent）。

  

---

  

## 四、Adapter（适配器）详解

### 什么是 Adapter

Adapter 是 Gateway 与各 IM 平台之间的**协议翻译层**。每个平台的消息格式、认证方式、事件类型都不同，Adapter 负责将平台特定的格式双向转换为 OpenClaw 内部统一的消息结构。

  

### Adapter 架构

```Plain
IM 平台（Telegram / Discord / Signal ...）
       ↕ 平台原生协议（HTTP / WebSocket / gRPC）
   Adapter（协议翻译层）
       ↕ 统一消息结构
   Gateway（统一入口）
       ↓
   Agent 运行时
```

  

### 统一消息结构

所有平台消息经 Adapter 转换后，都变成同一种内部格式：

  

```TypeScript
{
  id: string,           // 消息唯一 ID
  channelType: string,  // 来源平台（telegram / discord ...）
  channelId: string,    // 频道/群组 ID
  userId: string,       // 发送者 ID
  content: string,      // 消息正文
  attachments: [],      // 附件（图片、文件等）
  timestamp: number,    // 时间戳
  metadata: {}          // 平台特有的额外字段
}
```

  

### 各平台 Adapter 差异

|   |   |   |
|---|---|---|
|平台|连接方式|特殊处理|
|Telegram|Long polling / Webhook|支持 Inline Button、命令前缀 `/`|
|Discord|WebSocket Gateway|需处理 Intent 权限、频道类型|
|Signal|Signal CLI 本地进程|需本机注册设备，延迟较高|
|WhatsApp|REST API + Webhook|消息模板审核限制|
|iMessage|AppleScript / macOS API|仅限 macOS，不支持跨设备|
|Slack|Events API + Socket Mode|需处理 OAuth Scopes、工作区隔离|

  

  

### 自定义 Adapter

通过 `packages/sdk` 可以开发自定义 Adapter，接入任意平台（如企业微信、飞书、自建 IM）：

  

```TypeScript
import { BaseAdapter } from '@openclaw/sdk';

export class MyAdapter extends BaseAdapter {
  async connect() { /* 建立连接 */ }
  async send(message: OutboundMessage) { /* 发送消息 */ }
  async onMessage(handler: MessageHandler) { /* 接收消息 */ }
  async disconnect() { /* 断开连接 */ }
}
```

  

### Adapter 的容错机制

- 连接断开时自动重连，支持指数退避（exponential backoff）
    
- 消息发送失败时进入重试队列
    
- 平台限流（rate limit）时自动降速排队发送
    
- 每个 Adapter 独立运行，一个平台故障不影响其他平台
    

---

  

## 五、定时任务（Cron / Wakeups）详解

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976421536-7473666c-d648-4ee8-9957-e060e7eae550.png" width="543" title="" crop="0,0,1,1" id="u5943af67" class="ne-image">

  

### 两种触发机制

|   |   |   |
|---|---|---|
|类型|说明|适用场景|
|**Cron**|按固定时间表重复执行|每日摘要、定期检查、周报|
|**Wakeup**|一次性未来某时刻唤醒|提醒、延迟任务、跟进事项|

  

  

### Cron 任务

使用标准 cron 表达式配置，Agent 在指定时间点被唤醒，执行预设任务后再次休眠：

  

```YAML
cron:
  - expression: "0 9 * * 1-5"   # 工作日早 9 点
    task: "发送今日日程摘要"
  - expression: "0 18 * * *"    # 每天下午 6 点
    task: "整理今日工作区文件"
```

  

### Wakeup 任务

Agent 在对话中自主设置，在未来某个时间点唤醒自己继续任务：

  

```Plain
用户："一小时后提醒我开会"
       ↓
Agent 调用 wakeup 工具，注册 wakeup(timestamp, context)
       ↓
一小时后 Gateway 触发唤醒
       ↓
Agent 恢复上下文，发送提醒消息
```

  

### 定时任务的执行流程

```Plain
时间到达（Cron 触发 / Wakeup 触发）
       ↓
Gateway 创建新的 session
       ↓
加载任务预设的上下文（或恢复 wakeup 时保存的上下文）
       ↓
Agent 执行任务（可调用工具、发消息、写文件）
       ↓
afterTurn() 写回记忆，session 结束
```

  

### 与记忆系统的联动

- Cron 任务执行结果自动写入 `memory/` 日志
    
- Wakeup 时可恢复触发前保存的对话上下文
    
- 定时任务产出的文件写入 `workspace/`，被 `clawmem-watcher` 自动索引
    

### 注意事项

- 设备关机或 Gateway 未运行时，定时任务**不会执行**（本地优先的代价）
    
- 建议在 VPS 等常开设备上运行以保证定时任务可靠性
    
- 大量 Cron 任务并发时注意 LLM API 的 rate limit
    

---

  

## 六、扩展工具（Built-in Tools）详解

### 工具总览

OpenClaw 内置六类扩展工具，Agent 通过 `tool_use` 调用，结果以 `tool_result` 返回：

  

```Plain
Agent（LLM 推理）
  └── tool_use
        ├── 浏览器自动化（Browser / CDP）
        ├── 本地设备（Nodes）
        ├── 定时任务（Cron / Wakeups）
        ├── 可视化工作区（Canvas / A2UI）
        ├── MCP 服务器
        └── ClawHub 技能注册
```

  

---

  

### 1. 浏览器自动化（Browser / CDP）

通过 Chrome DevTools Protocol（CDP）控制本地 Chromium 实例，实现完整的浏览器操作能力。

  

**能做什么：**

  

- 网页抓取与内容提取
    
- 表单填写与提交
    
- OAuth 登录流程（自动处理重定向）
    
- 2FA 验证码输入（配合 Nodes 读取通知）
    
- 截图与页面录制
    
- JavaScript 执行（操作 DOM、读取页面状态）
    

**使用场景举例：**

  

```Plain
"帮我登录 GitHub，把 starred 列表导出成 CSV"
"每天早上抓取竞品价格并写入表格"
"帮我填写这个报销申请表"
```

  

**安全边界：**

  

- 浏览器实例在本地运行，Cookie 和会话不离开设备
    
- 建议为 OpenClaw 创建独立浏览器 Profile，避免与个人浏览数据混用
    

---

  

### 2. 本地设备（Nodes）

直接访问运行设备的硬件和系统资源。

  

|   |   |
|---|---|
|Node 类型|功能|
|camera|调用摄像头拍照|
|screen|截图 / 屏幕录制|
|location|获取当前地理位置（移动设备）|
|notification|发送系统通知|
|file|读写本地文件系统|
|clipboard|读写剪贴板|
|shell|执行 shell 命令（需授权）|

  

  

**使用场景举例：**

  

```Plain
"截图发给我看一下现在屏幕显示什么"
"把桌面的 PDF 文件整理归类到对应文件夹"
"当有新通知时自动摘要发给我"
```

  

**权限控制：**

  

- 每类 Node 需要在 `config.yaml` 中显式开启
    
- `shell` Node 权限最高，建议严格限制可执行命令白名单
    

---

  

### 3. Canvas（A2UI）

Agent 驱动的可视化工作区，可以动态生成和渲染 UI 界面，用于展示结构化信息、交互式表单、数据图表等。

  

**特点：**

  

- Agent 可以主动"绘制"一个界面推送给用户
    
- 支持按钮、表单、列表、图表等组件
    
- 用户在 Canvas 上的操作可以反馈给 Agent 继续处理
    

**使用场景举例：**

  

```Plain
"用表格展示这周的任务完成情况"
"生成一个可交互的预算分配界面"
"把搜索结果做成卡片列表展示"
```

  

---

  

### 4. MCP 服务器（Model Context Protocol）

MCP 是 Anthropic 提出的开放协议，允许 Agent 连接外部数据源和服务。OpenClaw 支持作为 MCP 客户端接入任意 MCP 服务器。

  

**可接入的 MCP 服务器类型：**

  

|   |   |
|---|---|
|类别|示例|
|文件与知识库|Google Drive、Notion、Obsidian|
|开发工具|GitHub、GitLab、Jira、Linear|
|通讯协作|Gmail、Outlook、Calendar|
|数据库|PostgreSQL、SQLite、MongoDB|
|商业服务|Salesforce、HubSpot、Stripe|
|自定义|任何实现了 MCP 协议的服务|

  

  

**配置方式：**

  

```YAML
mcp_servers:
  - name: github
    command: "npx @modelcontextprotocol/server-github"
    env:
      GITHUB_TOKEN: "your_token"
  - name: google-drive
    command: "npx @modelcontextprotocol/server-gdrive"
```

  

**ClawMem（记忆 MCP 服务器）：**

ClawMem 是专为 OpenClaw 设计的记忆增强 MCP 服务器，提供 28 个 MCP 工具，包括：

  

- `clawmem_search`：语义搜索历史记忆
    
- `clawmem_get`：精确获取某条记忆
    
- `clawmem_session_log`：查看 session 日志
    
- `clawmem_timeline`：按时间轴浏览记忆
    
- `clawmem_similar`：查找相似内容
    

---

  

### 5. ClawHub 技能注册

ClawHub 是 OpenClaw 的官方技能注册中心，类似 npm registry。

  

**功能：**

  

- Agent 可以自动发现适合当前任务的技能
    
- 一键安装社区贡献的技能包
    
- 技能版本管理与更新
    

**使用方式：**

  

```Plain
"安装一个能处理 PDF 的技能"
       ↓
Agent 查询 ClawHub，找到匹配技能
       ↓
自动下载安装到 workspace/skills/
       ↓
下次对话即可使用
```

  

**安全警告：** ClawHub 缺乏严格的安全审核机制，Cisco 研究发现存在恶意技能。安装前务必审查技能来源和代码。

  

---

  

## 七、Skills（技能系统）详解

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976401705-238e193f-d5f9-4637-b31c-73217e0b724d.png" width="528" title="" crop="0,0,1,1" id="u964a1dd1" class="ne-image">

  

### 什么是 Skill

Skill 是 OpenClaw 的最小扩展单元。每个 Skill 是一个**目录**，目录内必须包含一个 `SKILL.md` 文件，描述该技能的用途、触发条件和使用方式。

  

### 目录结构

```Plain
my-skill/
  ├── SKILL.md          # 技能描述（必须）
  ├── index.js          # 执行逻辑（可选）
  ├── config.yaml       # 技能配置（可选）
  └── resources/        # 静态资源（可选）
```

  

### 三级优先级（高 → 低）

```Plain
工作区技能（~/.openclaw/workspace/skills/）
        ↓ 覆盖
全局安装技能（全局 skills 目录）
        ↓ 覆盖
内置捆绑技能（随软件发布的默认技能）
```

  

### 技能的执行流程

```Plain
Agent 收到任务
       ↓
扫描可用 Skill 列表，读取各 SKILL.md
       ↓
LLM 判断是否需要调用某个 Skill
       ↓
触发 tool_use → 执行 Skill 逻辑
       ↓
Skill 可读写 workspace/ 下的文件
       ↓
执行结果作为 tool_result 返回 LLM
       ↓
LLM 继续推理或结束本轮
```

  

---

  

## 八、Agent 运行时详解（一个 Turn 的生命周期）

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976102816-5fc8df81-ca9c-4c44-a43f-ad7699e20829.png" width="439" title="" crop="0,0,1,1" id="u4c07e56b" class="ne-image">

  

```Plain
消息到达（来自 Gateway）
       ↓
ContextEngine（上下文引擎）
  ├── ingest()：存储新消息
  ├── buildContext()：按 token 预算裁剪历史
  └── retrieve()：向量检索补充长期记忆
       ↓
LLM 推理（Claude / GPT / DeepSeek）
  → 生成 content blocks
       ↓
响应解析
  ├── type = text → 直接回复用户
  └── type = tool_use → 工具调用循环
       ↓（tool_use 路径）
可调用工具
  ├── 浏览器 / CDP
  ├── Nodes / 文件
  ├── Cron / Wakeups
  ├── Canvas / A2UI
  ├── MCP 服务器
  └── Skills / ClawHub
       ↓ tool_result 回传 → 重新推理（agentic loop）
afterTurn()
  → 写回存储 · 更新会话历史 · 响应推送用户
```

  

---

  

## 九、五大身份文件关系（身份体系）

|   |   |   |
|---|---|---|
|文件|层级|职责|
|`AGENTS.md`|规则层|启动编排 · 操作规则 · 中枢|
|`SOUL.md`|AI 自身|哲学 · 价值观 · 稳定底层设定|
|`IDENTITY.md`|AI 自身|名称 · 表情 · 外观|
|`USER.md`|用户层|用户偏好 · 个人档案|
|`MEMORY.md`|记忆层|铁律规则 · 持久事实（人工蒸馏）|
|`memory/`|记忆层|每日会话日志（原始自动写入）|

  

  

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774235456130-d871975e-a151-439e-b060-24d2c4856110.png" width="563" title="" crop="0,0,1,1" id="CXus7" class="ne-image">

  

### 关键关系

```Plain
AGENTS.md
  ├──加载并体现人格──→ SOUL.md
  ├──主会话加载──────→ USER.md
  └──刻意分离────────→ IDENTITY.md

SOUL.md ←──调节语气表达──→ USER.md

IDENTITY.md ──记忆检索──→ USER.md
IDENTITY.md ──写入日志──→ memory/

USER.md ──────────────→ MEMORY.md
memory/ ──人工蒸馏────→ MEMORY.md
```

  

`**AGENTS.md**`**（规则层 · 中枢）** 启动时第一个被读取，相当于 Agent 的"操作手册"。它决定加载哪些文件、执行什么编排规则。主动加载 `SOUL.md` 体现人格，主动挂载 `USER.md` 作为会话上下文，同时与 `IDENTITY.md` 刻意分离——外观是独立的关注点，不混入行为规则。

  

`**SOUL.md**`**（AI自身 · 灵魂）** 定义 AI 的哲学立场和价值观，是稳定不变的底层设定。它的作用是"调节语气表达"——每次回复时 `USER.md` 里的用户偏好会与 `SOUL.md` 里的价值观共同影响输出风格，两者都读，但 SOUL 的优先级更高。

  

`**IDENTITY.md**`**（AI自身 · 外观）** 管理名称、表情、视觉外观等表层属性。与 `SOUL.md` 刻意分离的设计意图是：换一套外观（改名字、换头像）不应该影响 AI 的价值观，反之亦然。运行时它会往 `memory/` 写入日志，也会触发记忆检索注入 `USER.md`。

  

`**USER.md**`**（用户层 · 个人档案）** 存储用户的偏好、习惯、个人背景。每次主会话开始时由 `AGENTS.md` 加载进 context，让 AI 知道"在跟谁说话"。它是 `SOUL.md` 的调节对象，也是最终向 `MEMORY.md` 输送内容的来源。

  

`**MEMORY.md**`** + **`**memory/**`**（记忆层）** 两者分工明确：`memory/` 是原始日志，每次会话自动写入；`MEMORY.md` 是经过人工筛选蒸馏的"铁律"——只有真正值得长期保留的事实和规则才会从日志里提炼进来。这个"人工蒸馏"环节是设计亮点，防止记忆层被无用信息污染。

  

五个文件属于workspace级别，而 workspace 和 Agent 是一对一绑定的。

  

### 设计亮点

- **SOUL 与 IDENTITY 刻意分离**：换外观不影响价值观
    
- **人工蒸馏机制**：日志 → 人工筛选 → 铁律，防止记忆噪音
    
- **两层记忆分工**：自动写入（日志）与手动维护（铁律）
    

---

  

## 十、上下文管理及记忆系统

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774234621412-cd927773-309c-48bc-a0e2-4778e1767962.png" width="448" title="" crop="0,0,1,1" id="BuIn0" class="ne-image">

  

### 短期记忆（In-context Memory）

当前 session 内，存在 context window 里，session 结束即消失。

  

- 消息队列（user / assistant turns）
    
- 工具调用记录（tool_use + tool_result）
    
- 从长期记忆检索注入的片段
    

### 长期记忆（Persistent Memory）

|   |   |
|---|---|
|类型|说明|
|用户记忆|跨所有 Agent 共享，属于人类本身|
|Agent 记忆|某个 Agent 自己的操作日志，session 粒度|
|静态知识库|手动放入 `resources/`，无时间衰减|

  

  

OpenClaw 记忆系统的核心哲学是：文件是唯一的真相来源，Agent 只记得写到磁盘上的东西。没有黑盒数据库，没有专有格式，全是你自己可以打开编辑的纯 Markdown。

  

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774234654684-3f8acf64-bba5-4815-acab-9fe90df125dc.png" width="603" title="" crop="0,0,1,1" id="kNMpL" class="ne-image">

  

**Bootstrap 有硬性上限。** 单个文件超过 20,000 字符会被截断，所有 Bootstrap 文件合计上限 150,000 字符约 50K tokens。可用 `/context list` 命令检查文件是否被正确加载、是否被截断。

  

#### Memory Flush（compaction 前自动写入）

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774235117371-07678b27-c671-4dee-ac47-3196533c5073.png" width="561" title="" crop="0,0,1,1" id="MaO53" class="ne-image">

  

**Memory Flush 是最关键的设计。** 当 session token 估计值超过 `contextWindow - reserveTokensFloor - softThresholdTokens`，触发一次静默 Agentic Turn，提示 Agent 把重要内容写入日志；Agent 通常回复 `NO_REPLY`，用户完全感知不到这个过程。每个 compaction 周期只触发一次，工作区只读时跳过。

  

#### Hyber检索

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774234885481-91731e49-ff92-44d5-a9a6-88cf72d54c89.png" width="634" title="" crop="0,0,1,1" id="C7L6M" class="ne-image">

  

Hybrid 检索是记忆的"大脑"。 BM25 关键词搜索 + 向量语义搜索双路并行，支持 MMR 多样性重排和时间衰减，支持 OpenAI、Gemini、Voyage、Mistral、Ollama 以及本地 GGUF 多种 Embedding 提供商。Agent 通过 memory_search 和 memory_get 两个工具访问，memory_get 文件不存在时返回空字符串而非抛错。

  

### 写入和读取时机

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774235229966-b3c78369-57e4-423f-bc54-37d4ae8758c4.png" width="537" title="" crop="0,0,1,1" id="NVVWI" class="ne-image">

  

**读取时机（写进去之后怎么回到 context）**

  

`MEMORY.md` 每次启动自动作为 Bootstrap 注入——这是长期记忆影响 Agent 行为最直接的方式，不需要检索。

  

更早的日志文件（两天前的）则不会自动加载，Agent 需要主动用工具`memory_search` 语义检索或 `memory_get` 精确读取才能看到它们。今天和昨天的日志在 session 开始时自动读取，两天以上的任何内容都需要 Agent 主动搜索。(工具是否调用由agent自主决定)

  

### 记忆方案扩展

OpenClaw 通过**插槽（Slot）+ Hooks 的插件体系**支持扩展记忆模块，核心为两个slots

  

- `plugins.slots.memory` 是浅层记忆插槽，控制 `memory_search` 和 `memory_get` 两个 Agent 工具的具体实现。默认是 `memory-core`（SQLite + BM25/Vector Hybrid），设为 `"none"` 可完全禁用记忆检索。
    
- `plugins.slots.contextEngine` 是更底层的插槽，控制整个 ContextEngine——也就是每次 LLM call 之前 `ingest()`、`buildContext()`、`retrieve()` 三个阶段的全部逻辑。替换这个插槽等于接管了"LLM 看到什么"这件事的完整控制权。
    

|   |   |   |   |   |
|---|---|---|---|---|
|维度|原生 memory-core|OpenViking|Mem0|MemOS|
|出身|OpenClaw 官方内置|火山引擎（字节跳动）|Mem0 官方出品|MemTensor 团队|
|存储介质|本地 Markdown + SQLite 索引|VikingFS 虚拟文件系统|向量 DB，云端/本地可选|云端/本地 SQLite 可选|
|检索方式|BM25 + Vector Hybrid（0.7/0.3）|L0/L1/L2 分级递归向量检索（类cache）|语义向量 + LLM 提炼|语义 + 关键词混合检索|
|记忆策略|时序记忆、长期记忆、五类身份文件、向量化检索|三级目录 + L0/L1/L2 分层加载（like cache）|按类别embedding，如身份信息（姓名、职业等）、 偏好（工具选择、沟通风格等）、目标和意图|工作记忆（WorkingMemory）、情节记忆（EpisodicMemory）和语义记忆（SemanticMemory）三个层次|
|写入触发|手动 / compaction 前 Flush 静默 Turn|会话结束自动提取入库|每轮对话结束 Auto-Capture|每轮对话结束自动 Add 上传|
|读取触发|Bootstrap 全量注入 + 工具按需召回|每轮回复前 Auto-Recall|每轮回复前 Auto-Recall|每轮回复前 Recall 注入|
|抗 compaction 丢失|弱，Flush 可能来不及|强（外部存储）|强（外部存储）|强（外部存储）|
|多 Agent 隔离|workspace 物理路径隔离|user/agent 目录分离|agentId 参数隔离（有限）|multiAgentMode 原生支持最好|
|数据隐私|完全本地，最高|本地部署，高|云/本地可选|云/本地可选|
|token 效率|Bootstrap 全量注入，成本较高|-91% token（官方数据）|按需召回，节省显著|-70% token（官方数据）|
|slot||contextEngine|memory|memory|
|接入复杂度|零（内置）|需 Python 3.10 + 本地服务|30 秒，需 API Key|需 API Key 或本地部署|
|推荐场景|隐私优先，轻量单 Agent，零外部依赖|长任务/本地优先，多 Agent 协作，token 成本敏感|极速接入，不想运维，云端无所谓|多 Agent 团队，共享记忆池，技能进化需求|

  

  

- 目前的记忆策略其实都不大健全
    
- 目前是否有记忆的必要？
    

|   |   |   |   |   |
|---|---|---|---|---|
|策略名称|核心逻辑|优点|缺点|适用场景|
|全量记忆|将所有对话历史完整放入上下文|实现简单，无需复杂算法|触发上下文长度上限，响应变慢成本升高|仅适用于对话轮次很少或内容短的场景，如简单 QA 或一次性问答|
|滑动窗口|只保留最近 N 条消息，超出则丢弃最旧的|实现非常简单，并且低价|健忘性强，旧信息永久丢失|适用于短对话场景或历史依赖不强的任务，如 FAQ 助手、简单闲聊机器人|
|相关性过滤|用 LLM 筛选历史消息中"重要"内容保留，不重要的丢弃|保证关键知识不会遗忘|如何准确评估"重要性"是难点|适合信息密集且需要筛选的场景，如知识型对话机器人或研究助理工具|
|摘要/压缩|用 LLM 将旧对话压缩成摘要，再放入上下文|大幅节省上下文长度，长期记忆能力强|质量取决于 LLM，可能遗漏细节|适用于长对话且需要保留上下文要点的场景，如 AI 心理陪伴助手，需记住用户关键信息但不必逐字记住每句话|
|向量数据库|将消息 Embedding 后存入向量库，按语义相似度检索注入上下文|语义级别检索，存储容量大，效率高|依赖嵌入模型质量，增加系统复杂度|需要长期记忆的对话系统，如个性化助理；非常适合在聊天之外存储知识，让记忆检索具备类似 RAG 的效果|
|知识图谱|将记忆结构化为实体-关系图，支持图遍历和推理|将记忆结构化后能进行精细的检索和推理|维护成本高，规模大时面临存储问题|适合知识密集型应用和需要跨事件推理的智能体，如企业客户支持 AI、科研助理 AI 等|
|分层记忆|结合短期（滑动窗口）与长期（向量库/摘要）记忆，按规则在层间迁移|结合短期记忆与长期记|||

  

  

---

  

## 十一、Agents 设计

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976129201-4864f44e-8580-4442-8b44-8cf62ffc6fbb.png" width="575" title="" crop="0,0,1,1" id="u679f04f0" class="ne-image">

  

### 主从分层结构

```Plain
主 Agent（接收用户消息 · 规划 · 调度）
  ├── 子 Agent A（独立 context 隔离）
  ├── 子 Agent B（独立 context 隔离）
  └── 子 Agent C（独立 context 隔离）
```

  

### 子 Agent 隔离

通过 **AsyncLocalStorage** 实现，插件状态不跨 Agent 边界泄漏。

  

### 单个 Agent 配置六维度

|   |   |
|---|---|
|维度|内容|
|模型配置|provider · model · temperature|
|工具权限|允许调用的工具列表|
|记忆配置|`memorySearch.extraPaths`|
|系统提示词|persona · 任务边界|
|上下文引擎|`plugins.slots.contextEngine`（可替换）|
|Channel 绑定|响应哪些消息平台|

  

  

### RPC 模式

子 Agent 可跨设备运行，结果通过 RPC 返回主 Agent，支持分布式 Agent 网络。

  

---

  

## 十二、prompt的加载

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774250117267-3e1e97db-ad2c-4df1-bba7-a444f362b635.png" width="526" title="" crop="0,0,1,1" id="ue020368f" class="ne-image">

  

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1774250138733-0b4b746c-6d69-4844-8a82-3bf57d045ac7.png" width="661" title="" crop="0,0,1,1" id="ubf51abbc" class="ne-image">

  

**Project Context 的注入顺序**

  

Bootstrap 文件按固定顺序注入：AGENTS.md → SOUL.md → TOOLS.md → IDENTITY.md → USER.md → HEARTBEAT.md → BOOTSTRAP.md（仅新 workspace 首次）→ MEMORY.md（有则注入，否则找 memory.md 后备）。

  

每次 Turn 都重新注入，所有文件都消耗 token。`memory/*.md` 日志文件不自动注入，只能通过 `memory_search` 和 `memory_get` 工具按需读取。

  

---

  

## 十三、关键设计原则总结

1. **本地优先**：所有数据在用户设备上，不依赖云端
    
2. **模块化插件架构**：核心仅 ~8MB，功能通过插件扩展
    
3. **Adapter 统一抽象**：各 IM 平台差异被 Adapter 屏蔽，Agent 只看统一消息格式
    
4. **身份****与行为分离**：SOUL（价值观）与 IDENTITY（外观）独立管理
    
5. **记忆分层**：短期（context）→ 长期（持久化）→ 铁律（人工蒸馏）
    
6. **强隔离多 Agent**：AsyncLocalStorage 保证子 Agent 状态不泄漏
    
7. **agentic loop**：工具调用结果回送 LLM 再推理，直到任务完成
    
8. **定时自主性**：Cron + Wakeup 让 Agent 可以脱离用户主动触发而自主执行
    
9. **扩展工具生态**：Browser、Nodes、Canvas、MCP、ClawHub 覆盖从设备到云服务的全链路
    
10. **技能安全边界**：第三方技能存在提示注入风险，需谨慎审查来源
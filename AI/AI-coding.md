# 一、概况

## 1.1 发展

**（1）AICoding的工具**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=NTE0MzMwYjQ2NGI5YTkyZTYyYWY1NGYzNTEwOWNlYTVfMzR0d09IN3dOMzhoY0Iwc2VqQ25PaEdZUE1sdHloa25fVG9rZW46REw4ZWJsd2Vhb0YwM0R4U3RvSGNVRVhHbkdUXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

**（2）AiCoding生态**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGI0MGY0ZWU2NDc1ZDc0ZDBmNzYyOTU4NWI1YWUzMDJfZGUyY1VYOVhTR3VieWdxc21uQkhONEF6Q242QTA0UmtfVG9rZW46VlRIemJJN05jb3hBSEt4cnpOUmN2bmtKblFoXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

## 1.2 总结

  

$$\text{AI Coding 产出} = \underbrace{\text{模型能力}}_{\text{智商}} \times \underbrace{\text{上下文质量}}_{\text{知识}} \times \underbrace{\text{工作模式}}_{\text{执行力 }}$$

  

|发展阶段|工具/形态|核心进化点|上下文质量 (Context/知识)|工作模式 (Mode/执行力)|你的角色|
|---|---|---|---|---|---|
|**1. 史前时代**|**网页****编程**<br><br>(ChatGPT / Claude Web)|**"高智商，零记忆"**|**极低 (手动搬运)**<br><br>依赖手动 Copy & Paste。模型不知道项目全貌，容易产生“幻觉引用”。|**问答式 (Chat / Zero-shot)**<br><br>无状态的请求-响应。它无法触碰代码文件，代码能不能跑全靠人肉验证。|**搬运工**(Copy & Paste)|
|**2. 辅助时代**|**GitHub Copilot**<br><br>(标准插件版)|**"猜你只想写下一行"**|**中 (局部感知 / Local Context)**<br><br>基于 Jaccard 相似度或简单检索，只能看到当前 Buffer 和邻近 Tab，看不到架构层。|**补全式 (FIM / Next Token)**<br><br>基于 Fill-In-the-Middle 技术。它不进行逻辑推理，而是进行概率预测（System 1 快思考）。|**驾驶员**<br><br>(它只是副驾)|
|**3. Agent 时代**|**Cursor**<br><br>(AI Native IDE)|**"懂全工程的结对编程"**|**极高 (全库 RAG / Hybrid)**<br><br>本地向量索引 + 关键词检索。能精准定位到你的 `Utils` 类和数据库 Schema 定义。|**流式应用 (Flow / Speculative Edit)**<br><br>核心是 **"Shadow Workspace"**。它在后台预测性地 Apply 代码变更（Diff），不需要你离开编辑器。它是**人机协同 (Human-in-the-loop)** 的极致，强调低延迟的交互流。|**Code Reviewer**<br><br>(审核员)|
|**Claude Code**<br><br>(Terminal Agent)|**"会用工具的资深工"**|**环境感知 (Runtime Context)**<br><br>不仅看代码，还能通过 `ls` `grep` 看文件结构，通过 `cat` 读取运行时的 Log 和报错信息。|**推理-行动循环 (****ReAct** **Loop)**<br><br>典型的 **Reasoning + Acting** 模式。它不仅是生成文本，而是**Tool Use (工具调用)**：它自己决定执行 Bash 命令、运行测试、根据报错反思、再修改代码。|**指挥官**<br><br>(Commander)|
|**Manus**<br><br>(通用智能体)|**"全自动外包团队"**|**动态获取 (****Search** **+ Runtime)**<br><br>代码库 + 互联网搜索 + 运行环境。上下文是活的，会随着任务进展动态更新。|**全自动闭环 (Autonomous Loop)**<br><br>**Plan -> Execute -> Observe -> Repair**。它拥有独立的沙盒环境，具备更长程的规划能力（Long-horizon Planning）和自我纠错机制。|**甲方**<br><br>(发令者)|

## 1.3 主要工具

- **独立界面：**
    
    - Cursor：
        
    - Trae：
        
    - Qoder：
        
    - ......等等其他
        
- **终端 Agent**
    
    - Claude code：
        
    - CodeX ClI:
        
    - OpenCode: https://github.com/anomalyco/opencode
        
    - ......等等其他
        
- **通用****智能****体**
    
    - manus：
        
    - openHands：
        

https://app.all-hands.dev/conversations/61a83d3f9eed4373880191d84f57fade

- ....其他
    

# 二、Coding重要因素

## 2.1 AI Coding模型

### **2.1.1 SWE-bench Leaderboards (软件工程基准测试排行榜)**

https://www.swebench.com/index.html

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=MDViM2MxOGRmYjU5YjU2NThiMjcxNWRkZTI4MmFlNzBfYTB1ejNTeDM0ZklIZWhsa3dKTmUzUlpjQmRMd0pneVhfVG9rZW46WDRnRWJBcFlkb2w2aTl4ZVRSSmNrQlRGbnlmXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

### **2.1.2 Gemini 3.0** **pro****技术报告：**

https://storage.googleapis.com/deepmind-media/Model-Cards/Gemini-3-Pro-Model-Card.pdf

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWZhOWI5MTU2NmI3OTAyNzFmYjk2ZWY1YWJmZDVlY2NfVHE4R0hXTmJGVlVxQ0JOcnJNMk1Dd2pPa0xSZXZyUlVfVG9rZW46REdkWmJCT3JMb0J6NFl4RlhFT2N0UVlWbnFDXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

  

## 2.2 上下文处理

### 2.2.1 显式上下文

**（1）用户输入上下文完整性。**

**例子：**

**任务目标：** 在 `OrderService.java` 中实现一个 `createOrder` 方法。

**步骤 1：思考依赖链**

在让 AI 写代码前，先想一下这个方法涉及谁？

1. 入参是什么？ -> `OrderCreateReq.java` (DTO)
    
2. 数据存哪？ -> `OrderMapper.java` (数据库接口)
    
3. 有没有工具类？ -> `IdGeneratorUtils.java` (生成订单号)
    

**步骤 2：构建显式上下文 (Prompt)**

在 Cursor 的 Chat (Cmd+L) 中输入：

> 请在 **@OrderService.java** 中实现 `createOrder` 方法。
> 
> **参考上下文：**
> 
> 1. 入参结构看这里：**@OrderCreateReq.java** (确保字段取值正确)
>     
> 2. 数据库操作看这里：**@OrderMapper.java** (确保使用 MyBatis-Plus 风格)
>     
> 3. 订单号生成请复用：**@IdGeneratorUtils.java** 中的 `nextId()` 方法。
>     
> 
> **要求：** 逻辑包括校验库存、计算总价、落库。

**步骤 3：****AI** **的反应 (对比)**

- **如果不给上下文：** AI 可能会写 `order.setOrderId(UUID.randomUUID().toString())` —— **错！** 因为你公司规定用雪花算法，只有给了 `@IdGeneratorUtils` 它才知道。
    
- **给了上下文后：** AI 会精准生成：
    

```Plain
// AI 生成的代码
public void createOrder(OrderCreateReq req) {
    // 1. 引用了正确的 Utilslong orderId = IdGeneratorUtils.nextId();

    // 2. 引用了正确的 DTO 字段
    Order order = new Order();
    BeanUtils.copyProperties(req, order);
    order.setId(orderId);

    // 3. 引用了正确的 Mapper
    orderMapper.insert(order);
}
```

**（2）提问的“关键词密度”**

**输入的 Prompt（提示词）就是****搜索****关键词。**

- **原理：** 隐式检索依赖于你问题中的关键词与代码库的匹配度。
    
- **负面案例：**
    
    - _问：_ “这个功能怎么改？”（太模糊，AI 不知道去哪个文件夹捞数据）。
        
- **正面案例：**
    
    - _问：_ “**UserMapper** 里的 **selectById** 逻辑怎么优化？”
        
    - _影响：_ 明确的类名和方法名会极大提升 RAG 的命中率（Recall）。
        

### 2.2.2 隐式检索上下文

AI 自动去你的代码库里捞数据，以cursor为例，其核心技术是RAG。

**（1）噪音控制：**

- **原理：** RAG 会把项目里的文件切片、向量化。如果你不告诉它“别看垃圾堆”，它就会把垃圾也索引进去。
    
- **负面影响：**
    
    - 当你的 Java 项目里包含 `target/` 目录（编译后的 .class 文件）或 `logs/`（几百 MB 的日志）时。
        
    - **后果：** 你问“哪里报错了？”，AI 检索到了 `app.log` 里的文本，而不是 `OrderService.java` 里的逻辑。这不仅浪费 Token，还会误导 AI。
        
- **关键动作：** **配置** **`.cursorignore`**。
    
    - _必须忽略：_ `target/`, `.git/`, `node_modules/`, `*.json` (如果是大数据文件), `*.log`.
        

**（2）代码的”友好度“**

- **命名规范：**
    
    - _差：_ `public void func1(String s)` -> AI 搜“处理订单”时，根本搜不到这个方法。
        
    - _好：_ `public void processOrder(String orderId)` -> AI 一搜“Order”或“Process”，立马命中。
        
- **注释即锚点 (Comments as Anchors)：**
    
    - 在核心业务逻辑上写 **JavaDoc** 是给 AI 留下的“面包屑导航”。
        
    - _例子：_ 如果你在方法上写了 `/** 处理退款核心逻辑，涉及支付宝回调 */`。
        
    - _效果：_ 当你问“支付宝退款怎么处理的？”，即使方法名里没有 "Alipay"，AI 也能通过注释里的关键词把这个文件“捞”出来。
        

### 2.2.3 规则与系统上下文

**项目级规范：**

`.cursorrules`

- **定义：**`.cursorrules` 是位于项目根目录下的一个纯文本文件（类似于 `.gitignore`）。 Cursor 会在**每一次**对话开始前，自动读取这个文件的内容，并将其作为 **System Prompt（系统提示词）** 的一部分强制灌输给 AI。
    
- **核心价值：** 它解决了 **“重复指令”** 的痛点。你不需要每次都说“我是写 Java 的，用 MyBatis，别用 JPA”，把这些写进规则里，AI 永远不会忘。
    

### 2.2.4 对话上下文

对话历史。

- 定义：用户窗口期内交互信息
    
- 价值：当功能进行切换、或者上下文过长，需要重新开启对话。
    
- 为什么要重新开启对话：
    
    - **注意力衰减 (Attention Decay)：** 当对话历史过长（例如超过 30 轮），模型对中间信息的“注意力权重”会下降，导致忽略之前的约束条件（即 _Lost in the Middle_ 现象）。
        
    - **上下文污染 (Context Pollution)：** 旧的错误代码、临时的变量定义、过时的报错信息会成为“噪声”，干扰 AI 对当前任务的判断。
        

  

**总结：上面的所有工作都在处理上下文质量，即上下文的完整性与上下文的有效度。**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=MjM0ZDJjZmQxMzhmODU3MDg1MjNmZTFjNTVjNDM1MjhfRWlBeVc3a2VXNE5HaXM4aVo5OFZ6ZlhwU1BBRHlqWHlfVG9rZW46Vk56ZGJqUE96b2k0Rkd4aW9nS2N2bFFXbk1jXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

## 2.3 交互能力

### 2.3.1 工作原理：ReAct、 Plan-Execute 、**Multi-Agent**

这是 AI 如何“动脑子”的部分。它不再是简单的“问答机”，而是变成了“行动派”。

- **ReAct** **(Reasoning + Acting) 模式：****https://arxiv.org/abs/2210.03629**
    
    - **定义：** 这是 Agent 的核心循环。即 **“思考 (Thought) -> 行动 (Action) -> 观察 (Observation)”** 的闭环。
        
    - **流程解析：**
        
        - **思考：** 用户让我把 `UserDTO` 转成 JSON。
            
        - **行动：** 调用 `fastjson` 库。
            
        - **观察（关键）：** 发现报错 `ClassNotFoundException`（观察到了环境反馈）。
            
        - **再思考：** 原来项目里只有 `Jackson`，没有 `fastjson`。
            
        - **再行动：** 改用 `ObjectMapper`。
            
    - _一句话总结：_ **ReAct 就是** **AI** **的“试错循环”。**
        
- **Plan-and-Execute (规划与执行) 模式：**
    
    - **定义：** 针对复杂任务的“上帝视角”。先生成 To-Do List，再逐个击破。
        
    - **流程解析：**
        
        - **Planner (规划器)：** 收到任务“重构订单模块”。它不写代码，而是输出一个步骤清单：1. 创建新表；2. 迁移数据；3. 修改 Service；4. 补测试。
            
        - **Executor (执行器)：** 拿着清单，一步步去跑 ReAct 循环。
            
    - _价值：_ 防止 AI 陷入细节（Rabbit Hole）而忘了最终目标。
        
- **Multi-Agent（多****智能****体）**
    
    - * **定义：** 将复杂问题分发给多个具有特定角色（Roles）和工具（Tools）的 **Sub-Agents** 协作完成。
        
    - **核心价值：** 解决 **"Lost in the Middle"** 和 **“能力冲突”**。一个专注于“写代码”的智能体不应该被“扫描漏洞”的繁琐规则干扰。
        
    - **Java 团队模拟：**
        
        - **Main Agent (路由/架构师)：** 负责整体规划与任务分发。
            
        - **Sub-Agent A (Coder)：** 专门负责 Java 实现，拥有 IDE 操作 Skill。
            
        - **Sub-Agent B (Reviewer)：** 专门负责静态代码检查与性能审计。
            
        - **Sub-Agent C (Tester)：** 专门负责编写 JUnit 测试用例并验证覆盖率。
            

**以扩展方式提供（SubAgent）**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=NDAwYzUxM2Y3NmZjZmYyYzAyYjI2OWM0YTlhOWMxNzBfWTJDUVlTbFFrRWI1VVZPSDRjbzdMRTcyUW1vb1d4WWVfVG9rZW46WU1pT2JtZ2ZCb1k0M0N4anFnMWNFS3ZublViXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

### 2.3.2 能力支持：Skill 与 MCP 协议

这是 AI 突破 IDE 限制、操作真实物理世界的“手”。

- **Skill (技能/工具)：**
    
    - https://agentskills.io/home
        
    - **定义：** AI 看的**功能白皮书**。**它告诉 AI**：这个工具叫什么（Name）、能干什么（Description）、需要什么参数（Parameters）。
        
    - **Java 场景：** AI 拥有“查看 JVM 堆栈”或“分析 Maven 依赖树”的特定技能。
        

**技能会自动从以下位置加载：**

每个技能都应是一个包含 `SKILL.md` 文件的文件夹：

- **MCP (Model Context Protocol)：**
    
    - https://modelcontextprotocol.io/specification/draft
        
    - **定义：**一种通用的标准协议，用于连接 AI 模型（Client）与外部数据源/工具（Server）。
        
    - **核心价值：** AI 客户端无需为每个外部工具重写集成代码，企业可以编写私有的 MCP Server，让 AI 安全地访问内部的数据库、文档库或微服务接口，而无需把数据上传到云端。
        

### 2.3.3 交互方式

**（1）Cursor：Code Review 模式（协作式）**

这是目前最成熟、最受开发者信任的模式。

- **工作机制：** AI 作为“结对编程”的伙伴。它生成代码修改建议（Diff），但**不直接提交**。
    
- **核心特点：** * **Tab/Apply 确认制：** 每一行代码的变更都需要人类视觉确认（Accept/Reject）。
    
    - **即时反馈：** 用户可以在 AI 生成过程中随时打断，或者针对某一行代码提出修改意见。
        
- **价值：** 极高的安全性，适合核心业务逻辑开发。
    
- **类比：** 资深架构师带着实习生，实习生写完每一段都得给架构师看一眼。
    

**（2）Claude Code：全负责模式（Agentic / CLI-First）**

这是 Anthropic 推出的基于终端的指令模式，代表了 **Agent 交互** 的兴起。

- **工作机制：** 用户在终端下达一个宏观指令（如：“修复所有 Deprecated 的 API 调用”），Claude Code 会接管终端。
    
- **核心特点：** * **自主探索：** 它会自己 `ls`、`grep`、运行测试，并根据报错自我修正。
    
    - **批量修改：** 它倾向于一次性完成一个完整的 Task（任务），而不是一行代码。
        
    - **半自主：** 它在执行敏感操作前会询问权限，但中间的读写和调试过程是自动连贯的。
        
- **价值：** 极大地解放了“脏活累活”的生产力，适合重构、迁移和复杂 Bug 修复。
    
- **类比：** 一个经验丰富的**外包开发**，你告诉他目标，他自己去折腾，最后给你一个 PR（拉取请求）。
    

**（3）Manus：全自动模式（Fully Autonomous / General Purpose Agent）**

- **工作机制：** 它是更高维度的“端到端”解决。你不必告诉它怎么改代码，甚至不必提供代码库，部署都给你写好了。
    
- **核心特点：** * **目标导向（Goal-Oriented）：** 交互输入通常是“帮我做一个带支付功能的电商后台”。
    
    - **跨域联动：** Manus 不仅会写 Java，还会自己去网上查最新的支付 API 文档、自己去服务器部署 Docker 容器、自己去验证网页 UI。
        
    - **全透明过程：** 你看到的是一个正在自动打字的屏幕或不断跳动的任务列表，直到它告诉你“搞定了，这是地址”。
        
- **价值：** 适合从 0 到 1 的原型开发或非技术人员实现复杂功能。
    
- **类比：** 一个**全栈数字员工**。你不是在和他写代码，你是在给他下达业务指令。
    

  

# 三、cursor的使用

## 3.1 模型的选择

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=NTQ1YThkNzA3ZDI5MjE3MzdkMmU2Nzc2NmIwNmU5MzBfd3pWN2pqc2ZVcExOT0lnYUQ3YmhWaVJkY214a240S1ZfVG9rZW46RDRHR2IxSkxobzdvN3R4T1BtdGNaZFJiblBnXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

## 3.2 上下文构建

### **3.2.1 显示上下文**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=NjQzZTZjOWNhZWVjYmNmYWU5MmIxZmExY2VkMTgyYzVfaklpSjNFbnlrcXJ5Z3c1ckw5U29YcTNVdHhycVhGUUFfVG9rZW46TU10b2JsYlNUbzNFblZ4UFhvcWNWN1c0bmxlXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

- **添加代码块**
    

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=M2ZmZTEzY2Y5N2E1ZTA1MjA5ZWJhZTFkZWE5YjhjMTVfU2lLS1BhN1Mxd1l3VWFhM2lBdXdseWhpMHV5SzdlY3pfVG9rZW46U2s3aWJVcG40b2pJTHZ4RzVmUmNTaWp2bm82XzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

- **Files & Folders (文件与文件夹)**：
    
    - **对应**：**C3 显式上下文**。
        
    - **作用**：精准“喂料”，告诉 AI 必须参考哪些类（如 `OrderMapper.java`）来写代码。
        
- **Docs (文档)**：
    
    - **作用**：引入外部技术文档（如 MyBatis-Plus 官网文档），防止 AI 使用过时的 API。
        
- **Terminals (终端)**：
    
    - **作用**：让 AI 看到你当前的运行报错或日志，属于**纠错闭环**中的观察环节。
        
- **Branch (分支对比)**：
    
    - **作用**：让 AI 分析当前分支与主分支的区别，常用于 Code Review（代码评审）。
        
- **Browser (浏览器)**：
    
    - **作用**：让 AI 实时联网搜索，解决由于模型训练截止日期导致的“知识滞后”问题。
        

### **3.2.2 隐式检索上下文**

```Properties
# 测试 .cursorignore 配置
# 忽略所有 .log 文件
*.log

# 忽略 temp 目录
temp/

# 忽略 node_modules
node_modules/

# 忽略特定的测试文件
*test_ignore*.txt

# 忽略 build 目录下的所有内容
build/
```

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=ODNkZDczNjFjOTQ5MDdkNWU5NWIxM2M0MWZmMjk4ZDhfcmpiaUhka01Ha3AxU0hlYktXRDVYc1o0Q2trRmR3RDdfVG9rZW46QUVtbmJzZk5qb2pnSDd4Rm5MN2NOS091bm5lXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

### **3.2.3 规则与系统上下文**

```Markdown
# 项目代码规范

## 代码风格规则
- 所有 Python 函数必须使用 snake_case 命名
- 所有 Python 类必须使用 PascalCase 命名
- 每个函数都必须有详细的 docstring
- 使用 4 个空格缩进，不使用 tab

## 验证标识
- 为了验证规则是否生效，在生成 Python 代码时请在文件顶部添加：
  # RULES_ACTIVE ✓
```

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=MzVmMTIzN2Y2ODQ5MDA5ZDczZTljMWExMGQ0MGM2YmNfVzlwSjlOczhYaVRmWVNtMTNIckQxZVJqa2tHMlQ5NzRfVG9rZW46RXY5dmJiR1AwbzM2WU94TUZ5Z2NLcGptbkJiXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

### **3.2.4 对话上下文**

## 3.3 工作模式

### **3.3.1 工作原理**

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZThkMDdhZmE2Njk3YTE2OTQwNDMyMjkyNjU5YjdlOWVfMkNobnM0alBXWTYwTmFQRUo0eFA1ZzhSSGphUlE0UTVfVG9rZW46RFZUcmJ2dVRSbzAwWGl4bjN1SmNTRTNTblFkXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

|Cursor 模式|对应的 AI 思考 / 架构范式|底层逻辑与“动脑子”部分的对应关系|
|---|---|---|
|**Agent**|**ReAct** **+ Multi-Agent**|**核心对应：“行动派”与“试错循环”。**<br><br>在后台不断执行“思考 -> 行动 -> 观察”的 **ReAct** 闭环。它能感知环境反馈，若修改后编译报错，会通过“再思考”自主修正。对于复杂任务，它会启动多个 **Sub-Agents** 协作，分别负责搜索、编写和测试，解决能力冲突。|
|**Plan**|**Plan-and-Execute**|**核心对应：“上帝视角”与“To-Do** **List****”。**<br><br>强制模型先进入 **Planner** 阶段，输出完整的改动清单。这种范式的价值在于防止 AI 陷入局部细节 (Rabbit Hole) 而偏离主路径，确保在执行前先达成逻辑共识。这对于大型 Java 项目的模块重构尤为有效。|
|**Debug**|**专项观察型** **ReAct**|**核心对应：“环境反馈驱动”。**<br><br>其 ReAct 循环是从“观察 (Observation)”阶段强制触发的——即捕获到的报错日志。它会根据反馈结果（如 `ClassNotFoundException`）反推逻辑，重新定位缺失的依赖或错误的调用链路，属于诊断型闭环。|
|**Ask**|**纯 RAG (只读问答)**|**核心对应：“语义检索知识库”。**<br><br>它被剥夺了 Action 阶段的“写权限”，仅保留“思考”和基于代码库的语义检索。它不产生试错行为，主要利用 **RAG** 技术从项目索引中提取相关片段来回答“这段代码是什么意思”等咨询类问题。|

### **3.3.2 能力支持**

**SKILL：**

|位置范围|路径 (Path)|说明|
|---|---|---|
|**项目级**|`.cursor/skills/`|仅在当前打开的项目中生效，适合存放该项目特有的业务逻辑或规范。|
|~~**项目级 (Claude 兼容)**~~|~~`.claude/skills/`~~|~~如果你同时使用 Claude Desktop，或希望保持配置通用性。~~|
|**用户级 (全局)**|`~/.cursor/skills/`|在你电脑上打开的所有 Cursor 项目中都会生效（如通用的代码风格、工具库）。|
|~~**用户级 (全局)**~~|~~`~/.claude/skills/`~~|~~在你电脑上所有 Claude 环境中生效。~~|

**MCP****：**

|   |   |   |   |
|---|---|---|---|
|作用范围 (Scope)|操作系统|完整路径 (Path)|说明|
|**全局配置** (Global)|**Windows**|`%USERPROFILE%\.cursor\mcp.json`<br>*(例如 `C:\Users\YourName\.cursor\mcp.json`_)_|在所有项目中生效。|
|**全局配置** (Global)|**macOS / Linux**|`~/.cursor/mcp.json`|在所有项目中生效。|
|**项目级配置** (Project)|**所有系统**|`.cursor/mcp.json`<br>_(位于项目根目录下)_|仅在当前项目生效，方便随 Git 仓库共享给团队。|

  

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGQyMDljOTAzZjA3Y2FiNDc1OTEzZDM3OTEwZTQ0ZmJfNnRQeFJnQ0RXSVBoUGNDNFMzNjV6OGlXSGk5Mk1wOGJfVG9rZW46Q3ZoWWJ1cmZ5b1p1ZUJ4WktxSmNSR25ibk9lXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

**SubAgent:**

https://cursor.com/cn/docs/context/subagents

  

# 四、claude code的使用

## 4.1 模型的选择

Claude code一般支持Anthropic 模型。可通过配置使用其他模型。

## 4.2 上下文构建

### **4.2.1 显式上下文 (Explicit Context)**

**命令添加**

```Plain
/add src/main/java/com/example/dto/OrderCreateReq.java
```

- **效果**：这相当于把这个文件的全部内容强制写入了当前的对话内存（Context Window）。
    

**自然语言引用（Agent 优势）**

这是 Claude Code 最强的地方。你不需要手动找文件路径，只需要说：

> “请读取 **OrderCreateReq** 的定义，然后据此修改 OrderService。”

- **它的反应**：Claude 会自己运行 `ls -R | grep OrderCreateReq` 找到文件路径，然后调用 `read_file` 工具读取内容。它自己完成了“显式构建”的过程。
    

---

### **4.2.2 隐式检索上下文**

> **操作核心**：Claude Code 不依靠近似匹配 (Vector Search)，而是依靠**工具链 (Tool Use)** 和 **忽略规则**。

（1）主动检索 (Active Search)

当你问一个模糊问题（如“哪里用到了雪花算法？”）时，Claude Code 的操作步骤是：

1. **思考**：用户想找雪花算法 -> 关键词可能是 `Snowflake`, `IdGenerator`, `nextId`。
    
2. **执行**：自动在终端运行 `grep -r "Snowflake" .` 或 `grep -r "nextId" .`。
    
3. **读取**：根据搜索结果的文件名，选择性读取文件内容。
    

（2）噪音控制 (`.claudeignore`)

这非常关键！因为 Claude Code 经常运行 `ls` 和 `grep`，如果你的目录里有垃圾，它会被刷屏，导致 Token 爆炸。

- **必须操作**：在项目根目录创建 `.claudeignore` 文件（语法同 `.gitignore`）。
    
- **推荐内容**：
    
- Plaintext
    

```Plain
# 忽略编译产物
target/
build/
dist/

# 忽略日志和大数据
logs/
*.log
*.json
*.csv

# 忽略 IDE 配置
.idea/
.vscode/
```

- **作用**：当你让它“分析整个项目结构”时，它会自动跳过这些目录，把注意力集中在 `src` 目录下。
    

---

### 4.2.3 系统上下文

> **操作核心**：Cursor 用 `.cursorrules`，Claude Code 用 **`CLAUDE.md`**。

这是 Claude Code 特有的“说明书”。当 Claude Code 启动或进入一个目录时，它会优先寻找并阅读 `CLAUDE.md`。

在你的项目根目录（例如 claude-pratice 目录）创建一个名为 `CLAUDE.md` 的文件。

```Plain
# Project: ShareSkill Agent## Build & Test Commands- Build: `mvn clean package -DskipTests`- Run Main: `java -jar target/app.jar`- Test: `mvn test`## Architecture- Framework: Spring Boot 3
- Database: MySQL (MyBatis-Plus)
- AI Modules: LangChain4j

## Code Style Guidelines1. **Naming**: Use CamelCase for classes. Methods must be explicit (e.g., `createOrder` not `add`).
2. **Persistence**: NEVER use JPA. Always use MyBatis-Plus `BaseMapper`.
3. **Logs**: Use `@Slf4j`. Catch exceptions and log with `log.error("Action failed", e)`.
4. **Context**: When writing services, always check related DTOs and Utils first.
```

- **效果**：有了这个文件，你再也不用每次都说“我是用 MyBatis 的”，它在回答你任何问题前，都会先“看一眼”这个备忘录。
    

---

### **4.2.4 会话上下文 (Conversation Context)**

> **操作核心**：Claude Code 提供了专门的**内存管理指令**，这比 Cursor 更硬核。

**`/compact`** **(压缩上下文) - 杀手级功能**

当对话进行很久（比如你修了 10 个 Bug），Token 消耗巨大且 Claude 开始“迷糊”时：

- **操作**：输入 `/compact`
    
- **原理**：Claude 会把自己之前的对话历史进行**摘要 (Summary)**，把 20k Token 的罗嗦对话压缩成 500 Token 的精华（保留结论，丢弃过程），然后**清空旧历史，只保留摘要**。
    
- **场景**：当你完成了一个功能，准备开始下一个，但又想保留一点“记忆”时。
    

**`/clear`** **或** **`/reset`****：**

- **操作**：彻底清空记忆。
    
- **场景**：完全切换任务，比如从“写后端 Java”切换到“写前端 Vue”，为了防止上下文污染，直接重置。
    

`/cost` (成本监控)：

- **操作**：随时查看当前会话消耗了多少 Token 和美金。这能倒逼你优化上下文（比如及时发现自己忘加 `.claudeignore` 导致读了 huge file）。
    

  

## 4.3 工作模式

### 4.3.1 能力支持

（1）SKILL

**个人 Skills** (`~/.claude/skills/`) - 存储在用户目录，所有项目可用

**项目 Skills** (`.claude/skills/`) - 存储在项目中，通过 Git 与团队共享

（2）MCP

https://code.claude.com/docs/zh-CN/mcp

（3）subAgent

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=MmNlOWE3MzYwYWQ1YTU3M2I0YjU0NjllOWVhZTQ3ZjdfMG9iQ2xWdlV6WkxxZldkaTg0M3V4U0tPWjNWZlVSQk1fVG9rZW46TDUwMmJpa1k5b3pjSVh4cGxUUWNDN2pFbnBkXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

每个 sub-agent 都有其自己的：

- 上下文窗口
    
- 系统提示词
    
- 工具和任务
    

  

https://code.claude.com/docs/zh-CN/sub-agents

[https://github.com/wshobson/agents](https://github.com/wshobson/agents)

|   |   |   |   |
|---|---|---|---|
|类型|路径|适用场景|优先级|
|**项目级**|`.claude/agents/*.md`|**团队协作**。配置随代码库提交，所有成员共享。|最高 🥇|
|**用户级**|`~/.claude/agents/*.md`|**个人工具箱**。跨项目通用的私人助手。|中等 🥈|
|**插件级**|(由插件提供)|第三方扩展提供的通用能力。|最低 🥉|

# 五、manus的使用

  

# 六、总结

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=YTMwZWEwNzIxMThiMDdjMGQ0NDVkY2Y5ZjJjYTY5N2ZfYlZDcHR1cjdKem1CTUt5enFuazdMN0lWUnR6V3ZQclpfVG9rZW46QTRLSmJLbU44b0dRU3Z4U2VqOGNWdXp6bmNiXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)

  

![](https://hf7l9aiqzx.feishu.cn/space/api/box/stream/download/asynccode/?code=YTk0ODgwOTBkNTkzYWM0MjgzZjU1YTdiYzgyNzQzNDVfaFBrYXQwOXV3U014RTc3bFJINzgyeWxkUENTZmdnQVlfVG9rZW46Qk03dmJyaUV5b1dtSWd4YjdCSmNnWHZ4bnpkXzE3NzU2NDAzMTU6MTc3NTY0MzkxNV9WNA)
# MemoryOS of AI Agent
参考资料：[[2025.emnlp-main.1318.pdf]]

> [!abstract]
> MemoryOS 是一个受操作系统内存管理启发的 AI 智能体分层记忆框架。它通过 `Storage`、`Updating`、`Retrieval`、`Response Generation` 四个模块，统一管理短期、中期和长期记忆，以提升长对话中的连贯性、历史信息利用能力与个性化适配能力。

## 一、研究背景与问题

大型语言模型（LLMs）依赖固定长度的上下文窗口进行推理与生成，因此天然存在以下限制：

- 难以维持长时间跨度对话的连贯性
- 容易出现事实不一致
- 个性化记忆不足
- 多会话知识留存能力弱
- 角色设定稳定性不足

现有记忆机制通常可分为三类：

- 知识组织类
- 检索机制导向类
- 架构驱动类

但这些方法大多只关注某一个维度，例如存储结构、检索方式或更新策略，缺少一个像操作系统一样统一协调的内存管理框架。MemoryOS 的目标，就是为 AI 智能体提供这种系统化、全面化的记忆管理能力。

## 二、核心方案：MemoryOS 设计

MemoryOS 通过四个核心模块协同工作：

- `Memory Storage`
- `Memory Updating`
- `Memory Retrieval`
- `Response Generation`

整体目标是实现：

- 分层存储
- 动态更新
- 自适应检索
- 面向生成的上下文组装

---

## 三、内存存储模块（Memory Storage）

MemoryOS 采用三级分层存储结构：`STM`、`MTM`、`LPM`。

### 1. 短期记忆（STM）

短期记忆用于存储实时对话数据，以“对话页（page）”为基本单位。

单个对话页可表示为：

$$
p_i = (Q_i, R_i, T_i)
$$

其中：

- $Q_i$：用户查询
- $R_i$：模型响应
- $T_i$：时间戳

为了维持短期连续对话的上下文关联性，系统还会构建对话链。链页面包含由 LLM 生成的元信息：

$$
\mathrm{meta}^{\text{chain}}_i
$$

该元信息主要用于：

- 判断语义连续性
- 总结链内页面内容

### 2. 中期记忆（MTM）

中期记忆采用“段页式”存储结构，将主题相近的多个对话页组织为一个“段（segment）”。

系统通过页面与段之间的相似度分数判断某个对话页是否应归入某个段：

$$
\mathcal{F}_{\text{score}}(p, s) = \cos(e_s, e_p) + \frac{|K_s \cap K_p|}{|K_s \cup K_p|}
$$

其中：

- $e_s$：段的语义向量
- $e_p$：对话页的语义向量
- $K_s$：段的关键词集合
- $K_p$：对话页的关键词集合

当相似度分数超过阈值时，页面被归入对应段：

$$
\mathcal{F}_{\text{score}}(p, s) > \tau_s
$$

实验中通常取：

$$
\tau_s = 0.6
$$

段内内容可由 LLM 自动总结，形成更高层级的主题表征。

### 3. 长期个人记忆（LPM）

长期个人记忆用于存储用户与智能体的核心个性化信息，主要分为两部分。

#### 用户画像

包含：

- 静态属性：如性别、姓名等
- 动态用户知识库 `User KB`：存储从交互中提取出的事实信息
- 用户特质：共 90 个维度，覆盖基本需求、个性特征、AI 对齐维度与内容平台兴趣标签

#### 智能体画像

包含：

- 固定角色设定 `Agent Profile`
- 动态智能体特质 `Agent Traits`

其中 `Agent Traits` 用于存储用户新增设定、长期交互历史中沉淀下来的偏好与行为线索。

---

## 四、内存更新模块（Memory Updating）

MemoryOS 会在不同存储层级之间进行动态迁移，以维持整体记忆系统的有效性。

### 1. STM → MTM

STM 使用固定长度队列保存对话页，并遵循先进先出（FIFO）原则。

当队列达到上限时，最旧的对话页会被迁移到 MTM。

### 2. MTM → LPM

MTM 中的段通过“热度评分（Heat）”进行管理。热度由以下三类因素共同决定：

- 检索次数 $N_{\text{visit}}$
- 段内对话页数量 $L_{\text{interaction}}$
- 时间衰减项 $R_{\text{recency}}$

热度计算公式为：

$$
\mathrm{Heat} = \alpha N_{\text{visit}} + \beta L_{\text{interaction}} + \gamma R_{\text{recency}}
$$

其中时间衰减项定义为：

$$
R_{\text{recency}} = \exp\left(-\frac{\Delta t}{\mu}\right)
$$

参数含义：

- $\alpha, \beta, \gamma$：权重系数
- $\Delta t$：距离上次访问或激活的时间间隔
- $\mu$：时间衰减控制参数

当某个段的热度超过阈值时，会被迁移到 LPM：

$$
\mathrm{Heat} > \tau_h
$$

实验中通常取：

$$
\tau_h = 5
$$

迁移后，系统会据此更新：

- `User KB`
- 用户特质
- `Agent Traits`

如果 MTM 中段的数量超过容量上限，则会淘汰热度最低的段。

### 3. LPM 更新

`User KB` 和 `Agent Traits` 采用固定长度队列管理，实验中容量通常设置为：

$$
C_{\text{LPM}} = 100
$$

整体更新策略依然遵循 FIFO。

此外，用户特质可通过 LLM 从迁移进来的段与对话页中持续抽取信息，实现自主演化。

完成迁移后，对应段的访问计数会被清零：

$$
N_{\text{visit}} \leftarrow 0
$$

这会使其热度随之下降，避免重复高频晋升。

---

## 五、内存检索模块（Memory Retrieval）

当用户发起查询时，系统会从三个层级的记忆中联合检索相关信息。

### 1. STM 检索

STM 检索策略最直接：

- 直接获取当前所有对话页
- 保留最新上下文
- 优先保证局部对话连贯性

### 2. MTM 检索

MTM 检索分为两个阶段。

第一阶段，从所有段中筛选与查询最相关的 `Top-m` 个段：

$$
\mathcal{S}_{\text{top}} = \operatorname{Top\text{-}m}\; \mathcal{F}_{\text{score}}(q, s)
$$

实验中常取：

$$
m = 5
$$

第二阶段，在候选段内部继续筛选 `Top-k` 个最相关对话页。

不同数据集中的典型设置：

- `GVD`: $k = 5$
- `LoCoMo`: $k = 10$

每次检索后，还会更新该段的访问统计与最近访问时间，例如：

$$
N_{\text{visit}} \leftarrow N_{\text{visit}} + 1
$$

$$
T_{\text{last}} \leftarrow T_{\text{now}}
$$

### 3. LPM 检索

LPM 检索面向个性化信息。

系统会分别从以下模块中检索与当前查询最相关的条目：

- `User KB`
- `Agent Traits`

通常各取语义相关性最高的 `Top-10` 条记录，同时结合：

- 用户画像中的核心信息
- 智能体画像中的核心信息

以保证生成结果具有稳定的个性化特征。

---

## 六、响应生成模块（Response Generation）

最终，系统会将来自 `STM`、`MTM`、`LPM` 的检索结果与当前用户查询统一组装为提示词，再输入 LLM 生成响应。

这一过程确保输出同时具备：

- 上下文连贯性：主要依赖 `STM` 与 `MTM`
- 历史信息深度：主要依赖 `MTM`
- 个性化适配：主要依赖 `LPM`

可以将其概括为：

$$
\mathrm{Prompt} = f(q, \mathrm{STM}, \mathrm{MTM}, \mathrm{LPM})
$$

其中 $q$ 表示当前用户查询。


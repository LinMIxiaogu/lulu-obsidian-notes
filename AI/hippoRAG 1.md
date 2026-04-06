## 一、RAG 的核心价值与现状痛点

- 定义 RAG：用 “外部检索 + 模型生成” 让大模型不用重新训练也能获得最新、最准确的知识，是现在企业里最常用的大模型增强方案。
    
- 现状痛点：现有 RAG 还不够聪明，在 “事实记忆、情境理解、知识关联性” 这三件事上很难同时做好。
    

## 二、RAG 技术迭代：从基础到进阶的三大阶段

### 1. 第一阶段：传统 RAG（向量 / 关键词检索）
    

- 核心逻辑：基于 BM25（关键词匹配）或 Contriever、NV-Embed-v2（向量语义匹配），直接从语料中检索相似片段。
    
- 优势：简单高效、工程落地成本低，擅长单跳事实查询（如 “某人的职业是什么”）。
    
- 致命缺陷：无法模拟人类记忆的核心特质 ——
    
    - 缺乏**情境理解能力**：难以整合长文本、复杂叙事（如理解小说剧情、多文档逻辑）；
        
    - 缺乏**知识关联能力**：无法完成多跳推理（如 “Erik 的出生地属于哪个县” 需要关联 “出生地→县” 两跳信息）。
        

### 2. 第二阶段：结构增强型 RAG（RAPTOR/GraphRAG）
    

- 迭代思路：给无结构语料添加 “结构化信息”，弥补传统 RAG 的能力短板。
    
- 主流方案与局限：
    
    - RAPTOR：用 LLM 生成摘要构建多层级语义树，提升长文本理解，但摘要引入噪声，事实查询性能下降；
        
    - GraphRAG：用知识图谱（KG）链接实体与段落，但图谱扩展导致冗余，多跳推理效率低；
        

## 三、HippoRAG

- HippoRAG—— 一种受人类长期记忆的**海马体索引理论**启发的新型检索框架。
    
    - 海马体记忆索引理论提出，人类长期记忆由三个组件构成，它们协同工作以实现两个主要目标：模式分离和模式补全
        
    - 模式分离：新皮质将感知信息加工成更高层级特征，经海马旁回（PHR）送入海马体建立索引并形成关联。
        
    - 模式补全：当海马体通过 PHR 收到部分线索时，模式补全会启动检索，由海马体恢复完整记忆。
        
- HippoRAG 协同协调 LLMs、知识图谱和个性化 PageRank算法，以模拟新皮质和海马体在人类记忆中的不同作用。在多跳问答（QA）任务上表现优异。
    

### 传统 RAG 的痛点

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2EyZGIyZjU2ZGZkZDBkZDVjMGY3MWYwNmQxYjU1NDlfSEdDSHJBcTEwVFo3b2JMSFgwdk94VVBZOUNZbzVDaUpfVG9rZW46UzVXUWJ0YXNOb25YY254YjI0aWNMS0p0bm9jXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

### 任务场景

目标：从包含数千名斯坦福教授和阿尔茨海默病研究者的语料中，找出 “**研究阿尔茨海默病神经科学的斯坦福教授**”（答案：Thomas 教授）。

关键难点：没有任何一个段落同时提到 “斯坦福教授” 和 “研究阿尔茨海默病” 这两个信息，需跨段落关联分散知识。

|   |   |
|---|---|
|**主体**|**表现 / 原理**|
|Current RAG（现有 RAG）|失败。因为现有 RAG 孤立编码每个段落，无法关联分散在不同段落的 “斯坦福雇佣 Thomas” 和 “Thomas 研究阿尔茨海默病” 这两个信息，除非有段落同时包含两个特征。|
|Human Memory（人类记忆）|轻松成功。依赖大脑的联想记忆能力 —— 由海马体（图中蓝色 C 形结构）的索引结构驱动，能快速关联分散知识。|
|HippoRAG|成功。模仿人类海马体的索引机制，构建类似的关联图谱，让 LLM 具备跨段落知识整合能力，从而精准定位 Thomas 教授。|

### HippoRAG 核心流程

#### 人类记忆系统的“三大组件”（理论基础）

标注了人类长期记忆的三个核心部分，是 HippoRAG 的设计灵感来源，对应关系如下：

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=NmI3YTNjNmMzM2RiN2E5ZmQ5MzQ2MTc5Yzc5NGQ5MDJfOHB3QW1DWEJQUmw1TE9Ob2w3YVZrSWZpQjdzeWxwUkJfVG9rZW46UkQxVWJzcnp6b04zVEx4bVNlVmNxeWlLblJiXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

|   |   |   |
|---|---|---|
|**人类记忆组件**|**HippoRAG 中替代模块**|**核心功能**|
|新皮质（Neocortex）|指令微调 LLM|从文本中提取命名实体和知识三元组，完成 “模式分离”|
|海马旁回区域（PHR）|检索编码器|计算实体相似度，添加同义词边（如 “Stanford” 与 “Stanford University”），避免关联断裂|
|海马体（Hippocampus）|知识图谱（KG)+个性化 PageRank（PPR）|通过 KG 记录实体关联，用 PPR 完成 “模式补全”（从部分查询线索召回完整关联）|

#### 离线索引阶段（Offline Indexing）

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjNkNTU2MzNjNjUwOTU2MmYwNTk0NzY0YTYxY2YyM2VfeDBydFp6RWtzZU03Um8wcnBqejFIdG14eVdJcmc0V3JfVG9rZW46R1M2VGJhSFJ5bzU2aHp4QUc2YmNCdkFsbkhjXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

这一阶段对应人类 **“记忆编码”** 过程，核心是将无结构文本转化为结构化的 “人工海马体索引”（即知识图谱），流程如下：

1. **输入**：检索语料中的原始段落（如包含 “Thomas 研究阿尔茨海默病”“斯坦福雇佣 Thomas” 的分散段落）；
    
2. **LLM处理**：通过OpenIE（开放信息抽取） 提取三元组（如`(Thomas, researches, Alzheimer’s)`、`(Stanford, employs, Thomas)`），同时记录每个实体（节点）在哪些段落中出现（构建`|节点数|×|段落数|`矩阵）；
    
3. **检索编码器**：计算 KG 中实体的余弦相似度，对相似度＞0.8 的实体添加 “同义词边”（如 “Alzheimer’s” 与 “阿尔茨海默病”），丰富图谱关联；
    
4. **输出**：无模式知识图谱（人工海马体索引），包含所有实体、关系边（三元组 + 同义词边）及实体 - 段落关联记录。
    

#### 在线检索阶段（Online Retrieval）

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=NjgzM2E1MzlhZmZkYmFkODBmYTA5ODkyYmYwNTVjMGZfYWw4dWhQcUhVeVNSNVhQMWJvV2hoTkx4TnNjd1dkTUhfVG9rZW46SGdOMGJlOXRtb1JRRVd4ZUo5WmNiUHM2bm9mXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

这一阶段对应人类 **“记忆检索”** 过程，核心是从 KG 中快速召回与查询相关的段落，流程如下：

1. **输入**：用户查询（如 “Which Stanford professor works on the neuroscience of Alzheimer’s?”）；
    
2. **LLM提取线索**：通过 NER（命名实体识别）提取查询中的关键实体（如 “Stanford”“Alzheimer’s”）；
    
3. **检索编码器匹配节点**：将查询实体与 KG 中的节点匹配（如 “Stanford” 对应 KG 中的 “Stanford” 节点），确定 “查询节点”；
    
4. **PPR模式补全**：以查询节点为种子，运行 PPR 算法在 KG 中扩散概率（如 “Stanford”→“Thomas”→“Alzheimer’s”），找到高关联的实体节点；
    
5. **段落排序输出**：将 PPR 得到的节点概率与 “实体 - 段落矩阵” 相乘，得到每个段落的评分，按评分召回 Top-K 段落（如包含 Thomas 的相关段落）。
    

#### PPR 算法

#### PageRank（网页排名算法）

一个网页的重要性，由指向它的其他网页**数量**和这些 “投票网页” 自身的**重要性**共同决定。（详细见附录）


**PPR 与标准 PageRank 的区别**：

- PPR 是 PageRank 的变体，仅通过用户定义的 “源节点” 在图中分配概率。 —— 标准 PageRank 对全图节点均匀分配初始概率。
    

**PPR 在 HippoRAG 中的适配设计：**

- **初始概率分配：**节点特异性定义为节点出现段落数的倒数，PPR 前需将查询节点的初始概率与节点特异性相乘。——这一调整解决了原始 PPR “对高频节点过度激活” 的问题
    
- **概率传播后的段落映射**：PPR 得到节点概率分布后，与 “节点 - 段落出现矩阵 P” 相乘，得到段落排序分数。
    
    - 这一步是 PPR 从 “节点级推理” 到 “段落级检索” 的关键衔接 —— 原始 PPR 仅输出节点分数，HippoRAG 通过矩阵乘法将节点概率映射到段落。
        

- 初代 HippoRAG：KG + 个性化 PageRank（PPR）算法提升关联性，但 “**实体中心主义**” 丢失上下文，**情境理解弱。**
    

## 四、HippoRAG2

### 横向维度：三大评估任务（对应人类记忆核心特质）

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=MWZhNDJlZDI3MDdhOWVhOGI1MWVkODZhMGE4ZWY5OThfYW41VTRMOFZUSm1vOHlmellJWGZMaThRUHpiaWI1amZfVG9rZW46S3hIbmJzMldOb3Q2Rkh4NGFVTGNFTENSbnBkXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

|   |   |
|---|---|
|**任务类型**|**三行关键事实**|
|**事实记忆**|提供 3 个 “人物 - 职业” 基准事实（如 “George Rankin is a politician”），明确任务是 “单跳事实检索”，**验证模型能否精准定位这类独立事实。**|
|**情境理解**|提供 3 个 “事件链” 基准事实（如 “灰姑娘参加舞会→王子用玻璃鞋寻人→两人重逢”），明确任务是 “长文本逻辑整合”，**验证模型能否串联多事件语境。**|
|**知识关联性**|提供 3 个 “多跳推理链” 基准事实（如 “埃里克出生地是 Montebello→Montebello 属于 Rockland 县”），明确任务是 “跨知识关联”，**验证模型能否完成两跳推理。**|

### HippoRAG 2 的三大创新点

#### 1. 创新一：疏密整合—— 概念与语境的无缝融合
    

##### 核心问题

传统 RAG 和初代 HippoRAG 仅关注 “概念（短语 / 实体）”，丢失了概念所属的 “语境（段落）”，导致检索时信息不完整。例如，仅检索 “Montebello” 这个短语，无法知晓它是 “Erik Hort 的出生地” 还是其他地理实体。

##### 技术实现

- 稀疏编码（概念层）：将段落中的核心概念提取为 “短语节点”（如 “Erik Hort”“Montebello”“born in”），对应人类记忆中对核心信息的精简存储。
    
- 密集编码（语境层）：将完整段落作为 “段落节点”，保留概念的原始上下文（如包含 “Erik Hort 出生于 Montebello” 的完整句子）。
    
- 关联设计：新增 “包含” 关系边，将每个段落节点与从中提取的所有短语节点关联，形成 “短语 - 段落” 双向链路。
    
- 补充优化：检索编码器检测短语间同义词（如 “born in” 与 “birthplace”），添加 “同义词边”，实现跨段落知识关联。
    

##### 直观效果

构建的 KG 包含两类节点（短语节点 + 段落节点）和三类边（关系边、同义词边、上下文边），检索时既能精准定位核心概念，又能回溯完整语境，避免信息丢失。

#### 2. 创新二：深度上下文关联—— 精准捕捉查询意图
    

##### 核心问题

初代 HippoRAG 采用 “NER 到节点” 的匹配方式（提取查询中的实体如 “Erik Hort”，再匹配 KG 中的实体节点），忽略了查询的完整语义（如 “出生地属于哪个县” 的逻辑关系），导致检索与意图错位。

##### 技术实现

- 三种匹配方案对比：
    
    - NER to Node（初代方案）：提取查询实体→匹配 KG 实体节点，仅捕捉单个概念；
        
    - Query to Node：将**整个查询**与 KG 短语节点做向量匹配（查询是完整句子，节点是短语）；
        
    - Query to Triple（HippoRAG 2 默认方案）：将**整个查询**与 KG 中的三元组做向量匹配，直接捕捉概念间关系。
        

##### 举例说明

查询：“What county is Erik Hort’s birthplace a part of?”（Erik Hort 的出生地属于哪个县？）

- NER to Node：仅提取 “Erik Hort”，匹配到相关实体节点，但无法关联 “出生地→县” 的逻辑；
    
- Query to Triple：直接匹配 KG 中的三元组（Erik Hort, born in, Montebello），精准捕捉 “人物 - 出生地” 关系，为后续多跳推理奠定基础。
    

#### 3. 创新三：识别记忆 —— 过滤噪声，优化种子节点
    

##### 核心问题

向量检索会返回大量与查询弱相关的三元组，若直接作为 PPR 算法的种子节点，会导致检索偏差（如查询 “出生地所属县”，返回 “Erik Hort 的职业”“出生日期” 等无关三元组）。

##### 技术实现（论文 Figure 4 图示支撑）

模拟人类记忆的 “识别” 过程，分两阶段过滤：

1. 初步检索：用嵌入模型从 KG 中检索与查询最相关的 Top-k 三元组（默认 k=5）；
    
2. LLM 过滤：用 Llama-3.3-70B-Instruct 作为 “识别记忆过滤器”，根据查询意图筛选有效三元组，输出`T' ⊆ T`（筛选后的三元组子集）。
    

##### 举例说明（论文 Figure 5 案例）

查询：“What county is Erik Hort’s birthplace a part of?”

- 初步检索得到 5 个三元组：（Erik Hort, born in, Montebello）、（Erik Hort, born in, New York）、（Erik Hort, is a, American）、（Erik Hort, born on, February 16, 1987）、（Erik Hort, is a, Soccer player）；
    
- LLM 过滤后保留 2 个核心三元组：（Erik Hort, born in, Montebello）、（Erik Hort, born in, New York），过滤掉职业、出生日期等无关信息。
    

#### 4. 完整流程拆解：HippoRAG 2 的 “离线索引 + 在线检索”
    

#### 离线索引：构建人类“海马体”索引

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=YWQzZWNjNzU3NzlhMmFjZTExMjlhZjUyN2QzYWQ1ODdfT05FUmJXajRaZDVPdHdWQXFyUjlhSkpUWXY3S0t3VWdfVG9rZW46VUhEcWJrY2F4b1FpWXh4Y2NURGNlTGQwbnllXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=NGQ0MDE5NjM0OTZmZGI4Y2VkMzQ5ZTYwNTZmMzU5MDdfcEFiV1ZnWlh3djJkaVZZQldlSG11bkhkSFV5MmEzSDlfVG9rZW46WEdzWmJuWko0b3FYTTF4dExFVWNKR3pqbmlkXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

#### 在线检索：模拟人类记忆提取（论文 Figure 5 完整案例）

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=NjhjNGNlZDE1MTQ4MTcyYjU0ODFlYmRhMGQ4Yjg4ZWNfMXF0NjRzWHpCdkdESnRQOHlEQ0lxaUJvaXJjN2g4T3NfVG9rZW46RmdIUmJoNnA1b3BOWUd4SlFxMGNUcTdPbjdjXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

  

![](https://lcn83t7qqhto.feishu.cn/space/api/box/stream/download/asynccode/?code=ODIzNWZmMDczNDA0M2UyYzM2NTg0NWQ1NGQ2Y2MwM2FfSHJVZzdielhDQkJZQVNiamViSlJnd05JRTlzRlp0aGdfVG9rZW46Q013aGJVcU9YbzFZaDB4RnZGMGNld1kxbmpjXzE3NzU0NTYxOTY6MTc3NTQ1OTc5Nl9WNA)

以查询 “What county is Erik Hort’s birthplace a part of?” 为例，分步拆解：

1. 查询关联：用嵌入模型匹配 KG 中的三元组和段落，得到 Top-5 三元组和相关段落；
    
2. 识别记忆过滤：LLM 筛选出 2 个核心三元组（见创新三案例）；
    
3. 种子节点确定：
    
    1. 短语节点：从过滤后的三元组中提取（Montebello：1.0 分，Erik Hort：0.995 分，New York：0.989 分）；
        
    2. 段落节点：所有相关段落节点（如 “Erik Hort”“Montebello, New York” 等），权重因子设为 0.05（论文 Table 5 优化结果）；
        
4. PPR 算法运行：以种子节点为起点，多跳遍历 KG，计算节点概率权重（深色节点表示权重高）；
    
5. 段落排序：按 PageRank 得分排序，Top-3 关键段落：
    
    1. 段落 1：Erik Hort 的基础信息（确认出生地为 Montebello）；
        
    2. 段落 3：Montebello, New York 的地理信息（明确其属于 Rockland County）；
        
6. QA 生成：将 Top-5 段落输入 LLM，生成最终答案 “Erik Hort 的出生地 Montebello 属于美国纽约州的 Rockland County”。
    

  

  

## 附录：

### 参考论文：

[[HippoRAG_Neurobiologicall.pdf ]]
[[Topic-Sensitive PageRank 2.pdf]]
[[From RAG to Memory.pdf]]

### PR 算法：PageRank（网页排名算法）

PageRank 是 Google 搜索引擎的核心基础算法，由 Larry Page 和 Sergey Brin 于 1996 年在斯坦福大学提出，核心目标是**通过网页间的链接结构评估网页的相对重要性**，从而优化搜索结果排序。

#### 1. 核心思想：“链接即投票，权重看投票者质量”
    

PageRank 的本质是模拟 “随机冲浪者” 在网页间的跳转行为，其核心逻辑可概括为两点：

- 一个网页的重要性，由**指向它的其他网页数量**和**这些 “投票网页” 自身的重要性**共同决定。例：若 “网页 A” 被 10 个网页链接，且其中 5 个是高权重网页（如百度首页、知乎官方页），则 “网页 A” 的重要性会远高于被 10 个低权重网页链接的 “网页 B”。
    
- 避免 “孤立网页” 或 “链接闭环” 导致的计算失效：引入**阻尼因子（α，通常取 0.85）**，表示 “随机冲浪者有 α 概率继续点击当前网页的出链，有 (1-α) 概率随机跳转到任意网页”，确保算法收敛到唯一稳定的排名向量。
    

#### 2. 数学定义与计算逻辑
    

##### （1）基础迭代公式

设网页总数为 N，`Rank(v)`表示网页 v 的 PageRank 值，`B(v)`表示所有指向 v 的网页集合（“入链网页集合”），`N(u)`表示网页 u 的出链数量（即 u 指向其他网页的总数），则迭代公式为：Ranki+1(v)=∑u∈B(v)N(u)Ranki(u)

- 初始时，所有网页的`Rank`值均为`1/N`（均匀分布）；
    
- 反复迭代上述公式，直到`Rank`向量的变化小于预设阈值（如 1e-6），此时的向量即为最终的 PageRank 排名。
    

##### （2）矩阵形式（大规模计算优化）

为适配海量网页（如数十亿级）的计算，PageRank 可通过**转移矩阵**表示：

- 定义转移矩阵`M`（N×N）：若网页 j 有链接指向网页 i，且 j 的出链数为`N(j)`，则`M[i][j] = 1/N(j)`；否则`M[i][j] = 0`。
    
- 引入阻尼因子后，矩阵更新为`M' = (1-α)M + α·(1/N)·E`（其中`E`是全 1 矩阵），此时 PageRank 向量`Rank*`是`M'`的**主特征向量**（满足`Rank* = M'·Rank*`），对应 “随机冲浪者” 的长期稳定访问概率。
    

#### 3. 关键扩展：Topic-Sensitive PageRank（主题敏感 PageRank）
    

原始 PageRank 是 “全局通用” 的（不考虑查询主题），可能导致 “高链接网页在无关主题查询中误排靠前”（如体育类高权重网页在医疗查询中排名过高）。文档提出的**主题敏感 PageRank**通过以下改进解决该问题：

- **离线预计算多主题排名向量**：基于 Open Directory Project（ODP，开放目录项目）的 16 个顶级分类（如 Arts、Health、Computers），为每个主题生成一个 “偏向性 PageRank 向量”。具体做法：将主题对应的 ODP 网页集合作为 “偏好集合”，调整阻尼因子的跳转概率（仅跳转到该主题的网页，而非全局网页），计算该主题下的 PageRank。
    
- **查询时动态组合向量**：针对用户查询（或查询上下文，如 “在医疗文档中搜索‘blues’”），通过**单字语言模型**计算查询与 16 个主题的相似度，再以相似度为权重，将对应主题的 PageRank 向量线性组合，得到 “查询专属的重要性分数”，确保排名与主题高度相关。
    

#### 4. 应用场景
    

- 搜索引擎结果排序（如 Google 早期核心逻辑）；
    
- 网页重要性评估、垃圾网页识别（低权重网页更可能是 spam）；
    
- 扩展到社交网络（如评估用户影响力）、学术论文引用排名（如 Google Scholar）
    

### RAPTOR

### 核心原理

RAPTOR 的核心是自下而上构建多层级语义树，将无结构文本转化为结构化的树状索引，检索时可自上而下匹配多粒度上下文，解决传统 RAG 处理长文本时的碎片化、语义割裂问题，同时支持跨文本块的复杂推理与信息整合。

### 实现步骤（以长文档分析为例）

1. **文本切分与嵌入**：将原始长文档（如法律合同、学术论文）切分为细粒度文本块（Chunks），计算每个文本块的向量嵌入，作为树的 “叶子节点”。
    
2. **聚类与摘要生成**：用聚类算法（如高斯混合模型 GMM）对叶子节点做软聚类（单个节点可属于多个聚类），调用 LLM 为每个聚类生成摘要，作为 “中间层节点”，提炼该组文本块的核心语义。
    
3. **递归构建语义树**：重复 “聚类 - 摘要” 步骤，对上层中间节点继续抽象，直至生成一个涵盖全文档核心信息的 “根节点”，形成完整的分层语义树。
    
4. **分层检索与答案生成**：查询时，先匹配根节点与中间层节点定位大致范围，再精准检索对应叶子节点获取细节，结合多粒度上下文生成答案，适配复杂问题与长文本场景。
    

### 优势、局限与适用场景

|   |   |
|---|---|
|**维度**|**具体说明**|
|核心优势|解决长文本碎片化问题，关联跨文本块语义；适配多步骤推理、跨章节信息整合等复杂任务；无需超大参数量嵌入模型，轻量模型即可实现高精度检索|
|主要局限|LLM 生成摘要易引入噪声，导致简单事实查询性能下降；构建树结构需额外计算资源，递归聚类与摘要过程耗时|
|适用场景|金融 / 法律长文档分析（年报、合同）、学术论文深度问答、企业多文档关联知识库、技术手册细节检索与整合等|

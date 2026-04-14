# 意图识别 SFT 训练笔记

## 1. 数据结构

### 1.1 原始数据的拼接处理

**原始数据来源：** `data/train.json`

每条数据格式如下：
```json
{
  "text": "[{\"type\": \"human\", \"content\": \"我要去北京\"}, {\"type\": \"ai\", \"content\": \"好的请问需要什么帮助\"}, {\"type\": \"human\", \"content\": \"帮我订机票\"}]",
  "label": "transportation"
}
```

注意：`text` 字段本身是一个 JSON 字符串，需要二次解析。

**`__getitem__` 处理流程（`src/data_processor.py` 第55~128行）：**

**Step 1 — 解析对话列表**
```python
messages = json.loads(raw_text)
# 得到：[{type:human, content:我要去北京}, {type:ai, content:好的...}, {type:human, content:帮我订机票}]
```

**Step 2 — 找最后一条 human 消息作为当前输入**
```python
# 从后往前遍历，找 type == "human" 的最后一条
current_msg = "帮我订机票"
history_msgs = 前面所有消息
```

**Step 3 — 构造历史文本**
```python
history_text = "用户：我要去北京 助手：好的请问需要什么帮助"
```

**Step 4 — Tokenize**
```python
# 有历史：
tokenizer(text=history_text, text_pair=current_msg)

# 无历史（单轮对话）：
tokenizer(text=current_msg)
```

**最终喂给模型的格式：**
```
[CLS] 用户：我要去北京 助手：好的请问需要什么帮助 [SEP] 帮我订机票 [SEP]
```

---

### 1.2 为什么用 text_pair 而不是直接拼接

**直接拼接：**
```
[CLS] 用户：我要去北京 助手：好的 帮我订机票 [SEP]
  A    A  A  A  A  A   A  A  A   A  A  A  A
```

**text_pair 方式：**
```
[CLS] 用户：我要去北京 助手：好的 [SEP] 帮我订机票 [SEP]
  A    A  A  A  A  A   A  A  A    A       B  B  B    B
```

**核心原因：Segment Embedding**

BERT 原生支持双句输入，内部有 Segment Embedding 机制：
- **A 段** = 历史上下文
- **B 段** = 当前用户消息（需要分类的主体）

BERT 通过 Segment Embedding 明确知道哪些 token 是"背景"，哪些是"要分类的主体"，让模型更容易把注意力集中在当前消息上。

直接拼接所有 token 都是 A 段，模型无法区分历史和当前消息，理解起来更难。

**类比：**
- 直接拼接 = 把历史邮件和新邮件混在一起
- text_pair = 用分隔线把历史和新邮件分开，一眼就知道哪部分是需要处理的新内容

---

## 2. 训练参数配置

### 2.1 dropout 的作用与设置

**问题：dropout=0.1 是什么意思，为什么能防过拟合？**

Dropout 在训练时**随机丢弃一部分神经元**（置为0），让模型不依赖某几个特定神经元，强迫它学习更鲁棒的特征。

```
没有 Dropout：
[CLS向量] → Linear → logits

有 Dropout(0.1)：
[CLS向量] → 随机丢弃10%的值置为0 → Linear → logits
```

- `0.1` 表示每次前向传播随机把 **10% 的神经元置为0**
- 只在**训练时**生效，推理时自动关闭（`model.eval()` 后关闭）
- 本项目的 dropout 只加在**分类头前**，BERT 主干内部也有自己的 dropout

---

### 2.2 max_length 与 hidden_size 的区别

**问题：max_length=512 和 hidden_size=1024 都和模型有关，它们是同一个概念吗？**

两者是完全不同维度的概念：

| 参数 | 含义 | 类比 |
|------|------|------|
| `max_length=512` | 输入序列的最大 **token 数量**（长度） | 一篇文章最多写512个字 |
| `hidden_size=1024` | 每个 token 的 **向量维度**（宽度） | 每个字用1024维向量表示含义 |

```
输入序列（最多512个token）：
[CLS] 用户：我要去... [SEP] 帮我订机票 [SEP] PAD PAD...
  ↓       ↓                    ↓
每个token经过BERT后变成1024维向量：
token1 → [0.1, 0.3, ..., 0.8]  ← 1024个数字
token2 → [0.2, 0.5, ..., 0.1]  ← 1024个数字
```

**max_length 和模型上限的关系：**

roberta-wwm-ext-large 位置编码最大支持 **512**，设超过512会报错。设小于512（如128、256）是允许的，只是会截断更长的输入。

**hidden_size 必须和模型严格匹配：**

| 模型 | hidden_size |
|------|------------|
| bert-base-chinese | 768 |
| roberta-wwm-ext（base）| 768 |
| roberta-wwm-ext-large | **1024** |

如果用 large 模型但配置写了768，分类层输入维度对不上会直接报错。

---

### 2.3 batch_size 与多线程数据加载

**问题：batch_size 为什么越大越稳定但越吃显存？num_workers 是怎么加速数据读取的？**

**batch_size 越大越稳定的原因：**

每次更新梯度是基于这批样本的**平均梯度**：

```
batch_size=4：                     batch_size=64：
[样本1] → 梯度方向↗                [样本1~64] → 梯度方向→（更接近真实方向）
[样本2] → 梯度方向↘  平均后偏差大   噪声被平均掉，方向更稳定
[样本3] → 梯度方向→
[样本4] → 梯度方向↙
```

batch 越大，梯度方向越接近真实数据分布，训练曲线越平滑稳定。

**batch_size 越大越吃显存的原因：**

每个样本训练时需要在 GPU 上存储：
- 输入 tensor：`batch_size × 512 × 1024`
- 每层的激活值（反向传播时要用）
- 梯度

显存占用与 batch_size **成正比**，64个样本同时计算占的显存是4个样本的16倍。

**num_workers 多线程加载原理：**

不是直接读到显存，而是读到 **CPU 内存（RAM）**，形成流水线：

```
硬盘             CPU内存（RAM）                GPU显存
train.json  →  4个worker进程并行预读  →  GPU训练当前batch
               [worker1: 准备batch2]     ← GPU训练batch1时
               [worker2: 准备batch3]        worker已把batch2备好
               [worker3: 准备batch4]        训练完立刻取，无需等待
               [worker4: 准备batch5]
```

`num_workers=0` 时 CPU 和 GPU 串行工作，GPU 训练完才去读下一个 batch，GPU 会有空转等待。`num_workers=4` 让 GPU 和 CPU 并行，消除等待时间。

---

### 2.4 shuffle 打乱数据的原理

**问题：shuffle=true 是怎么打乱数据的，为什么要打乱？**

训练数据在磁盘上是固定顺序的，如果不打乱会有两个问题：

**问题1：类别偏置**

数据按类别排列时，前几个 batch 全是同一类，模型会先过度拟合该类，后面看到其他类时已经形成偏见，收敛方向偏差大。

**问题2：batch 内类别不均衡**

```
不 shuffle：
batch1: [route, route, route, route...]  ← 全是一个类，梯度极不稳定
batch2: [hotel, hotel, hotel, hotel...]

shuffle 后：
batch1: [route, hotel, transportation, combined...]  ← 各类混合
batch2: [other, route, hotel, transportation...]     ← 梯度更平稳
```

**实现原理：**

每个 epoch 开始前，`DataLoader` 生成一个随机排列的索引序列，按新顺序取样本：

```
原始索引：[0, 1, 2, 3, 4, 5, ...]
shuffle后：[3, 7, 1, 9, 0, 5, ...]  ← 每个 epoch 重新随机
```

每轮训练数据顺序都不同，模型每次都在"见新东西"。

**注意：** 验证集和测试集不需要 shuffle，只是评估，顺序不影响结果，代码中也是 `shuffle=False`。

---

### 2.5 学习率的定义

**问题：学习率是什么？**

学习率控制每次梯度更新时参数移动的步长：

```
新参数 = 旧参数 - 学习率 × 梯度
```

- 学习率太大：步子迈太大，来回震荡，错过最优点
- 学习率太小：步子太小，收敛极慢

类比下山找最低点：学习率是每步走多远，太大会跨过谷底，太小要走很久。

---

### 2.6 为什么学习率要分两个

**问题：SFT 不是直接调整原模型参数吗，为什么要分 backbone_learning_rate 和 learning_rate 两个？**

**首先理解分类头是什么**

BERT 主干本身只是一个特征提取器，输出的是 `[CLS]` 向量（1024维的数字），只是语义向量，没有任何分类含义：

```
BERT输出：[0.2, -0.5, 0.8, ..., 0.1]  ← 1024个数，只是语义，没有意义
```

我们在 BERT 后面**新加了一层 Linear(1024→5)**，把1024维语义向量压缩成5个分类分数，这就是分类头：

```
[CLS]向量（1024维）
    ↓
权重矩阵（1024×5）× 输入 + 偏置（5）= 5125个参数
    ↓
[route分数, hotel分数, transportation分数, combined分数, other分数]
```

**两部分的初始状态完全不同**

```
BERT主干（330M参数）：
  已在100GB中文语料上预训练
  "北京"、"机票"、"订酒店" 这些词的关系已经理解很好
  权重是有意义的，包含丰富语言知识

分类头（5125个参数）：
  随机初始化
  权重全是随机数
  完全不知道怎么分类，需要从零学起
```

**如果两个用同样的学习率会怎样**

统一用大学习率 1e-4：
```
分类头：合适，5125个参数，更新幅度正常，收敛正常
BERT主干：更新幅度太大，330M个参数全部大幅改变
         → 把预训练学到的语言知识全覆盖掉
         → 灾难性遗忘，效果反而变差
```

统一用小学习率 2e-5：
```
BERT主干：合适，温和微调，保留语言知识
分类头：更新幅度太小，5125个参数学得极慢
       → 训练很多轮还没学会分类，收敛慢
```

**所以分开设置，各取所需**

```
BERT主干：backbone_learning_rate = 2e-5  ← 小步微调，保留语言知识
分类头：  learning_rate          = 1e-4  ← 大步快学，从零学会分类
```

两个都在真实更新参数，只是更新速度不同，这就是**差分学习率**的核心思想。

**类比**

一个有20年经验的厨师来学做一道新菜：
- 基本刀工、火候（BERT的语言知识）：不需要推翻重来，微调就好 → 小学习率
- 新菜的调料配方（分类头）：是全新的东西，需要快速学 → 大学习率

如果让厨师把20年经验全推翻重学基本功，反而做不好这道新菜。

---

### 2.7 loss、交叉熵与 weight_decay 权重衰减

**问题：loss 是什么？交叉熵是什么？weight_decay 为什么能防止过拟合？**

**loss 是什么**

loss（损失）是衡量模型预测结果和真实标签之间**差距**的数值，越小说明模型预测越准：

```
真实标签：route
模型预测：[route:0.8, hotel:0.1, transportation:0.05, combined:0.03, other:0.02]
→ 预测对了，差距小，loss 小

真实标签：route
模型预测：[route:0.1, hotel:0.7, transportation:0.1, combined:0.05, other:0.05]
→ 预测错了，差距大，loss 大
```

训练的目标就是不断调整参数让 loss 越来越小。

**交叉熵是什么**

交叉熵是分类任务最常用的 loss 计算方式：

```
交叉熵 loss = -log(正确类别的预测概率)
```

举例：
```
真实标签：route
预测概率：route=0.8  → loss = -log(0.8) = 0.22  ← 预测准，loss 小
预测概率：route=0.1  → loss = -log(0.1) = 2.30  ← 预测错，loss 大
```

概率越高 loss 越小，概率越低 loss 越大。不是写死的，这是针对分类任务的标准选择，回归任务会用均方误差（MSE）等其他 loss。

**weight_decay 的名称**

正式名称是**权重衰减**，也叫 L2 正则化，两个名字说的是同一件事。

**完整 loss 公式**

```
总 loss = 交叉熵 loss + 0.01 × 所有参数的平方和
        = 交叉熵 loss + 0.01 × (w1² + w2² + w3² + ... + wN²)
```

**为什么能防止过拟合**

过拟合时模型某些参数会变得非常大，因为它在死记特定样本的特征：

```
没有 weight_decay：
[0.001, 0.002, 8.5, 0.003, 0.001]  ← 某个参数特别大，模型过度依赖它死记某个特征

有 weight_decay：
[0.3, 0.5, 0.4, 0.2, 0.3]  ← 参数都比较均匀，没有特别突出的大值
```

参数越大，惩罚项越大，总 loss 越大。模型为了降低 loss，就不敢让参数变太大，只保留真正有用的规律，不去死记个别样本的特征，泛化能力更强。

**0.01 系数的作用**

控制惩罚力度：
- 太大（如 0.1）：参数被压得太小，模型欠拟合，什么都学不好
- 太小（如 0.0001）：惩罚太弱，几乎没有效果
- 0.01：经验上的平衡值，惩罚适度

**类比：** 考试时老师规定答案写得越长扣分越多，逼着你只写核心要点，不能把所有细节都堆上去。

---

### 2.8 warmup_steps 预热步数

**问题：warmup_steps 是什么，为什么训练开始时需要预热？**

训练开始时学习率不是直接从目标值开始，而是从0慢慢升上去：

```
没有 warmup：
step1：学习率直接 2e-5，参数更新幅度大
       → 模型刚开始什么都不懂，大幅更新容易走错方向

有 warmup：
step1：    学习率 0
step2：    学习率 0.0000001
step3：    学习率 0.0000002
...
step2500： 学习率达到 2e-5  ← 预热结束，正式开始训练
```

`warmup_steps=0` 表示不手动指定，由 `warmup_ratio=0.1` 自动计算：

```
warmup步数 = 总步数 × 0.1
```

前 10% 的步数用于预热，之后学习率按余弦曲线下降。

**类比：** 发动机冷启动需要先热车，直接猛踩油门容易熄火。

---

### 2.9 gradient_accumulation_steps 梯度累积

**问题：gradient_accumulation_steps 和 batch_size 是什么关系，累积梯度是直接累加吗？**

**batch_size 和 gradient_accumulation 是两个层面的累积：**

`batch_size=64` 是 GPU 内部并行：
```
64条样本同时进入模型 → GPU内部自动平均梯度 → 更新一次参数
```

`gradient_accumulation_steps=8` 是代码层面手动攒梯度：
```
第1个batch → 计算梯度，不更新，先存着
第2个batch → 计算梯度，不更新，累加进去
...
第8个batch → 计算梯度，累加 → 更新一次参数
```

**两者叠加：**

```
batch_size=8,  gradient_accumulation=8：
  每次8条 → GPU内平均 → 得到1个梯度，攒8次 → 更新一次
  等效 batch_size = 8 × 8 = 64

batch_size=64, gradient_accumulation=1：
  每次64条 → GPU内平均 → 直接更新
  等效 batch_size = 64
```

两种方式等效 batch_size 相同，但前者显存占用小很多。

**累积梯度是直接累加，但 loss 要先除以累积步数：**

```python
loss = loss / gradient_accumulation_steps  # 防止梯度被放大8倍
loss.backward()  # 每次 backward 自动把梯度累加到现有梯度上
```

不除的话累加8次相当于梯度放大了8倍，和 batch_size=64 的效果就不等效了。

---

### 2.10 max_grad_norm 梯度裁剪

**问题：max_grad_norm=1.0 是什么，作用在哪里？**

**梯度爆炸是什么：**

训练中偶尔会出现某个 batch 的梯度异常大（比如一条脏数据），导致参数被更新得乱七八糟，模型直接崩掉：

```
正常梯度：[0.1, 0.2, 0.3, ...]  → 参数正常更新
爆炸梯度：[150, 300, 500, ...]  → 参数一次更新跑飞，模型崩溃
```

**梯度裁剪的原理：**

设一个上限，超过就等比例压缩，方向不变只压幅度：

```
梯度范数 = sqrt(150² + 300² + 500²) = 607
超过 max_grad_norm=1.0

所有梯度 × (1.0 / 607)
→ [150/607, 300/607, 500/607]
→ [0.25,   0.49,   0.82  ]  ← 方向不变，幅度压回正常范围
```

**作用位置：** 在 `trainer.py` 中，`optimizer.step()` 之前执行：

```python
# 先裁剪梯度
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
# 再用裁剪后的梯度更新参数
optimizer.step()
```

顺序很关键，必须先裁剪再更新，否则爆炸的梯度已经把参数更新坏了。

---

### 2.11 FGM 对抗训练：原理、扰动大小与目标层

**问题：FGM 是什么，原理是什么？fgm_epsilon 扰动大小如何理解？fgm_emb_name 指定的是什么？**

**FGM 是什么**

FGM（Fast Gradient Method）对抗训练，目的是让模型更鲁棒。

核心思想：训练时故意给输入加一点干扰，让模型在"被干扰的输入"上也能预测正确，这样模型就不会因为输入的微小变化而判断出错。

**原理：在 word embedding 上加扰动**

正常训练：
```
输入文本 → tokenize → embedding向量 → BERT → 分类
```

FGM 训练，每个 batch 额外多做一步：
```
Step1：正常前向+反向，得到 embedding 的梯度方向
Step2：沿梯度方向给 embedding 加扰动（让 loss 变大的方向）
       → 制造一个"对模型最难的输入"
Step3：用被扰动的 embedding 再做一次前向+反向
Step4：把两次梯度加在一起更新参数
Step5：还原 embedding，去掉扰动
```

模型同时学会应对"正常输入"和"被干扰的输入"，泛化能力更强。

**fgm_epsilon=1.0 扰动大小**

扰动的计算公式：
```
扰动 = epsilon × 梯度 / 梯度的范数
```

epsilon 控制扰动幅度：
- 太小（0.1）：扰动太弱，对抗效果不明显
- 太大（3.0）：扰动太强，embedding 被破坏，训练不稳定
- 0.5~1.0：经验上的常用范围，本项目设为 1.0

**fgm_emb_name 目标层**

```
fgm_emb_name: "bert.embeddings.word_embeddings"
```

指定对哪一层加扰动，这里是 word embedding 层，即文字转向量的那一层。只对这一层加扰动，不影响其他层。

---

## 3. train_epoch 训练循环流程

### 3.1 train_epoch 完整流程拆分

**问题：train_epoch 的完整训练流程是什么，如何拆分学习？**

`train_epoch` 是训练的核心循环，按以下顺序拆分：

**第一步：数据准备**
- batch 从 DataLoader 取出来
- 移到 GPU 设备上

**第二步：前向传播**
- 数据喂给模型
- 模型输出 loss 和 logits
- loss 除以梯度累积步数

**第三步：反向传播**
- `loss.backward()` 计算梯度

**第四步：FGM 对抗步骤**（可选）
- 给 embedding 加扰动
- 再做一次前向+反向
- 还原 embedding

**第五步：梯度更新**
- 梯度裁剪
- `optimizer.step()` 更新参数
- `scheduler.step()` 更新学习率
- `optimizer.zero_grad()` 清空梯度

**第六步：指标记录**
- 计算 batch 准确率
- 记录 loss、lr 到 TensorBoard

**第七步：验证与保存**
- 每 eval_steps 做一次验证
- 早停判断
- 保存最优模型和检查点

---

### 3.2 第七步详解：验证流程与早停机制

**问题：validate() 内部做了什么？指标怎么计算？早停是怎么判断的？**

**触发条件**

```python
if self.global_step % self.config.training.eval_steps == 0:
    val_metrics = self.validate()
```

每隔 `eval_steps`（配置=100）个 optimizer step，触发一次完整验证。

---

**validate() 内部流程**

```python
self.model.eval()        # 切换推理模式，关闭 dropout
with torch.no_grad():    # 不计算梯度，省显存
    for batch in val_dataloader:
        probabilities = torch.softmax(logits, dim=-1)   # logit → 概率分布
        predictions = torch.argmax(logits, dim=-1)       # 取最大概率的类别
```

遍历完整验证集，收集所有预测后，用 sklearn 统一计算指标：

```python
accuracy  = accuracy_score(y_true, y_pred)
precision, recall, f1, _ = precision_recall_fscore_support(
    y_true, y_pred, average='weighted'   # 按类别样本数加权平均
)
```

**四个指标的含义：**

| 指标 | 含义 |
|------|------|
| accuracy | 所有样本中预测正确的比例 |
| precision | 预测为某类的样本中，真正属于该类的比例（预测准不准） |
| recall | 某类真实样本中，被正确预测出来的比例（有没有漏掉） |
| **f1** | precision 和 recall 的调和平均，**早停用这个** |

`average='weighted'`：5个类别各自算 f1，再按样本数量加权求平均，避免类别不平衡时被稀有类拖垮。

---

**早停判断逻辑**

```python
metric_value = val_metrics.get('f1')
if self.early_stopping(metric_value, self.model):
    logger.info("触发早停，停止训练")
    return
```

`EarlyStopping` 内部逻辑：

```
本次 f1 比历史最佳高出 min_delta(0.001)?
    → 是：重置 patience 计数器，更新最佳值
    → 否：计数器 +1
        计数器 >= patience(10)?
            → 是：返回 True，触发停止
            → 否：继续训练
```

连续 10 次验证 f1 都没有提升超过 0.001，就停止训练。

---

**保存最佳模型**

```python
if self.monitor.update_best_metrics(val_metrics, self.global_step):
    self.save_model(".../best_model")
```

每次验证后，如果 f1 是历史最高，就把当前模型权重保存到 `best_model/`。最终推理时用的是这个目录，而不是最后一个 checkpoint。

---

**整体流程总结**

```
每 eval_steps 步
  → 跑完整验证集（no_grad，model.eval）
  → 算 accuracy / precision / recall / f1
  → 早停计数器判断是否停止
  → f1 历史最高 → 保存 best_model
```

---

### 3.3 验证指标详解：accuracy、precision、recall、f1

**问题：accuracy 和 precision 有什么区别？这四个指标分别是什么含义？**

用这个项目的 5 分类，假设验证集有 **10 条样本**：

```
真实标签：[route, route, hotel, hotel, hotel, transport, transport, combined, other, other]
模型预测：[route, hotel, hotel, hotel, other, transport, transport, combined, other, route]
```

**混淆矩阵（5×5）**

行 = 真实类别，列 = 预测类别：

```
              预测route  预测hotel  预测transport  预测combined  预测other
真实route         1          1            0              0           0
真实hotel         0          2            0              0           1
真实transport     0          0            2              0           0
真实combined      0          0            0              1           0
真实other         1          0            0              0           1
```

对角线 = 预测正确的样本。

---

**Accuracy 准确率**

```
accuracy = 对角线之和 / 总样本数
         = (1 + 2 + 2 + 1 + 1) / 10
         = 7 / 10 = 70%
```

看的是**整体**，所有类别一起算预测对了多少。

---

**Precision 精确率（以 route 为例）**

看混淆矩阵中 **route 那一列**（所有被预测为 route 的）：

```
precision(route) = TP / 该列之和
                 = 1 / (1 + 0 + 0 + 0 + 1)
                 = 1/2 = 50%
```

模型预测了 2 次 route，只有 1 次真的是 route，另 1 次其实是 other → 误报了。

**你说"这是 route"，有多大把握是对的。** 低 precision = 误报多。

---

**Recall 召回率（以 route 为例）**

看混淆矩阵中 **route 那一行**（所有真实为 route 的）：

```
recall(route) = TP / 该行之和
              = 1 / (1 + 1 + 0 + 0 + 0)
              = 1/2 = 50%
```

真实有 2 条 route，只找出来 1 条，另 1 条被判成了 hotel → 漏掉了。

**真正的 route，你找出来了多少。** 低 recall = 漏报多。

---

**F1 分数（以 route 为例）**

```
f1(route) = 2 × precision × recall / (precision + recall)
           = 2 × 0.5 × 0.5 / (0.5 + 0.5)
           = 50%
```

precision 和 recall 的调和平均，两者都高 f1 才高，一个低就会被拉下来。

---

**weighted 加权平均（实际代码中用的）**

每个类别都算出自己的 precision / recall / f1，再按**该类真实样本数**加权平均：

```
route:     2条  权重=2/10
hotel:     3条  权重=3/10
transport: 2条  权重=2/10
combined:  1条  权重=1/10
other:     2条  权重=2/10

weighted f1 = f1(route)×0.2 + f1(hotel)×0.3 + f1(transport)×0.2 + f1(combined)×0.1 + f1(other)×0.2
```

样本多的类别对最终 f1 影响更大，避免稀有类别（如 combined 只有1条）把整体结果拖偏。

---

**四个指标对比总结**

| 指标 | 看哪里 | 问的问题 | 低说明什么 |
|------|--------|---------|-----------|
| Accuracy | 对角线之和/总数 | 整体预测对了多少 | 总体效果差（类别不平衡时可能虚高） |
| Precision | 某列 TP/列之和 | 预测为A的，真的是A吗 | 误报多 |
| Recall | 某行 TP/行之和 | 真实的A，找出来了吗 | 漏报多 |
| F1 | 2×P×R/(P+R) | precision 和 recall 综合表现 | 两者至少有一个差 |

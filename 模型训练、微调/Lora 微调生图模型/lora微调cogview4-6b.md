# Lora 微调 CogView4-6B

## 一、扩散生图模型工作原理

### 子问题 1：扩散生图模型的完整工作原理是什么？

**Q：总结扩散生图模型（以 CogView4 为例）的工作原理？**

**A：**

#### 训练阶段：学会"破坏"的规律

目标：给模型看一张"被破坏到一半"的图，让它学会预测破坏的方向。

```
Step 1  原图 → VAE编码 → latent x_0  [B, 16, 128, 128]
        （像素空间压缩60倍，后续所有操作都在latent空间）

Step 2  文字 → GLM-4编码 → prompt_embedding  [B, 224, 4096]
        （文字条件，控制生成内容）

Step 3  随机采样噪音 ε ~ N(0,1)，随机采样时间步 t ∈ [0, T]
        按线性插值加噪：
        x_t = (1-t) * x_0  +  t * ε
        t≈0 → 几乎是原图，t≈1 → 几乎是纯噪音

Step 4  把 x_t 切成 4096 个 2×2 的 patch，每个 patch → visual token
        文字 token（224个）和图像 token（4096个）拼在一起
        送入 30 层 DiT Transformer（双向全注意力，非因果）
        时间步 t 通过 AdaLayerNorm 注入每一层

Step 5  Transformer 输出预测值，与 model_label 算 MSE loss
        model_label = ε - x_0  （从原图指向纯噪音的速度向量）
        loss = mean((预测值 - (ε - x_0))²)
```

#### 推理阶段：沿反方向走回原图

```
纯随机噪音 z_T
    ↓
    用文字引导，Transformer 预测此刻的速度向量 v
    ↓
    往反方向走一小步：z_{t-1} = z_t - v * Δt
    ↓
    重复 ~20 步（Flow Matching 路径是直线，比 DDPM 快 50 倍）
    ↓
    z_0（干净的 latent）
    ↓
    VAE 解码 → 最终图像
```

#### 三个关键组件的分工

| 组件 | 职责 | 微调时是否更新 |
|---|---|---|
| VAE | 压缩/还原图像，做编解码 | 固定不动 |
| GLM-4 | 把文字变成向量 | 固定不动 |
| DiT Transformer | 接收加噪latent+文字+时间步，输出速度向量 | **唯一被训练的部分** |

#### Flow Matching vs DDPM

| | DDPM | Flow Matching (CogView4) |
|---|---|---|
| 路径形状 | 弯曲（随机游走） | 直线（线性插值） |
| 推理步数 | 1000步 | ~20步 |
| 训练目标 | 预测噪音 ε | 预测速度向量 ε - x_0 |
| 速度向量含义 | — | 从原图指向纯噪音的方向箭头 |

> **一句话总结：** 训练时把干净图像和随机噪音做线性插值得到"半噪图"，让 Transformer 学会预测插值方向；推理时从纯噪音出发，每步按预测方向反向走 20 步，配合文字条件引导，最终 VAE 解码出图像。

---

## 二、理解 LoRA 是什么

### 子问题 1：LoRA 本身是什么？

**Q：LoRA 的原理、核心超参数、注入位置和实际效果是什么？**

**A：**

#### 核心问题：全量微调为什么贵

CogView4-6B 的 DiT Transformer 有 60亿参数，全量微调（SFT）意味着：
- 所有参数都要存梯度 + 优化器状态（Adam）
- 显存需求 80GB+，需要 8卡 A100 才够

#### LoRA 的核心思想：低秩分解

LoRA 的假设：**微调时权重的变化量是低秩的**，即有意义的变化只发生在少数几个方向上。

不直接修改原始权重矩阵 W，而是在旁边插入两个小矩阵 A、B：

```
原始权重:  W           形状 [2560, 2560]，参数量 6,553,600
LoRA新增:  B × A       B: [2560, r]，A: [r, 2560]，r=128
                       参数量 = 2560×128 + 128×2560 = 655,360

实际计算:  W' = W + (alpha/r) × B × A
```

训练时 W 完全冻结，只有 A 和 B 在更新。

#### r 和 alpha：两个核心超参数

**r（秩）** 同时决定两件事：

```
第一：A、B 矩阵的大小
  r 越小 → A/B 越小 → 参数越少
  r 越大 → A/B 越大 → 参数越多

第二：能表达的"变化方向"数量
  B×A 是一个秩为 r 的矩阵
  r=1   → 所有变化只沿 1 个方向，极度受限
  r=128 → 能表达 128 个独立方向的变化（本项目）
  r=2560→ 退化成全量微调
```

**alpha（缩放系数）** 控制叠加幅度：

```
实际叠加量 = (alpha / r) × B×A

本项目 r=128, alpha=64：
  缩放系数 = 64/128 = 0.5
  → LoRA 学到的变化以半强度叠加，防止破坏原始能力
```

两个参数解耦，可以独立调节：

| 参数 | 管什么 |
|---|---|
| r | 变化的**复杂度**（能学多少方向） |
| alpha/r | 变化的**幅度**（学到的东西叠加多重） |

#### 注入位置：只加在注意力的 Q K V O

项目配置在 `src/cogkit/utils/lora.py:33`：

```python
LoraConfig(
    r=128,
    lora_alpha=64,
    target_modules=["to_q", "to_k", "to_v", "to_out.0"],
)
```

DiT 有 30 层，每层 4 个矩阵：

```
注入总数 = 30层 × 4个矩阵 = 120 对 (A, B)
VAE、GLM-4 文本编码器完全不碰
```

#### 初始化：B 全零保证训练稳定

```
A：随机初始化（高斯分布）
B：全部初始化为 0

→ 训练开始时 B×A = 0，模型行为和原始完全一样
→ 从零开始学习增量，不会一开始就破坏原模型
```

#### 保存产物极小

```
全量微调保存：~24GB（完整模型权重）
LoRA 保存：  ~300MB（只有 120 对 A/B 矩阵）
文件名：adapter_model.safetensors
```

推理时把 LoRA 叠加回原模型：`W' = W + 0.5 × B×A`，原模型文件本身不变。

> **一句话总结：** LoRA 冻结原始权重，在注意力的 Q/K/V/O 旁边各插入两个小矩阵 A（[r, d]）和 B（[d, r]），r 决定能学多复杂的变化，alpha/r 控制叠加幅度，训练时只更新 A/B。参数量从 60 亿降到百万级，显存从 80GB+ 降到 20GB，产物只有 300MB。

### 子问题 2：代码用的 PEFT 库是什么？

**Q：PEFT 是什么，在项目里起什么作用？**

**A：**

PEFT（Parameter-Efficient Fine-Tuning）是 HuggingFace 的库，**把 LoRA 的工程实现全包了**。

LoRA 是算法思想，PEFT 是实现它的工具：

```
LoRA  → 设计图：冻结W，插入低秩A/B，只训练A/B
PEFT  → 施工队：自动找层、注入矩阵、冻结权重、保存加载
```

项目里实际只用了 4 个函数（`src/cogkit/utils/lora.py`）：

```python
LoraConfig                 # 定义配置：r、alpha、注入哪些层
inject_adapter_in_model    # 遍历模型，自动注入 120 对 A/B
get_peft_model_state_dict  # 保存时只提取 A/B 权重
set_peft_model_state_dict  # 加载时把 A/B 恢复到对应位置
```

没有 PEFT 的话，需要手动遍历 30层 × 4个矩阵，逐一替换并冻结，极易出错。

---

## 三、训练流程

### 子问题 1：数据格式是什么？

**Q：t2i（文生图）任务的数据集格式要求是什么？**

**A：**

#### train/metadata.jsonl

每行一条 JSON，`file_name` 对应图片文件名，`prompt` 是图片的文字描述：

```json
{"file_name": "image_001.png", "prompt": "A curious mouse watches a distressed cow..."}
{"file_name": "image_002.png", "prompt": "A dynamic sequence unfolds on the deck of a ship..."}
```

#### test/prompt.jsonl

验证集只有 prompt，没有图片。训练过程中按这些 prompt 生成图片，用于观察模型效果：

```json
{"prompt": "A black and white animated scene unfolds with an anthropomorphic goat..."}
{"prompt": "A vibrant cherry red sports car sits proudly under the gleaming sun..."}
```

#### 为什么没有测试集

项目把验证集目录命名为 `test/`，但它扮演的是**验证集**的角色：训练过程中每隔 `validation_steps` 步，用其中的 prompt 生图，监控当前训练效果，属于边训边看。

没有设计真正测试集，是因为 LoRA 微调生图的效果评估通常是**定性的**（人眼看图质量），不像分类任务有客观指标，训练完直接人工生图评估即可。

| | 用途 | 什么时候用 |
|---|---|---|
| 训练集 `train/` | 更新模型权重 | 每个 step |
| 验证集 `test/` | 监控训练质量，观察有没有过拟合 | 每隔 `validation_steps` 步 |
| 测试集（无） | 训练完成后的最终客观评估 | 本项目不需要 |

#### 注意事项

- 图片格式：`.png`
- 训练过程会在 `train/` 目录下自动生成 `.cache/` 目录，预计算并缓存 latent 和 prompt embedding，加速后续 epoch
- 如果更换了数据集，**必须手动删除 `.cache/` 目录**再重新训练，否则用的是旧数据的缓存

### 子问题 2：batch_size、gradient_accumulation_steps、learning_rate 三者关系？

**Q：这三个训练参数各自的含义以及联动关系是什么？**

**A：**

#### batch_size

每次更新权重，模型需要看一批数据来计算梯度：

```
batch_size = 1  → 每步看 1 张图，算一次梯度，更新一次权重
batch_size = 4  → 每步看 4 张图，梯度取平均，更新一次权重
```

batch_size 越大，梯度方向越稳定（多张图的平均），训练更平滑。但 CogView4 单卡 20GB 显存，`batch_size=1` 已经快满，根本没法设更大的值，否则 OOM。

#### gradient_accumulation_steps：穷人的大 batch

```yaml
batch_size: 1
gradient_accumulation_steps: 4
```

实际执行过程：

```
Step 1: 看第1张图，算梯度，先不更新权重，梯度暂存
Step 2: 看第2张图，算梯度，累加到暂存的梯度上
Step 3: 看第3张图，算梯度，继续累加
Step 4: 看第4张图，算梯度，累加完毕 → 取平均 → 更新一次权重
```

效果和 `batch_size=4` 完全等价，但每步显存只占 1 张图的量：

```
等效 batch size = batch_size × gradient_accumulation_steps
```

#### 和 learning_rate 的联动

等效 batch size 变大时，梯度方向更稳定，每步可以走更大的步子，learning_rate 也应相应调大：

```
经验规则（Linear Scaling Rule）：batch size 翻倍 → learning_rate 也翻倍

batch_size=1, grad_accum=1  → lr=1e-5
batch_size=1, grad_accum=4  → lr=4e-5（等效batch=4，lr×4）
```

本项目默认 `batch_size=1, grad_accum=1`，等效 batch size=1，`lr=2e-5` 是对应的保守值。

> **一句话总结：** `gradient_accumulation_steps` 是用时间换空间，多走几步才更新一次权重，显存不变但等效于更大的 batch size。等效 batch size 变大时 learning_rate 也要跟着调大。

### 子问题 3：mixed_precision 的作用是什么？

**Q：mixed_precision: "bf16" 是什么，为什么用 bf16 而不是 fp16？**

**A：**

#### 正常训练用 fp32

默认情况下所有参数和计算都用 fp32（32位浮点数），每个数占 4 字节。

#### 混合精度：大部分用 bf16，关键处保留 fp32

```
fp32: 1位符号 + 8位指数 + 23位小数   4字节
bf16: 1位符号 + 8位指数 +  7位小数   2字节
fp16: 1位符号 + 5位指数 + 10位小数   2字节
```

**为什么用 bf16 不用 fp16：** bf16 和 fp32 的指数位数一样（都是8位），数值范围相同，梯度不容易上溢/下溢，训练稳定。fp16 只有5位指数，数值范围小，大模型训练容易数值崩掉。

#### 混合体现在哪里

不是所有地方都用 bf16，"混合"指的是：

```
前向传播、反向传播   → bf16（省显存、计算快）
权重主副本           → fp32（保留精度，防止累积误差）
梯度累积             → fp32（精确更新）
```

框架自动处理这个切换，不需要手动干预。

#### 实际收益

```
显存：模型激活值从 4字节→2字节，省约 30~40%
速度：A100 对 bf16 有专用 Tensor Core 加速，训练更快
```

对 CogView4-6B 来说，bf16 是标配，几乎没有理由改成 fp32。

### 子问题 4：A/B 矩阵插入哪些层，为什么这样设置？

**Q：LoRA 的 A/B 矩阵插在哪里，为什么只插注意力层，Q/K/V/O 四个全插吗？**

**A：**

#### 插入位置

本项目配置 `target_modules=["to_q", "to_k", "to_v", "to_out.0"]`，只插注意力层，FFN 不插：

```
每个 TransformerBlock（共30层）:
  ├── Attention
  │     ├── to_q     ← 插入 ✓
  │     ├── to_k     ← 插入 ✓
  │     ├── to_v     ← 插入 ✓
  │     └── to_out.0 ← 插入 ✓
  └── FFN            ← 不插 ✗

总计：30层 × 4个矩阵 = 120 对独立的 A/B
```

#### 每个层有自己独立的 A/B，不是共享

`r=128, alpha=64` 只规定了**结构**（A/B 的形状），每对 A/B 的参数值是独立学习的：

```
to_q 第1层  → 自己的 A_q1, B_q1
to_k 第1层  → 自己的 A_k1, B_k1
to_v 第1层  → 自己的 A_v1, B_v1
to_out 第1层 → 自己的 A_o1, B_o1
...共 120 对，各自学各自的更新方向
```

Q/K/V/O 职责不同，需要的更新方向也不同，独立的 A/B 才能让每个投影按自己的需要学习。

#### 为什么只插注意力层不插 FFN

注意力层决定图像 patch 之间、图像和文字之间怎么交互，是风格学习的核心。FFN 负责每个 token 独立的特征变换，对风格迁移影响相对小。只插注意力层在保证效果的前提下参数量更少。

#### 为什么 Q/K/V/O 四个全插

| 配置 | 参数量 | 适合场景 |
|---|---|---|
| 只插 Q/V | 最少 | LLM 轻量适配（原始论文默认） |
| 插 Q/K/V/O | 中等 | 生图风格微调（本项目） |
| 插注意力 + FFN | 最多 | 复杂内容学习，过拟合风险高 |

原始 LoRA 论文在 LLM 上发现只插 Q/V 就够用，但生图模型更依赖注意力捕捉空间关系，插全四个表达能力更强，是生图社区的主流做法。没有绝对标准，实际效果需要实验验证。

#### A/B 是叠加不是替代

```
推理时：W' = W + (alpha/r) × B×A
原始 W 始终保留，A/B 只是增量
```

这也是为什么 LoRA 产物只有 300MB，不需要保存完整模型。

### 子问题 7：训练好的 LoRA 如何合并权重并用于推理？

**Q：merge.py 做了什么，推理时如何加载 LoRA？**

**A：**

#### merge.py 是格式转换，不是权重合并

训练时 checkpoint 用 PyTorch DCP 格式存储（分布式多卡的碎片文件），推理时用不了，需要先转换：

```bash
python merge.py \
  --checkpoint_dir /output/checkpoint-200/ \
  --output_dir /output/lora_weights/ \
  --lora
```

输出 `adapter_model.safetensors`，A/B 的值不变，原模型权重完全没动。

> 注意：`low_vram=true`（QLoRA）训练时直接保存的就是 `adapter_model.safetensors`，不需要跑 merge.py。

#### 推理时动态叠加

不需要提前把 A/B 写进原模型，加载时自动叠加：

```bash
cogkit inference "your prompt" "THUDM/CogView4-6B" \
  --lora_model_id_or_path /output/lora_weights/
```

推理时每层自动计算 `W' = W + (alpha/r) × B×A`，原模型文件永远不被修改。

#### 完整流程

```
训练  → checkpoint-{step}/（DCP 格式）
merge.py --lora  → adapter_model.safetensors（300MB）
推理  → 加载原模型 + 指定 LoRA 路径 → 动态叠加 → 生图
```

---

## 四、部署与服务

### 子问题 1：训练好的模型如何封装成服务对外提供接口，为什么不能用 vLLM 或 Ollama？

**Q：训练好的模型如何封装成服务对外提供接口，为什么不能用 vLLM 或 Ollama？**

**A：**

#### 为什么不能用 vLLM 或 Ollama

vLLM 和 Ollama 都是专为 **LLM 文字生成**设计的推理框架，核心优化针对自回归 token 生成。CogView4 是扩散模型，推理是 20 步去噪循环，完全不是自回归生成，两者都不支持。

#### 项目自带 FastAPI 服务，三个文件串成完整链路

**`.env` 配置模型路径：**
```
COGVIEW4_PATH="/path/to/CogView4-6B"
LORA_DIR="/path/to/adapter_model.safetensors"
```

**`settings.py` 自动读取 .env：** 通过 `pydantic_settings` 把 `.env` 变量映射到配置类，不需要手动 `os.getenv()`。

**`launch.py` 启动服务：** `cogkit launch` 本质是调用 `fastapi_cli` 启动 `application.py`，等价于 `fastapi run application.py`。

启动时的完整链路：
```
cogkit launch --host 0.0.0.0 --port 8000
  ↓ 读取 .env，初始化 ImageGenerationService
  ↓ load_pipeline() 把模型加载进 GPU 常驻
  ↓ FastAPI 注册 /v1 路由，等待请求
```

#### 对外暴露的关键点

`--host 0.0.0.0` 监听所有网卡，外部才能访问（`127.0.0.1` 只有本机能访问）。

还需要在云服务器**安全组**开放对应端口。生产环境建议加一层 Nginx 反向代理，统一走 80/443 端口，可附加 SSL、限流、鉴权。

```
业务方 → Nginx（80端口）→ localhost:8000（cogkit服务）
```为什么没有早停和 dropout？

**Q：训练过程中没有早停机制和 dropout，如何防止过拟合？**

**A：**

#### 没有早停

早停需要一个**客观的验证指标**来判断"模型变差了，停止训练"。

分类任务可以用 accuracy，但生图任务没有这样的客观指标：验证集只是生图让人看，好不好看是主观判断，代码无法自动决策"现在该停了"。

本项目的替代方案是：靠 `checkpointing_steps` 定期保存 checkpoint，人工对比不同 step 的生成图，手动挑最好的那个 checkpoint，**把早停的判断权交给人**。

#### 没有 dropout

LoRA 本身就是一种隐式正则化：

```
原始权重 W 完全冻结，不会过拟合
只有 A/B 矩阵在训练，120对 × 参数量极小
参数量少本身就限制了过拟合能力
```

相比全量微调（60亿参数全动），LoRA 过拟合风险已大幅降低，额外加 dropout 收益很小。PEFT 的 `LoraConfig` 支持 `lora_dropout` 参数，本项目没有配置，默认为 0。

#### 过拟合实际靠什么控制

```
train_epochs 设小      → 不让模型反复看同一批图太多次
learning_rate 用小值   → 2e-5 已经很保守，更新幅度小
数据量足够多           → 数据越多越难过拟合
人工看验证图           → 发现生成图开始和训练集一模一样时手动停
```

### 子问题 6：LoRA 微调生图模型的训练流程是什么，和 NLP 微调有什么区别？

**Q：fit() 的完整步骤是什么，和 NLP 微调相比核心区别在哪里？**

**A：**

#### 整体流程：fit() 的 7 个步骤

```
Step 1  check_setting()                  检查分布式环境、配置合法性
Step 2  prepare_models()                 加载四个组件：GLM-4、Transformer、VAE、Scheduler
                                         预先编码所有训练数据，缓存到 .cache/
Step 3  prepare_dataset()                构建训练集和验证集的 DataLoader
Step 4  prepare_trainable_parameters()   冻结所有参数，只给 Transformer 注入 A/B
Step 5  prepare_model()                  用 DDP 包裹 Transformer，支持多卡
Step 6  prepare_optimizer()              只把 A/B 参数交给 Adam 优化器
Step 7  train()                          正式训练循环
```

#### Step 4 和 NLP 的区别

NLP 微调通常只有一个模型，整体参与训练。

生图有四个组件，但只有 Transformer 的 A/B 在训练。VAE 和 GLM-4 在 Step 2 完成编码后就卸载出 GPU，训练时完全不在场，它们的权重全程冻结。

#### Step 7 训练循环：loss 计算方式完全不同

这是和 NLP 最本质的区别。

NLP 有明确的人工标注标签，模型预测结果和标签算交叉熵 loss。

生图没有人工标注，**标签是自己构造的**：把干净图像和随机噪音线性插值得到加噪图，让 Transformer 预测插值方向，再和真实速度向量算 MSE loss。整个训练过程不需要任何人工标注，只需要图片和对应的文字描述。

#### 总结对比

| | NLP 微调 | 生图 LoRA 微调 |
|---|---|---|
| 参与训练的组件 | 整个模型 | 只有 Transformer 的 A/B |
| Loss 标签来源 | 人工标注 | 自己构造（加噪再预测速度） |
| 验证方式 | 算 accuracy | 生图人工看 |
| 训练目标 | 预测正确类别/token | 预测噪音速度向量 |

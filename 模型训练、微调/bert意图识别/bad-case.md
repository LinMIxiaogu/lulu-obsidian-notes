典型：
2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [81] ==真实=transportation | 预测=combined (置信度=0.926) | 文本: 去北海 4天3晚 2人情侣 ➕机票人均2千==
2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [750] ==真实=combined | 预测=transportation (置信度=0.941) | 文本: 到我住的酒店乘坐地铁路线==
2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [965] ==真实=combined | 预测=hotel (置信度=0.922) | 文本: 交通的话住哪里方便==
2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [1561] ==真实=combined | 预测=route (置信度=0.938) | 文本: 4.1日到4.5日，有海滩，有风土人情==

2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [240] ==真实=route | 预测=other (置信度=0.910) | 文本: 重庆美食3日游==
2026-04-14 15:00:10 | WARNING  | src.trainer:log_bad_cases:609 -   [291] ==真实=other | 预测=route (置信度=0.945) | 文本: 去大连、烟台、威海，淡季是什么时间段？==

```
2026-04-14 14:57:14 | INFO     | src.utils:setup_logging:268 - 日志系统已设置，日志文件: ./logs/training_20260414_145714.log
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:193 - === 系统信息 ===
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:194 - Python版本: 0
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:195 - PyTorch版本: 2.11.0+cu130
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:196 - CUDA可用: True
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:199 - CUDA版本: 13.0
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:200 - cuDNN版本: 91900
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:201 - GPU数量: 1
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:205 - GPU 0: NVIDIA RTX PRO 6000 Blackwell Server Edition
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:206 -   内存: 95.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:207 -   计算能力: 12.0
2026-04-14 14:57:14 | INFO     | src.utils:log_system_info:209 - ================
2026-04-14 14:57:14 | INFO     | src.utils:set_seed:58 - 随机种子已设置为: 42
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:78 - 检测到 1 个GPU设备
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:95 - 使用GPU设备: cuda:0
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:96 - GPU名称: NVIDIA RTX PRO 6000 Blackwell Server Edition
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:97 - GPU内存: 95.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:98 - 计算能力: 12.0
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:102 - ✅ 支持BFloat16精度
2026-04-14 14:57:14 | INFO     | src.utils:get_optimal_device:108 - ✅ 支持Flash Attention 2
2026-04-14 14:57:14 | INFO     | src.utils:optimize_gpu_memory:188 - GPU内存已优化
2026-04-14 14:57:14 | INFO     | src.utils:log_memory_usage:423 - 🚀 启动时GPU内存使用:
2026-04-14 14:57:14 | INFO     | src.utils:log_memory_usage:424 -   总内存: 95.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:log_memory_usage:425 -   已用内存: 0.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:log_memory_usage:426 -   缓存内存: 0.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:log_memory_usage:427 -   可用内存: 95.0 GB
2026-04-14 14:57:14 | INFO     | src.utils:create_output_dirs:233 - 输出目录已创建: ./outputs_new
2026-04-14 14:57:14 | INFO     | src.utils:create_output_dirs:234 - 日志目录已创建: ./logs
2026-04-14 14:57:14 | INFO     | __main__:prepare_data:103 - 📊 准备数据...
2026-04-14 14:59:45 | INFO     | src.data_processor:load_json_data:176 - 从 ./data/train.json 加载了 15998 条数据
2026-04-14 14:59:45 | INFO     | src.data_processor:load_json_data:176 - 从 ./data/val.json 加载了 1998 条数据
2026-04-14 14:59:45 | INFO     | src.data_processor:load_json_data:176 - 从 ./data/test.json 加载了 2004 条数据
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:145 - ✅ 数据加载完成:
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:146 -   📈 训练样本: 15998
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:147 -   📊 验证样本: 1998
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:148 -   🧪 测试样本: 2004
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:149 -   🏷️ 标签数量: 5
2026-04-14 14:59:45 | INFO     | __main__:prepare_data:150 -   📝 标签名称: ['combined', 'hotel', 'other', 'route', 'transportation']
2026-04-14 14:59:45 | INFO     | __main__:_log_sample_inputs:160 - 🔍 抽样打印前 1 条训练样本（decode 后的真实模型输入）
2026-04-14 14:59:45 | INFO     | __main__:_log_sample_inputs:161 - ================================================================================
2026-04-14 14:59:45 | INFO     | __main__:_log_sample_inputs:176 - --- 训练样本 0 | label=route | 有效长度=4 | [SEP] 位置=[3] ---
2026-04-14 14:59:45 | INFO     | __main__:_log_sample_inputs:177 - [CLS] 新 疆 [SEP]
2026-04-14 14:59:45 | INFO     | __main__:_log_sample_inputs:178 - ================================================================================
2026-04-14 14:59:45 | INFO     | src.utils:log_memory_usage:423 - 📊 数据加载后GPU内存使用:
2026-04-14 14:59:45 | INFO     | src.utils:log_memory_usage:424 -   总内存: 95.0 GB
2026-04-14 14:59:45 | INFO     | src.utils:log_memory_usage:425 -   已用内存: 0.0 GB
2026-04-14 14:59:45 | INFO     | src.utils:log_memory_usage:426 -   缓存内存: 0.0 GB
2026-04-14 14:59:45 | INFO     | src.utils:log_memory_usage:427 -   可用内存: 95.0 GB
2026-04-14 14:59:45 | INFO     | __main__:prepare_model:190 - 🤖 准备ModernBERT模型...
2026-04-14 14:59:45 | INFO     | __main__:prepare_model:197 - 📥 使用在线模型: hfl/chinese-roberta-wwm-ext-large
2026-04-14 14:59:45 | INFO     | src.model:__init__:481 - ModelManager初始化，设备: cuda:0
2026-04-14 14:59:45 | INFO     | __main__:prepare_model:218 - 🎯 测试模式：加载训练权重 ./outputs_new/best_model
2026-04-14 14:59:45 | INFO     | src.model:from_pretrained:400 - ✅ 成功读取模型信息: outputs_new/best_model/model_info.json
2026-04-14 14:59:45 | INFO     | src.model:from_pretrained:409 - 加载参数: labels=5, dropout=0.1
2026-04-14 14:59:45 | INFO     | src.model:_parse_model_path:96 - 检测到训练检查点，基础模型: hfl/chinese-roberta-wwm-ext-large
2026-04-14 14:59:45 | INFO     | src.model:_prepare_model_kwargs:109 - GPU计算能力: (12, 0)
2026-04-14 14:59:45 | INFO     | src.model:_prepare_model_kwargs:124 - 模型精度: torch.bfloat16
2026-04-14 14:59:45 | INFO     | src.model:_load_base_model:138 - 加载基础模型: hfl/chinese-roberta-wwm-ext-large
2026-04-14 15:00:06 | INFO     | src.model:_load_base_model:146 - ✅ 基础模型加载成功
2026-04-14 15:00:06 | INFO     | src.model:_init_classifier_layers:171 - 分类层初始化完成: 1024 -> 5
2026-04-14 15:00:06 | INFO     | src.model:_apply_precision_settings:176 - 应用精度设置: torch.bfloat16
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:191 - ==================================================
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:192 - 模型初始化完成:
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:193 -   输入路径: outputs_new/best_model
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:194 -   基础模型: hfl/chinese-roberta-wwm-ext-large
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:195 -   是否检查点: True
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:196 -   标签数量: 5
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:197 -   隐藏层大小: 1024
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:198 -   Dropout: 0.1
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:199 -   总参数: 325,786,117
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:200 -   可训练参数: 325,786,117
2026-04-14 15:00:06 | INFO     | src.model:_log_initialization_info:201 - ==================================================
2026-04-14 15:00:06 | INFO     | src.model:from_pretrained:420 - ✅ 模型实例创建成功
2026-04-14 15:00:06 | INFO     | src.model:from_pretrained:430 - 加载训练权重: outputs_new/best_model/pytorch_model.bin
2026-04-14 15:00:07 | INFO     | src.model:from_pretrained:446 - ✅ 训练权重加载成功
2026-04-14 15:00:07 | INFO     | src.model:from_pretrained:451 - 应用精度设置: torch.bfloat16
2026-04-14 15:00:07 | INFO     | src.model:_apply_precision_settings:176 - 应用精度设置: torch.bfloat16
2026-04-14 15:00:07 | INFO     | src.model:from_pretrained:469 - 🎉 模型加载完成
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:228 - ✅ 训练权重加载完成
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:245 - ✅ 模型信息:
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:246 -   模型名称: outputs_new/best_model
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:247 -   标签数量: 5
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:248 -   总参数: 325,786,117
2026-04-14 15:00:07 | INFO     | __main__:prepare_model:249 -   可训练参数: 325,786,117
2026-04-14 15:00:07 | INFO     | src.utils:log_memory_usage:423 - 🤖 模型加载后GPU内存使用:
2026-04-14 15:00:07 | INFO     | src.utils:log_memory_usage:424 -   总内存: 95.0 GB
2026-04-14 15:00:07 | INFO     | src.utils:log_memory_usage:425 -   已用内存: 0.0 GB
2026-04-14 15:00:07 | INFO     | src.utils:log_memory_usage:426 -   缓存内存: 0.0 GB
2026-04-14 15:00:07 | INFO     | src.utils:log_memory_usage:427 -   可用内存: 95.0 GB
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:78 - 检测到 1 个GPU设备
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:95 - 使用GPU设备: cuda:0
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:96 - GPU名称: NVIDIA RTX PRO 6000 Blackwell Server Edition
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:97 - GPU内存: 95.0 GB
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:98 - 计算能力: 12.0
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:102 - ✅ 支持BFloat16精度
2026-04-14 15:00:07 | INFO     | src.utils:get_optimal_device:108 - ✅ 支持Flash Attention 2
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:153 - 使用差分学习率:
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:154 -   BERT主干学习率: 2e-05
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:155 -   分类头学习率: 0.0001
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:156 -   BERT参数数量: 391
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:157 -   分类头参数数量: 4
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:203 - 优化器: AdamW
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:204 - 学习率调度器: cosine
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:205 - 总训练步数: 25000
2026-04-14 15:00:07 | INFO     | src.trainer:_setup_optimizer_and_scheduler:206 - 预热步数: 2500
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:74 - 🔥 使用BFloat16精度训练，无需梯度缩放
2026-04-14 15:00:07 | INFO     | src.monitor:__init__:69 - TensorBoard日志目录: ./logs/tensorboard/bert_intent_20260414_150007
2026-04-14 15:00:07 | INFO     | src.monitor:__init__:99 - 训练监控器已初始化 (TensorBoard: True, WandB: False)
2026-04-14 15:00:07 | INFO     | src.adversarial:__init__:56 - 🛡️ FGM 目标参数: ['bert.embeddings.word_embeddings.weight']
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:116 - 🛡️ 启用 FGM 对抗训练: emb_name=bert.embeddings.word_embeddings, epsilon=1.0
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:121 - 训练器初始化完成
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:122 -   设备: cuda:0
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:123 -   训练样本数: 15998
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:124 -   验证样本数: 1998
2026-04-14 15:00:07 | INFO     | src.trainer:__init__:125 -   测试样本数: 2004
2026-04-14 15:00:07 | INFO     | __main__:main:331 - 🎯 开始置信度阈值评估...
2026-04-14 15:00:07 | INFO     | src.trainer:evaluate_with_confidence_threshold:635 - 🎯 开始置信度阈值评估...
2026-04-14 15:00:10 | INFO     | src.trainer:_log_bad_cases:575 - ❌ 错误样本数: 233 / 2004 (错误率 11.6%)
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [4] 真实=route | 预测=combined (置信度=0.793) | 文本: 济南至西安，往返五天自驾游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [16] 真实=combined | 预测=route (置信度=0.879) | 文本: 目的地有什么好玩的吗
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [17] 真实=combined | 预测=route (置信度=0.809) | 文本: 一家4口自驾去云南
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [44] 真实=combined | 预测=route (置信度=0.832) | 文本: 周五晚上从济南出发，周日下午回济南，自己一个人去，旅游偏好玩女人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [46] 真实=route | 预测=combined (置信度=0.539) | 文本: 洛阳6天5晚"赏花+吃喝+亲子"行程（4.12-4.17）
Day 1（4.12）：抵达洛阳，初尝水席
下午：抵达洛阳龙门站，入住酒店（推荐洛邑古城或应天门附近
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [48] 真实=route | 预测=combined (置信度=0.809) | 文本: 📅 每日行程与美食规划

Day 1｜周五晚：夜色初遇柳州

路线：抵达柳州 → 柳江沿岸夜景散步 → 胜利夜市
美食推荐：
胜利夜市：
螺蛳粉：柳州最具代表性
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [51] 真实=combined | 预测=transportation (置信度=0.539) | 文本: 唐山到武汉看樱花
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [54] 真实=route | 预测=transportation (置信度=0.361) | 文本: 惠山风景区门票
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [59] 真实=combined | 预测=route (置信度=0.488) | 文本: 张掖，5天，亲子两个人，景点打卡，时间宽裕点，不要太赶，预算1万以内
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [78] 真实=route | 预测=transportation (置信度=0.715) | 文本: 宁波出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [81] 真实=transportation | 预测=combined (置信度=0.926) | 文本: 去北海 4天3晚 2人情侣 ➕机票人均2千
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [85] 真实=combined | 预测=transportation (置信度=0.961) | 文本: 5月1日出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [93] 真实=combined | 预测=route (置信度=0.516) | 文本: 我打算从南宁出发规划一条自驾路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [95] 真实=combined | 预测=route (置信度=0.953) | 文本: 葫芦岛出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [97] 真实=route | 预测=other (置信度=0.852) | 文本: 北京
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [102] 真实=combined | 预测=route (置信度=0.637) | 文本: 3天，黄山，重庆出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [115] 真实=route | 预测=combined (置信度=0.719) | 文本: 从上海出发自驾3-4小时旅游计划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [120] 真实=combined | 预测=hotel (置信度=0.801) | 文本: 我第三天和第四天要住和田归址啊
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [124] 真实=route | 预测=combined (置信度=0.582) | 文本: 6.12到宁波，6.20早上要去上海坐飞机，给我一个走遍江浙沪不绕路打卡有名景点的攻略
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [141] 真实=transportation | 预测=combined (置信度=0.426) | 文本: 便宜一点的
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [143] 真实=transportation | 预测=combined (置信度=0.727) | 文本: 杭州到澳门，周五出发周日回，行程规划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [149] 真实=combined | 预测=route (置信度=0.477) | 文本: 4月到6月之间，出发城市 无锡 镇江 南京都可以，想去新疆或者西藏等地区玩
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [153] 真实=route | 预测=combined (置信度=0.680) | 文本: 想去北京，3天两晚，2个人一起，行程不需要太赶，看风景，拍照出片，以及吃饭的地方，预算2000一个人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [156] 真实=combined | 预测=transportation (置信度=0.770) | 文本: 清明机票最便宜路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [172] 真实=combined | 预测=transportation (置信度=0.777) | 文本: 停车停在哪里 在古城里要看吊脚楼
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [176] 真实=combined | 预测=route (置信度=0.910) | 文本: 四月十三号要去成都 后去重庆 玩四天 必须看熊猫 帮我规划一下计划一共得花多少钱
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [177] 真实=combined | 预测=transportation (置信度=0.809) | 文本: 武汉 早晨六点到 大概晚上七点坐动车回黄冈 两个人带妈妈
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [191] 真实=transportation | 预测=combined (置信度=0.441) | 文本: 摩托车旅行
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [196] 真实=combined | 预测=route (置信度=0.531) | 文本: 两个家庭 分别是三岁半 七岁半 八岁半三个孩子 4月2日至5日三亚亲子游，从萧山机场出发，制定一个详细的旅行攻略
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [198] 真实=combined | 预测=transportation (置信度=0.727) | 文本: 想從廣州出發去芒市和西雙版納，要怎樣安排
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [218] 真实=combined | 预测=route (置信度=0.852) | 文本: 2人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [221] 真实=combined | 预测=route (置信度=0.590) | 文本: 我想从宁德出发到江门新会，我只能在星期五下午五点下班后出发，我想星期六早上就到新会，玩两天后，星期天下午或者晚上回宁德，我就一个人，想到新会市区走一走
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [237] 真实=route | 预测=combined (置信度=0.934) | 文本: 本人目前住在桂林市临桂区，计划年轻夫妻带上两个老人到阳朔自驾游。规划行程两天一晚，要求强度适中，白天傍晚都安排活动，兼顾自然风光和人文体验。酒店可考虑四星与五星
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [240] 真实=route | 预测=other (置信度=0.910) | 文本: 重庆美食3日游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [245] 真实=other | 预测=route (置信度=0.918) | 文本: 我是旅行社的 我该怎么上线路
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [247] 真实=transportation | 预测=other (置信度=0.676) | 文本: 之前带她做过，我没有要啊。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [269] 真实=combined | 预测=route (置信度=0.498) | 文本: 我打算从上海出发去新疆游玩 7天 五月十日去 #情侣出行
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [275] 真实=combined | 预测=route (置信度=0.840) | 文本: 郴州莽山五一假期四天三夜
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [291] 真实=other | 预测=route (置信度=0.945) | 文本: 去大连、烟台、威海，淡季是什么时间段？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [295] 真实=combined | 预测=transportation (置信度=0.871) | 文本: 6个人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [301] 真实=route | 预测=transportation (置信度=0.820) | 文本: 推荐国内的，适合这个时节
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [324] 真实=combined | 预测=route (置信度=0.570) | 文本: 广州长隆珠海长隆澳门香港五一游玩大连出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [329] 真实=other | 预测=route (置信度=0.742) | 文本: 需要花多少钱
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [349] 真实=route | 预测=combined (置信度=0.793) | 文本: 我想五一假期带孩子去西双版纳。我们一家四口人。我是河北邯郸市。可以接受提前出发，如何安排行程和景点。最省钱
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [367] 真实=route | 预测=combined (置信度=0.602) | 文本: 周末海边度假，要求舒适
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [373] 真实=combined | 预测=route (置信度=0.707) | 文本: 给我一个洛阳三日游，从西安出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [387] 真实=combined | 预测=route (置信度=0.727) | 文本: 我计划周五晚上从盐城飞广州，周日再从广州回盐城，期间终点想去顺德品尝美食，请帮我规划一下行程路线！
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [393] 真实=combined | 预测=route (置信度=0.918) | 文本: 厦门至河北游线路行程安排
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [399] 真实=transportation | 预测=route (置信度=0.344) | 文本: 从丹东出发回丹东 安排顺路
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [404] 真实=route | 预测=combined (置信度=0.906) | 文本: 第一天中午到北京前门青云阁酒店，共住四晚，第五天下午6点火车回家，想去长城故宫颐和园，天坛，天安门路边拍照，再想去坐黄包车，逛逛街
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [409] 真实=route | 预测=other (置信度=0.463) | 文本: 五月
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [414] 真实=transportation | 预测=combined (置信度=0.879) | 文本: 从西安出发到黑龙江，我想在黑龙江玩几天然后坐一辆慢火车穿过大兴安岭
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [417] 真实=route | 预测=combined (置信度=0.699) | 文本: 宜兴到苏州2人情侣，希望人文风景多一点
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [418] 真实=combined | 预测=route (置信度=0.531) | 文本: 4月5日从哈尔滨出发 前往江西旅游 5天四晚 想去南昌 景德镇 婺源 篁岭 望仙谷 请出一份详细攻略 包括 吃住行 前往景点的时间路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [425] 真实=combined | 预测=hotel (置信度=0.633) | 文本: 离我朋友还是太远
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [429] 真实=other | 预测=combined (置信度=0.914) | 文本: 两个人大概所需预算是多少
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [441] 真实=route | 预测=combined (置信度=0.641) | 文本: 去杭州，计划3天，亲子游，人均3000
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [447] 真实=combined | 预测=transportation (置信度=0.898) | 文本: 高铁最好在两小时内的那种
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [453] 真实=combined | 预测=route (置信度=0.852) | 文本: 大同2天1晚游完路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [455] 真实=other | 预测=route (置信度=0.934) | 文本: 带一岁半小娃
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [460] 真实=combined | 预测=hotel (置信度=0.473) | 文本: 3月12日广州飞北京帮我顺便找一下北京的酒店
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [476] 真实=combined | 预测=transportation (置信度=0.914) | 文本: 你给出的答案我不满意，第一：我需要在周五下班后才出发，所以航班起飞时间应该在18:00以后。第二：你给出的价格并不满足我的要求，我需要往返一千以内的价格
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [491] 真实=route | 预测=other (置信度=0.656) | 文本: 我要去贵阳参加分类考试，3月14号开始考，3月13号就要到 13考完试等着18号去体检
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [494] 真实=route | 预测=transportation (置信度=0.898) | 文本: 根据我订的规划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [507] 真实=other | 预测=transportation (置信度=0.770) | 文本: 你真傻逼
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [508] 真实=route | 预测=combined (置信度=0.486) | 文本: 列出详细预算表。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [523] 真实=combined | 预测=route (置信度=0.723) | 文本: 3月6日杭州出发到云南三天旅游推荐
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [525] 真实=route | 预测=combined (置信度=0.404) | 文本: 大約下午二時扺別府
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [528] 真实=transportation | 预测=route (置信度=0.754) | 文本: 蚌埠去九寨沟出行推荐
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [547] 真实=combined | 预测=route (置信度=0.762) | 文本: 家人比较喜欢坐飞机，还不喜欢长途跋涉做太久火车。从沈阳出发，能帮我规划一个好玩完美的两天目的地和行程吗？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [554] 真实=route | 预测=combined (置信度=0.699) | 文本: 江门出发，济州岛4天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [578] 真实=route | 预测=combined (置信度=0.723) | 文本: 白云山要门票吗加缆车
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [589] 真实=combined | 预测=transportation (置信度=0.664) | 文本: 宁波到贵州旅行
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [590] 真实=route | 预测=hotel (置信度=0.848) | 文本: 离白云山风景区远吗
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [604] 真实=route | 预测=transportation (置信度=0.730) | 文本: 要当天到
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [634] 真实=route | 预测=combined (置信度=0.496) | 文本: 亲子游，14-17岁的孩子，四天，广西河池金城江出发，出省
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [678] 真实=route | 预测=combined (置信度=0.645) | 文本: 洛阳到开封三天两晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [682] 真实=route | 预测=other (置信度=0.793) | 文本: 廈門必去必吃必買行程
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [687] 真实=route | 预测=other (置信度=0.684) | 文本: 森德照明(裕祥工业区)
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [693] 真实=transportation | 预测=route (置信度=0.691) | 文本: 南充到西安两天一夜
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [694] 真实=route | 预测=combined (置信度=0.762) | 文本: 带四岁孩子亲子游，威海出发常州
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [706] 真实=combined | 预测=route (置信度=0.883) | 文本: 我从3月9号晚上21:00到丽江，3月13号早上10:30的飞机离开丽江，当中两天我还想去大理游玩，请帮我规划行程
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [722] 真实=combined | 预测=transportation (置信度=0.656) | 文本: 想要一条从深圳到桂林，还要一条深圳到三亚的规划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [730] 真实=transportation | 预测=combined (置信度=0.715) | 文本: 计划四月末或者五月初，从吉隆坡到成都，成都玩一下再去沈阳
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [734] 真实=route | 预测=combined (置信度=0.816) | 文本: 6天五晚北京出发去越南
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [736] 真实=route | 预测=transportation (置信度=0.461) | 文本: 预约好了第一天去国博的票
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [741] 真实=transportation | 预测=other (置信度=0.809) | 文本: 做硬铺多少钱？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [742] 真实=hotel | 预测=route (置信度=0.824) | 文本: 不要昆山、苏州、无锡、宜兴的
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [750] 真实=combined | 预测=transportation (置信度=0.941) | 文本: 到我住的酒店乘坐地铁路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [759] 真实=route | 预测=combined (置信度=0.594) | 文本: 长沙出发周末游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [761] 真实=combined | 预测=transportation (置信度=0.914) | 文本: 有交通方式吗，5-7天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [764] 真实=route | 预测=other (置信度=0.871) | 文本: 每年升旗五月一人多吗
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [766] 真实=other | 预测=combined (置信度=0.758) | 文本: 我在青岛
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [811] 真实=hotel | 预测=combined (置信度=0.531) | 文本: 推荐几个伊斯坦布尔和安卡拉，性价比高，出行方便的酒店价格在800以内
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [813] 真实=combined | 预测=route (置信度=0.703) | 文本: 3月11号威海去云南玩 带妈妈姥姥 15号我从云南飞宁波 妈妈姥姥飞回威海
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [818] 真实=route | 预测=transportation (置信度=0.969) | 文本: 这个月就可以，北京出发，到哪都可以，天气舒服就好，不要大城市，出行方式随便
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [831] 真实=route | 预测=combined (置信度=0.555) | 文本: 自己一个人想去山西玩，要玩到山西所有的主要景点，帮我安排一下整个行程路线，我从苏州出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [838] 真实=combined | 预测=route (置信度=0.535) | 文本: 四天三晚或五天四晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [845] 真实=route | 预测=combined (置信度=0.699) | 文本: 我自驾开车从北京立水桥出发，到大兴经海海路附近接人，想去开车不超过两个小时的，人比较少的地方，景色比较好的地方去旅游，请帮我规划一下路线。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [850] 真实=route | 预测=combined (置信度=0.602) | 文本: 带两个男孩，一个9岁一个6岁，4.1-4.6出行，帮我规划一个低价出行地点，出成都
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [869] 真实=combined | 预测=transportation (置信度=0.926) | 文本: 5天的
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [880] 真实=other | 预测=combined (置信度=0.648) | 文本: 我这个路线怎么吃
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [898] 真实=combined | 预测=route (置信度=0.770) | 文本: 五天四晚去韩国
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [902] 真实=combined | 预测=route (置信度=0.641) | 文本: 郑州及其周边 10号到12号 13号9点的飞机
2个人，其中一个是老人
不要太累，要有户外。
现在想去的景点如下:
大观音寺庙
陈訾花卉市场
人民公园
建业小镇
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [903] 真实=combined | 预测=route (置信度=0.432) | 文本: 平潭岛自驾
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [905] 真实=other | 预测=route (置信度=0.953) | 文本: 我需要图片为我清晰展示
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [915] 真实=route | 预测=other (置信度=0.621) | 文本: 龙山购物攻略
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [918] 真实=hotel | 预测=other (置信度=0.609) | 文本: 餐厅推荐
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [922] 真实=route | 预测=hotel (置信度=0.645) | 文本: 苏州或杭州有农家乐吗
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [927] 真实=other | 预测=route (置信度=0.605) | 文本: 不打卡景点，购物，美食，皮肤科，理发四件事
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [942] 真实=route | 预测=combined (置信度=0.508) | 文本: 从江油出发去西安自驾游3天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [962] 真实=other | 预测=transportation (置信度=0.828) | 文本: 怎么看不懂接车车牌号？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [965] 真实=combined | 预测=hotel (置信度=0.922) | 文本: 交通的话住哪里方便
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [968] 真实=hotel | 预测=transportation (置信度=0.582) | 文本: 北戴河洲顿亚朵酒店一晚价格？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [974] 真实=other | 预测=transportation (置信度=0.680) | 文本: 还是要我自己找啊
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1032] 真实=transportation | 预测=combined (置信度=0.648) | 文本: 我3月5号到达江苏无锡，想住在八佰伴东亭附近，然后3月8号或者3月9号、3月10号飞到广东
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1048] 真实=combined | 预测=route (置信度=0.652) | 文本: 只有两天玩的时间，第三天就要回
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1051] 真实=hotel | 预测=combined (置信度=0.641) | 文本: 3月5号到8号在泰国林查班港
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1072] 真实=route | 预测=transportation (置信度=0.766) | 文本: 重庆周边游低价票
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1087] 真实=route | 预测=combined (置信度=0.492) | 文本: 成都去荔波行程安排
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1091] 真实=combined | 预测=hotel (置信度=0.859) | 文本: 我要高性价比的酒店，重点在吃和玩上面
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1105] 真实=route | 预测=combined (置信度=0.699) | 文本: 明天下午重庆出发去周边玩2天 预算较低
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1109] 真实=other | 预测=combined (置信度=0.434) | 文本: 预算
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1118] 真实=transportation | 预测=hotel (置信度=0.463) | 文本: 正定恒阳酒店（正定古城夜市店到正定机场多远）
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1119] 真实=hotel | 预测=combined (置信度=0.486) | 文本: 6号下午到8号中午走
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1136] 真实=route | 预测=combined (置信度=0.531) | 文本: 周日下午返程
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1139] 真实=combined | 预测=other (置信度=0.691) | 文本: 你觉得如果我在日本旅居三个月，韩国旅居两个月，一共带多少钱？韩国是不是没什么好吃的。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1142] 真实=transportation | 预测=other (置信度=0.824) | 文本: Jakarta to chongqing return ticket
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1150] 真实=transportation | 预测=combined (置信度=0.426) | 文本: 西安到威海周末三天，
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1156] 真实=combined | 预测=transportation (置信度=0.676) | 文本: 5.2-7日，温州-大连
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1169] 真实=combined | 预测=other (置信度=0.332) | 文本: 价格估算
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1186] 真实=combined | 预测=route (置信度=0.590) | 文本: 澳门两天一晚旅游规划，拱北口岸进入
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1188] 真实=route | 预测=hotel (置信度=0.945) | 文本: 都城大厦到关帝庙之间
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1201] 真实=other | 预测=combined (置信度=0.711) | 文本: 潮州，汕头，3天1个人，美食
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1228] 真实=combined | 预测=route (置信度=0.895) | 文本: 2天 周五晚去，周日晚回
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1239] 真实=transportation | 预测=route (置信度=0.486) | 文本: 带3岁宝宝成都出发去哪比较方便
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1245] 真实=combined | 预测=transportation (置信度=0.938) | 文本: 北京到沈阳，途中会路过哪些地方，怎么回沈阳最省钱
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1265] 真实=combined | 预测=route (置信度=0.969) | 文本: 3天2晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1284] 真实=combined | 预测=transportation (置信度=0.684) | 文本: 3月12日左右先去深圳，然后3天后再从深圳去成都
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1290] 真实=route | 预测=combined (置信度=0.484) | 文本: 两大三小在日照，明天晚饭前回潍坊，今天下雨，不介意绕点远附近城市都可以
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1291] 真实=route | 预测=combined (置信度=0.684) | 文本: 8月份常州到花鸟岛枸杞岛5天4晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1299] 真实=transportation | 预测=combined (置信度=0.836) | 文本: 时间不限，想要坐船途中旅游一圈
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1309] 真实=route | 预测=other (置信度=0.555) | 文本: 周四中午上完课，周一早上需要上课
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1310] 真实=combined | 预测=route (置信度=0.559) | 文本: 贵阳北到千户苗寨的路线规划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1314] 真实=combined | 预测=route (置信度=0.848) | 文本: 8月份，亲子游5-8天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1329] 真实=route | 预测=transportation (置信度=0.486) | 文本: 先到枸杞和先到花鸟岛有区别吗
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1334] 真实=route | 预测=other (置信度=0.715) | 文本: 嘉禾望岗商圈有什么
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1340] 真实=combined | 预测=hotel (置信度=0.734) | 文本: 6天5夜
麗江住2天旅程住宿在去香格里拉4日3放旅程住宿
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1341] 真实=combined | 预测=route (置信度=0.895) | 文本: 成都出发，风景历史
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1364] 真实=other | 预测=combined (置信度=0.891) | 文本: 可以
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1377] 真实=route | 预测=combined (置信度=0.570) | 文本: 去揭阳潮州汕头计划7天 3个人主要为吃美食，住宿一般即可
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1380] 真实=combined | 预测=route (置信度=0.883) | 文本: 贺州到桂林，玩两天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1389] 真实=combined | 预测=transportation (置信度=0.805) | 文本: 新疆_开封
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1391] 真实=transportation | 预测=combined (置信度=0.605) | 文本: 温州到长沙成都一个星期两个大人两个小孩
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1393] 真实=combined | 预测=hotel (置信度=0.875) | 文本: 选择近西湖酒店，自由行，公共交通便利，价位低于1000元每晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1398] 真实=hotel | 预测=combined (置信度=0.609) | 文本: 住宿到哪里，合适的房价
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1404] 真实=combined | 预测=route (置信度=0.496) | 文本: 4月25日大庆自驾浙江，5月4日返回大庆，带老人77岁，请给规划一下旅游攻略，往返沿途景点，必去的地方杭州，乌镇
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1407] 真实=route | 预测=other (置信度=0.840) | 文本: 马来西亚8日游，顺序为槟城亚庇吉隆坡，增加建议美食
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1411] 真实=other | 预测=route (置信度=0.408) | 文本: 这里面每个大概的费用是多少
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1412] 真实=route | 预测=combined (置信度=0.602) | 文本: 榆林市区，吃喝玩乐，两日计划
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1416] 真实=combined | 预测=transportation (置信度=0.613) | 文本: 在沛县自驾游去黄山市自驾游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1433] 真实=route | 预测=combined (置信度=0.609) | 文本: 13号晚出发去开封万岁山，16号回
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1441] 真实=combined | 预测=route (置信度=0.961) | 文本: 两个人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1444] 真实=other | 预测=route (置信度=0.688) | 文本: 日期9号
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1450] 真实=combined | 预测=route (置信度=0.609) | 文本: 温州去洛阳三天两夜
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1470] 真实=transportation | 预测=combined (置信度=0.574) | 文本: 两人去广西游大概需要多少钱？不包机票。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1474] 真实=route | 预测=combined (置信度=0.918) | 文本: 青岛高铁到河南8天左右。洛阳郑州安阳开封。两个大人。偏好不太累，文化古迹及老城生活
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1480] 真实=combined | 预测=route (置信度=0.730) | 文本: 潮汕5天4晚旅行
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1483] 真实=route | 预测=combined (置信度=0.859) | 文本: 1 出行3天2晚
2 4个人
3 家庭出游 
4 预算5K
南京出发 莫干山 安吉
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1488] 真实=route | 预测=combined (置信度=0.891) | 文本: 三位女士3月13-15日游武汉，13日中午抵达武汉天河机场，想去游玩湖北博物馆、黄鹤楼、东湖、武汉大学、黎黄陂路、江汉路、长江大桥、粮道街、昙华林、华中科技大学
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1502] 真实=transportation | 预测=combined (置信度=0.906) | 文本: 28日从太原到北京，找个酒店，晚上去鸟巢
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1505] 真实=other | 预测=combined (置信度=0.531) | 文本: 深度游玩惠山风景区，最省钱的特色小吃美食，景点推荐
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1509] 真实=combined | 预测=route (置信度=0.602) | 文本: 根据以上图片信息，制定一份6月旅行出行计划，6月带父母去陕西旅游，做一份详细的旅游攻略，吃住行，适合老年人旅游，上午一个景点下午一个景点，时间自由不紧张，时间7
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1511] 真实=combined | 预测=transportation (置信度=0.742) | 文本: 哦，我看见他那个地图上面有那个观光车，观光车是不是要钱的？观光车多少钱一个人？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1516] 真实=combined | 预测=route (置信度=0.633) | 文本: 2人其中一位七十多岁，从新乡出发，坐普通火车去苏州杭州上海7天游，帮我作为一个新城规划。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1538] 真实=route | 预测=other (置信度=0.762) | 文本: 厦门不太行，厦门的美食和福州的美食差不多，想找一个有特色美食的地方旅游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1561] 真实=combined | 预测=route (置信度=0.938) | 文本: 4.1日到4.5日，有海滩，有风土人情
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1562] 真实=combined | 预测=route (置信度=0.926) | 文本: 针对刚才提到的新加波和马来西亚旅游
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1570] 真实=combined | 预测=transportation (置信度=0.812) | 文本: 搜索低价机票，目的地标签类型：当季去哪儿
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1575] 真实=hotel | 预测=route (置信度=0.867) | 文本: 预算200一天打算玩3-4天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1582] 真实=combined | 预测=hotel (置信度=0.459) | 文本: 要入住弥勒两天 不走回头路
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1587] 真实=route | 预测=transportation (置信度=0.883) | 文本: 5月18日出发，舟山到咸阳，三天
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1609] 真实=combined | 预测=other (置信度=0.512) | 文本: 一共需要多少钱？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1612] 真实=other | 预测=route (置信度=0.676) | 文本: 一家三口
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1624] 真实=route | 预测=other (置信度=0.535) | 文本: 看演唱会安排
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1627] 真实=combined | 预测=route (置信度=0.844) | 文本: 我不去篁岭
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1630] 真实=combined | 预测=transportation (置信度=0.566) | 文本: 下周二早上落地上海，打算逛一下武康路和夜逛外滩，周三周四迪士尼，周五潜水艇博物馆，请帮我生成乘坐交通工具图
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1636] 真实=other | 预测=route (置信度=0.461) | 文本: 亚洲四小龙有哪些
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1651] 真实=combined | 预测=route (置信度=0.941) | 文本: 3月7号2点到杭州东站，当天可以去哪里玩，3月8号要去灵隐寺可以给我规划一个2天一夜的游玩吗，还有两个3岁小朋友
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1659] 真实=route | 预测=other (置信度=0.719) | 文本: 去广州买衣服
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1669] 真实=route | 预测=other (置信度=0.594) | 文本: 换一个
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1673] 真实=combined | 预测=route (置信度=0.816) | 文本: 我3月5号或者7号准备从苏州到崇左玩，想看德天瀑布，还有弄岗自然保护区，太平古城，还可以安排一些其他的方便去又好玩或有好吃的地方
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1678] 真实=route | 预测=combined (置信度=0.938) | 文本: 不需要离浴场近 选好玩的 路程方便的 一天消费500左右
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1687] 真实=transportation | 预测=combined (置信度=0.734) | 文本: 温州出发，飞机出行，不坐波音，3月14号和15号周末，想周五请假一天或者周一请假一天出去玩。来回机票加机建燃油不超过800元
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1692] 真实=hotel | 预测=combined (置信度=0.498) | 文本: 我要3/6号到武汉玩三天，给我安排在交通比较便捷的地方，能覆盖热门景点，最好离地铁口不要太远，能步行几分钟就到达目的地，离小吃街近，酒店折扣价格低于200，但不
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1694] 真实=combined | 预测=route (置信度=0.785) | 文本: 常州出发去江西周五下午出发周日下午往返帮我规划路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1698] 真实=other | 预测=route (置信度=0.879) | 文本: 全天的
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1701] 真实=combined | 预测=transportation (置信度=0.793) | 文本: 需要
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1714] 真实=combined | 预测=transportation (置信度=0.738) | 文本: 需要
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1717] 真实=transportation | 预测=route (置信度=0.551) | 文本: 归址位于策勒兰干阿克旁
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1723] 真实=route | 预测=combined (置信度=0.719) | 文本: 美食为主，第一天在渔人码头会展中心附近，第二天银河综艺馆附近，第三天回程氹仔附近
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1727] 真实=combined | 预测=route (置信度=0.949) | 文本: 含恭王府和国子监
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1731] 真实=route | 预测=combined (置信度=0.500) | 文本: 可以根据行程出一个地图
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1739] 真实=combined | 预测=hotel (置信度=0.777) | 文本: 4月4-6
去清遠 其中一天去長隆
最方便的民宿
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1743] 真实=route | 预测=transportation (置信度=0.535) | 文本: 北京出发往南走，先去南京还是苏杭
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1744] 真实=other | 预测=transportation (置信度=0.805) | 文本: 四月初从东兴报团三日游需要多少钱？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1752] 真实=combined | 预测=route (置信度=0.945) | 文本: 从渭南出发，两女一男旅游攻略，四月四号2点后出发4.7号上午回到渭南
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1758] 真实=other | 预测=route (置信度=0.883) | 文本: 好的
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1767] 真实=combined | 预测=transportation (置信度=0.816) | 文本: 广州出发去新疆6月最高性价比路线
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1773] 真实=route | 预测=transportation (置信度=0.910) | 文本: 从济南从发走古丝绸之路
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1776] 真实=other | 预测=route (置信度=0.836) | 文本: 到成都考察商业项目
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1782] 真实=route | 预测=other (置信度=0.498) | 文本: 浙江附近有什么相对冷门的旅游城市，但是美食又比较出名
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1790] 真实=route | 预测=other (置信度=0.465) | 文本: 预算多少钱？
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1792] 真实=route | 预测=other (置信度=0.656) | 文本: 洗头可以在周六晚
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1794] 真实=combined | 预测=route (置信度=0.926) | 文本: 我们从杭州出发4~5个人自驾游，想去丽水福州泉州，然后想顺便有什么好的景点
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1820] 真实=other | 预测=route (置信度=0.918) | 文本: 中山詹园图片
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1826] 真实=route | 预测=transportation (置信度=0.809) | 文本: 13号下午或晚上出发 15号走
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1831] 真实=combined | 预测=transportation (置信度=0.840) | 文本: 要卧铺下铺2个，中铺一个
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1856] 真实=other | 预测=route (置信度=0.957) | 文本: 两个人
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1885] 真实=combined | 预测=route (置信度=0.898) | 文本: 周末北京到张家口怎么玩
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1886] 真实=route | 预测=combined (置信度=0.871) | 文本: 从北京出发到保定沧州德州潍坊周日早上从出发周四晚上到潍坊全车自驾游，注重美食
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1889] 真实=route | 预测=transportation (置信度=0.945) | 文本: 不是 是晚的时间太短了
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1896] 真实=combined | 预测=hotel (置信度=0.750) | 文本: 从广州出发
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1913] 真实=combined | 预测=route (置信度=0.691) | 文本: 香港到武漢賞櫻兩日遊。
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1919] 真实=combined | 预测=route (置信度=0.703) | 文本: 周一沈阳到南昌下午1点飞机到，然后去景德镇还是婺源？婺源打算去望仙谷和弦高古城，景德镇打算手工做陶瓷，最后是周四晚上9点的飞机飞回沈阳，请做行程规划，及路线，如
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1923] 真实=route | 预测=combined (置信度=0.377) | 文本: 我不想特种兵，就想睡的舒服点，然后游玩一下，如果前一天爬山了或者玩的比较晚，可能第二天中午才起
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1925] 真实=combined | 预测=route (置信度=0.801) | 文本: 天津到恩施大峡谷，张家界怎样玩
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1969] 真实=other | 预测=route (置信度=0.922) | 文本: 潮汕在哪
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1979] 真实=combined | 预测=transportation (置信度=0.730) | 文本: 导入机票酒店
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1980] 真实=transportation | 预测=combined (置信度=0.465) | 文本: 出发3月5 到三亚 3月9到深圳
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1983] 真实=transportation | 预测=other (置信度=0.727) | 文本: 签证怎办理
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1987] 真实=route | 预测=combined (置信度=0.508) | 文本: 从深圳出发去贵阳猴耳天坑，路上还可以在哪里游玩省时省力
2026-04-14 15:00:10 | WARNING  | src.trainer:_log_bad_cases:609 -   [1998] 真实=combined | 预测=route (置信度=0.973) | 文本: 川内适合的地方
2026-04-14 15:00:10 | INFO     | src.trainer:_log_bad_cases:618 - 📄 错误样本已保存: ./outputs_new/bad_cases.json
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.500: 覆盖率 0.976, 准确率 0.897, F1 0.895
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.550: 覆盖率 0.961, 准确率 0.903, F1 0.901
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.600: 覆盖率 0.944, 准确率 0.909, F1 0.907
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.650: 覆盖率 0.923, 准确率 0.919, F1 0.917
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.700: 覆盖率 0.899, 准确率 0.928, F1 0.926
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.750: 覆盖率 0.869, 准确率 0.940, F1 0.938
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.800: 覆盖率 0.836, 准确率 0.947, F1 0.946
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.850: 覆盖率 0.791, 准确率 0.961, F1 0.959
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.900: 覆盖率 0.729, 准确率 0.974, F1 0.973
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:729 - 阈值 0.950: 覆盖率 0.567, 准确率 0.993, F1 0.991
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:750 - 置信度阈值评估结果已保存: ./outputs_new/confidence_threshold_evaluation.json
2026-04-14 15:00:10 | INFO     | src.trainer:evaluate_with_confidence_threshold:751 - 🏆 最佳阈值: 0.950 (F1: 0.991, 覆盖率: 0.567)
2026-04-14 15:00:10 | INFO     | __main__:main:348 - ✅ 置信度阈值评估完成
2026-04-14 15:00:10 | INFO     | __main__:main:349 - 🏆 最佳阈值: 0.950
2026-04-14 15:00:10 | INFO     | __main__:main:350 - 📊 最佳F1分数: 0.991
2026-04-14 15:00:10 | INFO     | __main__:main:351 - 📈 最佳覆盖率: 0.567
2026-04-14 15:00:10 | INFO     | src.utils:log_memory_usage:423 - 🏁 完成时GPU内存使用:
2026-04-14 15:00:10 | INFO     | src.utils:log_memory_usage:424 -   总内存: 95.0 GB
2026-04-14 15:00:10 | INFO     | src.utils:log_memory_usage:425 -   已用内存: 0.6 GB
2026-04-14 15:00:10 | INFO     | src.utils:log_memory_usage:426 -   缓存内存: 0.7 GB
2026-04-14 15:00:10 | INFO     | src.utils:log_memory_usage:427 -   可用内存: 94.4 GB
2026-04-14 15:00:10 | INFO     | src.utils:clear_cache:433 - GPU缓存已清理
2026-04-14 15:00:10 | INFO     | __main__:main:375 - 🎯 程序执行完成

```
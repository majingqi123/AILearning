# W09 周计划：LLM 原理

- **所属阶段**：阶段四 LLM 原理、开源模型与部署（W9-W11）
- **起止日期**：
- **计划用时**：工作日 5 × 约 1.5 小时 + 周末约 3 小时，共约 10–11 小时
- **硬件要求**：与 W8 相同；只在周四采样实验和周末 KV Cache 实验用到小 GPT / GPU
- **主计划**：[AI 学习计划（23 周）](../AI学习计划-23周.md)
- **上一周**：[W08](W08-从零实现小GPT.md)
- **笔记位置**：`notes/W09/`，每天两个文件：学习笔记用 [学习笔记模板](../../notes/学习笔记模板.md)，自我总结用 [自我总结模板](../../notes/自我总结模板.md)

## 1. 本周目标

学完这一周，你应该能做到：

1. 讲清 BPE 分词在做什么，并手写一个简化版。
2. 讲清预训练、SFT、RLHF / DPO 各自解决什么问题。
3. 在 W8 小 GPT 上实现 temperature、top-k、top-p 等采样，并对比生成效果。
4. 讲清 KV Cache 缓存了什么、省掉了哪些重复计算。
5. 用自己的话写完“LLM 生成一个 Token 的全过程”。

## 2. 本周产出

| 产出 | 位置 | 对应日 |
|---|---|---|
| 简化 BPE 实现 | `labs/w09-llm/01-bpe.py` | 周一 |
| 预训练与 Scaling Laws 笔记 | `notes/W09/` | 周二 |
| 训练流程总图（预训练→对齐） | `notes/W09/training-pipeline.md` | 周三 |
| 采样参数对比 | `labs/w09-llm/02-sampling.py` | 周四 |
| KV Cache / Prefill-Decode 数据流图 | `notes/W09/` | 周五 |
| KV Cache 加速实验或实现笔记 | `labs/w09-llm/03-kv-cache/` | 周末 |
| “生成一个 Token 全过程”短文 | `notes/W09/one-token.md` | 周末 |

## 3. 开始前的准备

- [ ] 新建目录 `notes/W09/` 和 `labs/w09-llm/`。
- [ ] 确认 W8 的小 GPT 代码还能跑；本周会在其上加采样与（尽量）KV Cache。
- [ ] 克隆或浏览 [karpathy/minbpe](https://github.com/karpathy/minbpe)，周一对照。
- [ ] 可选参考：[rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch) 的分词与生成章节；[Happy-LLM](https://datawhalechina.github.io/happy-llm/) 对应章节。

---

## 4. 每日安排

### 周一：分词——BPE、WordPiece、SentencePiece

**学习目标**

1. 理解为什么不能直接用“空格分词”或“整词表”。
2. 掌握 BPE 的合并流程：统计频率 → 合并最高频相邻对 → 迭代。
3. 知道 WordPiece、SentencePiece 和 BPE 的定位差异即可。

**时间安排**：读资料 35 分钟，动手 40 分钟，笔记和自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [karpathy/minbpe](https://github.com/karpathy/minbpe) | 英文代码 | 必读 | README + 最简实现 |
| [Hugging Face：Byte-Pair Encoding tokenization](https://huggingface.co/learn/nlp-course/chapter6/5) | 英文 | 必读 | BPE 原理与例子 |
| [Happy-LLM：分词相关章节](https://datawhalechina.github.io/happy-llm/) | 中文 | 选读 | 中文术语与 SentencePiece 介绍 |
| [LLMs-from-scratch 分词章节](https://github.com/rasbt/LLMs-from-scratch) | 英文 | 选读 | 想看更完整实现时打开 |

**动手任务**

1. 用一小段英文文本，手写简化 BPE：从字符级开始，合并 N 次，打印词表增长过程。
2. 把同一句话分别用：空格分词、字符级、你的 BPE、（可选）`tiktoken` 或 HuggingFace tokenizer 编码，对比 Token 数。
3. 在笔记里用三句话区分 BPE / WordPiece / SentencePiece（不必深挖训练细节）。

**和已有知识的联系**：W8 用的是字符级词表，词表小但序列长。BPE 是在“词表大小”和“序列长度”之间折中——游戏资源里的“字典压缩”直觉很接近。

**验收标准**

- [ ] 简化 BPE 能对样例文本完成编码 / 解码往返。
- [ ] 能解释“未登录词”为什么在子词分词里不那么可怕。

**自测题**

1. BPE 训练时合并的依据是什么？
2. 为什么 LLM 几乎都用子词而不是整词？
3. 中文没有空格，SentencePiece 常见怎么处理？

---

### 周二：预训练——下一个 Token、数据与 Scaling Laws

**学习目标**

1. 讲清因果语言模型的预训练目标：预测下一个 Token。
2. 知道训练数据大致经历采集、清洗、去重、配比。
3. 理解 Scaling Laws 的直觉：参数量、数据量、算力怎么一起决定效果。

**时间安排**：读资料 50 分钟，整理笔记 25 分钟，自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Happy-LLM：预训练相关章节](https://datawhalechina.github.io/happy-llm/) | 中文 | 必读 | 预训练目标、数据、训练流程 |
| [OpenAI Scaling Laws 论文（Kaplan et al.）](https://arxiv.org/abs/2001.08361) | 英文 | 选读 | 只看摘要 + 主要结论图 |
| [Chinchilla 论文（Hoffmann et al.）](https://arxiv.org/abs/2203.15556) | 英文 | 选读 | 知道“数据量也要跟着参数量涨” |
| [Andrej Karpathy：State of GPT（视频）](https://www.youtube.com/watch?v=bZQunqJK9X0) | 英文 | 选读 | 预训练到对齐的全景，B 站可搜同名 |

**动手任务**

1. 写一页笔记：预训练在优化什么损失？和 W8 小 GPT 的训练目标是否相同？
2. 列一张表：数据侧常见步骤（采集 / 过滤 / 去重 / 配比）各防什么问题。
3. 用自己的话解释：同样算力下，模型过大而数据过少会怎样？

**验收标准**

- [ ] 能不看资料讲清 next-token prediction。
- [ ] 有 Scaling Laws 的三句话总结（不必记公式）。

**自测题**

1. 预训练为什么不直接用“问答对”当主要目标？
2. 去重主要是为了什么？
3. “涌现能力”和单纯堆参数是什么关系（你怎么看）？

---

### 周三：SFT、RLHF、DPO 与知识蒸馏

**学习目标**

1. 理解指令微调（SFT）如何把基座模型变成“能听指令”的助手。
2. 知道 RLHF（奖励模型 + PPO）在对齐什么；DPO / GRPO 简化了什么。
3. 理解知识蒸馏在 LLM 场景里常见用法。

**时间安排**：读资料 45 分钟，画流程图 25 分钟，自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Happy-LLM：对齐 / 微调相关章节](https://datawhalechina.github.io/happy-llm/) | 中文 | 必读 | SFT、RLHF、DPO |
| [InstructGPT 论文](https://arxiv.org/abs/2203.02155) | 英文 | 选读 | 摘要 + 方法概览图 |
| [DPO 论文](https://arxiv.org/abs/2305.18290) | 英文 | 选读 | 摘要：为何能去掉单独奖励模型 |
| [Hugging Face：RLHF 概念页](https://huggingface.co/blog/rlhf) | 英文 | 选读 | 流程图清楚 |

**动手任务**

1. 画一张训练流程图：Base Pretrain → SFT →（RLHF 或 DPO）→ 可对话模型。每个箭头旁写“输入数据是什么 / 优化目标是什么”。
2. 用表格对比 PPO-RLHF 与 DPO：要不要奖励模型、训练稳定性直觉、实现复杂度。
3. 写三句话：知识蒸馏在“大模型 → 小模型”时通常蒸馏什么（logits / 隐状态 / 偏好，知道即可）。

**验收标准**

- [ ] 流程图能给别人讲 5 分钟。
- [ ] 能说出 SFT 解决什么、RLHF/DPO 解决什么。

**自测题**

1. 只有预训练没有 SFT，模型聊天时常见什么问题？
2. 奖励模型的训练数据从哪来？
3. DPO 相对经典 RLHF 的主要工程好处是什么？

---

### 周四：推理采样——在小 GPT 上落地

**学习目标**

1. 理解自回归生成：每次只产一个 Token，再拼回输入。
2. 掌握 Greedy、temperature、top-k、top-p、重复惩罚的直观效果。
3. 在 W8 模型上实现并对比。

**时间安排**：读资料 20 分钟，动手 55 分钟，笔记和自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Hugging Face：Generation strategies](https://huggingface.co/docs/transformers/generation_strategies) | 英文 | 必读 | greedy / sampling / beam 概念 |
| [How to generate text（HF Blog）](https://huggingface.co/blog/how-to-generate) | 英文 | 必读 | temperature、top-k、top-p 图示 |
| W8 自己的 `generate` 代码 | — | 必读 | 在其上扩展 |

**动手任务**

1. 在小 GPT 上实现：
   - Greedy（取 argmax）
   - temperature
   - top-k
   - top-p（nucleus）
   - （可选）presence / frequency penalty 或简单重复抑制
2. 同一 prompt、同一 checkpoint，生成 5 组对比样例，写入 `labs/w09-llm/samples.md`。
3. 记录：哪种设置更稳？哪种更有创意但容易胡写？

**验收标准**

- [ ] 采样函数可切换策略。
- [ ] 有并排对比的生成样例与结论。

**自测题**

1. temperature → 0 时接近什么策略？
2. top-p=0.9 是在对什么集合做截断？
3. Beam Search 为什么在开放式聊天里不如采样常用？

---

### 周五：KV Cache、Prefill / Decode、幻觉

**学习目标**

1. 讲清 Prefill 与 Decode 两个阶段。
2. 理解 KV Cache：缓存了每层的 K、V，避免 Decode 时重复算历史。
3. 对幻觉有机制层面的直觉（不是背定义）。

**时间安排**：读资料 40 分钟，画图 30 分钟，自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [nanoGPT / 常见 KV Cache 讲解文](https://github.com/karpathy/nanoGPT) | 英文 | 必读 | 结合实现或注释理解 cache |
| [vLLM 文档：PagedAttention 背景](https://docs.vllm.ai/) | 英文 | 选读 | 先理解“KV 占显存”即可，Paged 细节周末再读 |
| [Happy-LLM：推理相关章节](https://datawhalechina.github.io/happy-llm/) | 中文 | 必读 | Prefill/Decode、KV Cache |
| [OpenAI：Hallucination 相关讨论 / 博客检索](https://openai.com/index/) | 英文 | 选读 | 建立“为何会编造”的直觉，不必深读论文 |

**动手任务**

1. 画数据流图：无 Cache 时，生成第 t 个 Token 要对长度为 t 的序列做哪些重复计算？有 Cache 后省掉哪些？
2. 估算：上下文长度从 1K 到 8K，KV Cache 显存大致如何变（写出和 batch、层数、头数、维度的关系，不必代入真实数字）。
3. 写一段“幻觉从哪来”：训练目标、检索缺失、采样随机性、过度自信——选你认同的 2–3 点。

**验收标准**

- [ ] 能讲清 Cache 的是 K/V 而不是注意力权重本身。
- [ ] 有 Prefill vs Decode 的对比笔记。

**自测题**

1. Prefill 阶段为什么可以并行算整个 prompt？
2. Decode 阶段通常为什么更吃显存带宽？
3. 加长上下文对 KV Cache 的主要代价是什么？

---

### 周末：小 GPT + KV Cache，写“一个 Token 全过程”

**学习目标**

1. 尽量给 W8 模型加上 KV Cache 并测速；做不出就精读实现并画流程图。
2. 把本周串成一篇短文：从输入文本到输出一个 Token。

**时间安排**（约 3 小时）：KV Cache 实验 100 分钟，短文 40 分钟，复盘 30 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [nanoGPT](https://github.com/karpathy/nanoGPT) | 英文 | 必读 | 生成路径与 cache 相关代码 |
| [llama.cpp](https://github.com/ggml-org/llama.cpp) | C/C++ | 选读 | 看 README 对量化与推理的说明，建立下下周直觉 |
| 本周全部笔记 | — | 必读 | 用来写全过程短文 |

**动手任务**

1. **路径 A（推荐）**：在小 GPT 的注意力里缓存过去的 K、V，Decode 时只算新 Token 的 Q 并与缓存的 K/V 做注意力。对比有无 Cache 的 tokens/s（同长度 prompt、同生成长度）。
2. **路径 B（备选）**：读 nanoGPT 或 llama.cpp 相关实现，画出：cache 的数据结构、写入时机、读取时机、形状变化。
3. 写 `notes/W09/one-token.md`（800–1500 字即可），覆盖：
   - 分词 → Embedding → 各层（注意力用到 Cache）→ logits → 采样 → 解码成文本
4. 更新 [进度追踪](../进度追踪.md)。

**验收标准**

- [ ] 有测速对比，或有完整实现流程图（二选一，推荐都有）。
- [ ] `one-token.md` 能给同事讲明白。
- [ ] 本周验收清单打勾。

---

## 5. 本周验收

对应主计划 W9 的验收：

- [ ] 能讲清楚从输入文本到输出一个 Token 的每一步。
- [ ] 能解释 KV Cache 缓存了什么、省掉了哪些重复计算。

## 6. 周复盘

- **实际用时**：
- **验收通过情况**：
- **本周最重要的三个收获**：
- **哪些自测题答不上来**：
- **遗留问题**（带到下周）：
- **哪种学习方式效果好，哪种不好**：
- **下周要调整的地方**：

## 7. 容易踩的坑

- **分词实现追求完美**：周一目标是理解算法，不是写生产级 tokenizer。
- **对齐算法陷进 PPO 公式**：抓住“数据与目标”，公式扫过即可。
- **采样参数一次改太多**：对照实验每次只改一个。
- **KV Cache 形状错了**：时间维是在增长的那一维，写注释标 `(batch, heads, seq, dim)`。

## 8. 下周预告

[W10](W10-HuggingFace与微调.md) 进入 Hugging Face 工具链：加载开源小模型、测显存与速度、理解 LoRA / QLoRA，并做一次小规模指令微调。

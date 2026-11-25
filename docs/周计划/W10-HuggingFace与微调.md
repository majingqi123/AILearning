# W10 周计划：Hugging Face 与微调

- **所属阶段**：阶段四 LLM 原理、开源模型与部署（W9-W11）
- **起止日期**：
- **计划用时**：工作日 5 × 约 1.5–2 小时 + 周末约 3–4 小时，共约 12 小时
- **硬件要求**：需要 NVIDIA 显卡。0.5B–1.5B 模型建议显存 6GB 以上；更大模型或本地 QLoRA 不够就用云端 GPU 做周三和周末实验
- **主计划**：[AI 学习计划（23 周）](../AI学习计划-23周.md)
- **上一周**：[W09](W09-LLM原理.md)
- **笔记位置**：`notes/W10/`，每天两个文件：学习笔记用 [学习笔记模板](../../notes/学习笔记模板.md)，自我总结用 [自我总结模板](../../notes/自我总结模板.md)

## 1. 本周目标

学完这一周，你应该能做到：

1. 用 Hugging Face Transformers 加载小型因果语言模型并对话。
2. 测出一份自己的显存、TTFT、TPS 数据表（含不同精度）。
3. 讲清 LoRA / QLoRA 相对全量微调省在哪里。
4. 准备一小份指令数据，跑通一次 PEFT 微调。
5. 能说明微调和 RAG 分别适合什么场景。

## 2. 本周产出

| 产出 | 位置 | 对应日 |
|---|---|---|
| 模型加载与环境记录 | `labs/w10-hf/00-setup.md` | 周一 |
| 对话与分词 Notebook | `labs/w10-hf/01-chat.ipynb` | 周二 |
| 性能数据表 | `labs/w10-hf/02-bench.md` | 周三 |
| LoRA / QLoRA 笔记 | `notes/W10/` | 周四 |
| 指令数据集 | `labs/w10-hf/data/` | 周五 |
| QLoRA 微调与前后对比 | `labs/w10-hf/03-qlora/` | 周末 |

## 3. 开始前的准备

- [ ] 新建目录 `notes/W10/` 和 `labs/w10-hf/`。
- [ ] 安装（版本按你环境微调即可）：`transformers`、`datasets`、`accelerate`、`peft`、`bitsandbytes`（Linux/合适 CUDA 环境；Windows 若 bitsandbytes 不顺，周末改用云端或 CPU/LoRA 小模型方案）。
- [ ] 选一个小模型作为本周主力，例如：
  - `Qwen/Qwen2.5-0.5B-Instruct` 或 `Qwen/Qwen2.5-1.5B-Instruct`
  - 或其他你能下载、显存放得下的 Instruct 小模型
- [ ] 提前确认 Hugging Face 访问（镜像或代理按你网络情况配置）。
- [ ] 阅读入口：[Hugging Face NLP Course](https://huggingface.co/learn/nlp-course)、[PEFT 文档](https://huggingface.co/docs/peft)。

---

## 4. 每日安排

### 周一：HF 生态与模型下载加载

**学习目标**

1. 分清 Model Hub、Transformers、Datasets、Tokenizers、PEFT 各自职责。
2. 按显存选好本周模型，成功 `from_pretrained`。

**时间安排**：读资料 30 分钟，环境与下载 45 分钟，笔记 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Hugging Face Learn（NLP Course）](https://huggingface.co/learn/nlp-course) | 英文 | 必读 | 第 1–2 章管线与库概览 |
| [Transformers 文档：快速开始](https://huggingface.co/docs/transformers/fast_start) | 英文 | 必读 | 加载模型的最小例子 |
| [Model Hub](https://huggingface.co/models) | 英文界面 | 必读 | 按任务 / 参数量筛选 |

**动手任务**

1. 写 `00-setup.md`：Python 版本、CUDA、`torch`/`transformers` 版本、选用的模型 ID、下载路径。
2. 用代码加载 tokenizer 与 `AutoModelForCausalLM`，打印参数量与 `device`。
3. 若下载失败：记录错误，改镜像或手动下载流程写进笔记（问题解决本身也是产出）。

**验收标准**

- [ ] 模型在本机或云端成功加载到可推理状态。
- [ ] 清楚自己选的模型大概参数量与占用。

**自测题**

1. `AutoModelForCausalLM` 和 Encoder-only 模型类有什么差别？
2. Hub 上模型名里的 `Instruct` / `Chat` 通常意味着什么？
3. PEFT 和 Transformers 是什么关系？

---

### 周二：Tokenizer、generate、对话模板

**学习目标**

1. 会用 `apply_chat_template` 构造多轮对话输入。
2. 会调用 `generate`，并查看分词结果。
3. 理解特殊 Token、注意力掩码在对话里的角色。

**时间安排**：读资料 25 分钟，动手 50 分钟，笔记和自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Chat templating](https://huggingface.co/docs/transformers/chat_templating) | 英文 | 必读 | 对话模板怎么用 |
| [Generation](https://huggingface.co/docs/transformers/main/en/generation_strategies) | 英文 | 必读 | `generate` 常用参数 |
| [Tokenizer summary](https://huggingface.co/docs/transformers/tokenizer_summary) | 英文 | 选读 | 分词器类型回顾 |

**动手任务**

1. 对同一句话打印：`tokenize`、解码回读、Token 数量。
2. 用 chat template 构造 system + user + assistant 多轮，生成回复。
3. 做 3 轮对话，把历史正确塞回 messages，观察模型是否记住前文。
4. （可选）取出某一层 Embedding 或最后隐状态，对两句近义/反义句算余弦相似度（为 W13 预热）。

**验收标准**

- [ ] 多轮对话脚本可运行。
- [ ] 能解释为什么要用模型配套的 chat template，而不是自己随便拼字符串。

**自测题**

1. 忘记 `add_generation_prompt` 可能导致什么现象？
2. `max_new_tokens` 和 `max_length` 区别是什么？
3. 特殊 Token 被错误解码成可见乱码时，优先查什么？

---

### 周三：测显存、TTFT、TPS

**学习目标**

1. 建立推理性能的基本指标：显存峰值、首 Token 延迟、生成速度。
2. 对比不同输入长度、FP16 与 4-bit 加载。

**时间安排**：读资料 15 分钟，压测 60 分钟，整理表格 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [bitsandbytes 文档 / HF 量化指南](https://huggingface.co/docs/transformers/quantization) | 英文 | 必读 | 4-bit 加载方式 |
| [Accelerate 文档](https://huggingface.co/docs/accelerate) | 英文 | 选读 | device map 相关 |
| W9 采样笔记 | — | 必读 | 生成参数保持一致，避免干扰对比 |

**动手任务**

1. 固定：模型、prompt、`max_new_tokens`、采样策略。
2. 测量并填表：
   - 精度：FP16（或 BF16） vs 4-bit
   - 输入长度：短 prompt vs 长 prompt（可用重复文本凑长度）
   - 指标：加载后显存、生成时峰值显存、TTFT、tokens/s
3. 写三句结论：你的机器上，量化主要省了什么？长上下文伤的是 TTFT 还是解码速度？

**验收标准**

- [ ] `02-bench.md` 里有完整数据表与结论。
- [ ] 测量方法写清楚，别人能复现。

**自测题**

1. TTFT 和 Decode TPS 分别主要受什么影响？
2. 4-bit 量化通常换来什么代价？
3. 为什么说“显存不够”和“算得慢”不是同一类问题？

---

### 周四：全量微调 vs LoRA / QLoRA

**学习目标**

1. 理解全量微调为什么贵（优化器状态、梯度、激活）。
2. 理解 LoRA：低秩矩阵、rank、alpha、注入位置。
3. 理解 QLoRA：量化底座 + LoRA 可训练适配器。

**时间安排**：读资料 50 分钟，笔记 25 分钟，自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [LoRA 论文](https://arxiv.org/abs/2106.09685) | 英文 | 必读 | 摘要、图 1、方法节 |
| [QLoRA 论文](https://arxiv.org/abs/2305.14314) | 英文 | 必读 | 摘要与核心思想 |
| [PEFT LoRA 文档](https://huggingface.co/docs/peft/conceptual_guides/lora) | 英文 | 必读 | `r`、`lora_alpha`、target modules |
| [Hugging Face：PEFT 概念](https://huggingface.co/docs/peft) | 英文 | 必读 | 快速浏览支持的方法 |

**动手任务**

1. 画图：全量微调 vs LoRA，标出“哪些权重在更新”。
2. 估算：7B 全量微调大致要存哪些状态；LoRA rank=8 时新增参数量量级（写公式即可）。
3. 记一张超参小抄：`r`、`lora_alpha`、`dropout`、常见 `target_modules`（如 `q_proj`/`v_proj`）。

**验收标准**

- [ ] 能不看论文讲清 LoRA 在拟合什么。
- [ ] 能说明 QLoRA 为什么能在消费级显卡上微调。

**自测题**

1. LoRA 的更新为什么是 \(\Delta W = BA\) 这种低秩形式？
2. `lora_alpha / r` 起什么作用？
3. 微调后“合并权重”是什么意思？合并前后部署有何不同？

---

### 周五：准备 50–200 条指令数据

**学习目标**

1. 做出一份格式正确、风格一致的小指令集。
2. 理解数据质量往往比算法花样更重要。

**时间安排**：设计格式 20 分钟，写数据 55 分钟，检查 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [Datasets 文档](https://huggingface.co/docs/datasets) | 英文 | 必读 | 从 json/jsonl 加载 |
| [Alpaca 数据格式（了解即可）](https://github.com/tatsu-lab/stanford_alpaca) | 英文 | 选读 | instruction / input / output 字段 |

**动手任务**

1. 选定垂直主题（推荐贴合你背景）：例如“解释这段 C++ 代码”“指出这段代码的风险”“把注释翻译成中文”。
2. 准备 50–200 条，建议 jsonl，字段统一，例如：`instruction`、`input`、`output`。
3. 抽 10 条做人工质检：有无泄漏答案、格式漂、中英混杂无规则、过短敷衍。
4. 划分 train / eval（哪怕 eval 只有 10–20 条）。

**验收标准**

- [ ] 数据文件可被 `datasets` 加载。
- [ ] 有简短 `data/README.md` 说明任务定义与格式。

**自测题**

1. 为什么评估集不能和训练集高度重复？
2. 指令数据太同质（同一句式刷很多条）会有什么风险？
3. 你的任务更适合微调，还是其实更适合 RAG？先写下判断，周末验证。

---

### 周末：QLoRA 微调与前后对比

**学习目标**

1. 用 PEFT 跑通一次 QLoRA（或显存不够时的 LoRA 小模型方案）。
2. 对比微调前后同题回答。
3. 了解 adapter 保存与权重合并。

**时间安排**（约 3–4 小时）：训练 120 分钟，对比评估 40 分钟，复盘 30 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [PEFT 训练示例](https://huggingface.co/docs/peft/quicktour) | 英文 | 必读 | LoRA 注入与保存 |
| [TRL / SFTTrainer 文档](https://huggingface.co/docs/trl) | 英文 | 选读 | 若用 SFTTrainer 更省事 |
| [Transformers 量化文档](https://huggingface.co/docs/transformers/quantization) | 英文 | 必读 | 4-bit 加载与训练注意点 |

**动手任务**

1. 加载基座（4-bit）+ 配 LoRA + 用你的数据训练若干 step/epoch（先求跑通，再求效果）。
2. 保存 adapter；写清如何加载适配器做推理。
3. 选 10 道评测题，表格式对比：微调前 / 微调后（正确性、格式服从、胡编情况）。
4. （可选）`merge_and_unload` 合并权重，记录合并前后体积与加载方式差异。
5. 在笔记里写清：这次任务如果改成 RAG，会怎么做？你更推荐哪条路？
6. 更新 [进度追踪](../进度追踪.md)。

**验收标准**

- [ ] 微调流程可复现（命令与关键超参写进 README）。
- [ ] 有前后对比表。
- [ ] 能口头说明微调 vs RAG 的适用边界。

---

## 5. 本周验收

对应主计划 W10 的验收：

- [ ] 有一份自己测的显存和速度数据。
- [ ] 完成一次微调，能说明它和 RAG 分别适合什么场景。

## 6. 周复盘

- **实际用时**：
- **验收通过情况**：
- **本周最重要的三个收获**：
- **哪些自测题答不上来**：
- **遗留问题**（带到下周）：
- **哪种学习方式效果好，哪种不好**：
- **下周要调整的地方**：

## 7. 容易踩的坑

- **模型选太大**：周一加载失败会拖垮整周；先 0.5B/1.5B。
- **Windows + bitsandbytes**：若不兼容，尽早转云端，不要空耗两天。
- **没设评测题就微调**：事后无法判断“变好了还是会背训练句”。
- **学习率与 epoch 过大**：小数据很容易过拟合，先短训看曲线。
- **把 API 模型当成本地微调对象**：本周对象是开源可下载权重的模型。

## 8. 下周预告

[W11](W11-部署与推理优化.md) 做本地部署：量化与 GGUF、Ollama、编译 llama.cpp、用 C++ 调推理，并把 CNN 导出 ONNX；周末补 vLLM / FlashAttention 原理与阶段四总结。

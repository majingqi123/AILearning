# W07 周计划：序列建模、注意力与 Transformer

- **所属阶段**：阶段三 深度学习（W5-W8）
- **起止日期**：
- **计划用时**：工作日 5 × 约 1.5 小时 + 周末约 3 小时，共约 10 小时
- **硬件要求**：无特殊要求；手写注意力用 CPU 即可
- **主计划**：[AI 学习计划（23 周）](../AI学习计划-23周.md)
- **上一周**：[W06](W06-PyTorch训练实践与CNN.md)
- **笔记位置**：`notes/W07/`，每天两个文件：学习笔记用 [学习笔记模板](../../notes/学习笔记模板.md)，自我总结用 [自我总结模板](../../notes/自我总结模板.md)

## 1. 本周目标

学完这一周，你应该能做到：

1. 讲清 RNN / LSTM 在解决什么问题、为什么难并行，以及为什么后来被 Transformer 取代。
2. 讲清注意力里的 Q、K、V，手写缩放点积注意力并打印权重。
3. 理解 Transformer 的主要部件：多头注意力、前馈层、残差、LayerNorm、位置编码、掩码。
4. 区分 BERT（Encoder-only）、GPT（Decoder-only）、T5（Encoder-Decoder）的用途。
5. 画出一张带张量形状的 Transformer 数据流图。

## 2. 本周产出

| 产出 | 位置 | 对应日 |
|---|---|---|
| RNN / Word2Vec 概念笔记 | `notes/W07/` | 周一 |
| 手写缩放点积注意力 | `labs/w07-transformer/01-attention.ipynb` | 周二 |
| Illustrated Transformer 读书笔记 | `notes/W07/` | 周三 |
| 手写 Multi-Head Attention | `labs/w07-transformer/02-mha.ipynb` | 周四 |
| BERT / GPT / T5 对比表 | `notes/W07/` | 周五 |
| Transformer 数据流图（含形状） | `notes/W07/transformer-shapes.md` 或图文件 | 周末 |

## 3. 开始前的准备

- [ ] 新建目录 `notes/W07/` 和 `labs/w07-transformer/`。
- [ ] 周三的主资料 [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) 是英文长文，可提前用翻译插件，或对照 [动手学深度学习的 Transformer 一节](https://zh.d2l.ai/chapter_attention-mechanisms/transformer.html)。
- [ ] 可选：B 站搜“Transformer 论文逐段精读”（李沐），约 1.5 小时，周末补看。

---

## 4. 每日安排

### 周一：序列、RNN / LSTM、词向量

**学习目标**

1. 知道序列数据为什么特殊（变长、有顺序依赖）。
2. 理解 RNN 的循环思路和长期依赖问题；知道 LSTM / GRU 在缓解什么。
3. 理解词向量：词变成稠密向量，相似词在空间中靠近。

**时间安排**：读资料 50 分钟，动手 20 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：循环神经网络](https://zh.d2l.ai/chapter_recurrent-neural-networks/index.html) | 中文 | 必读 | 序列模型、RNN 两节；不必跟写全部代码 |
| [动手学深度学习：LSTM](https://zh.d2l.ai/chapter_recurrent-modern/lstm.html) | 中文 | 必读 | 门控的直觉即可，公式扫一眼 |
| [动手学深度学习：词嵌入 Word2Vec](https://zh.d2l.ai/chapter_natural-language-processing-pretraining/word2vec.html) | 中文 | 必读 | 跳字模型和原理 |
| [Happy-LLM 第一章：NLP 基础概念](https://datawhalechina.github.io/happy-llm/) | 中文 | 选读 | 文本表示演进那一段 |

**动手任务**

1. 用自己的话画一张图：RNN 在时间步上如何展开，隐藏状态如何传递。
2. 写一段对比：RNN 为什么难以充分利用 GPU 并行；CNN 和后面的 Transformer 在并行上有什么不同。
3. （可选）用 `gensim` 或现成示例看几个词的最相似词；没有环境就读资料里的示意图。

**和已有知识的联系**：词向量和你熟悉的 Embedding 一样，都是把离散符号映射到连续向量。游戏里的“标签 → 属性向量”是手工的；Word2Vec 是从数据里学出来的。

**验收标准**

- [ ] 能讲清 RNN 的隐藏状态在传递什么。
- [ ] 能说出 LSTM 相比朴素 RNN 多了什么（门控 / 细胞状态），不必默写公式。
- [ ] 能解释“相似词的向量更接近”是什么意思。

**自测题**

1. 为什么说 RNN 难以并行训练？
2. “长期依赖”指的是什么问题？
3. Word2Vec 的训练目标直观上在做什么？

---

### 周二：注意力机制，手写缩放点积注意力

**学习目标**

1. 理解 Query、Key、Value 的角色。
2. 掌握公式 \(\mathrm{Attention}(Q,K,V)=\mathrm{softmax}(QK^\top/\sqrt{d_k})V\)。
3. 用 PyTorch 手写并打印注意力权重。

**时间安排**：读资料 35 分钟，动手 35 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：注意力机制](https://zh.d2l.ai/chapter_attention-mechanisms/index.html) | 中文 | 必读 | 注意力提示、注意力评分函数 |
| [动手学深度学习：注意力评分函数](https://zh.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html) | 中文 | 必读 | 缩放点积注意力 |
| [3Blue1Brown：Attention](https://www.3blue1brown.com/lessons/attention) | 英文，B 站有中文字幕 | 选读 | 视觉直觉很好 |

**动手任务**

1. 构造小例子：`seq_len=4`，`d_k=8`，随机生成 Q、K、V。
2. 手写缩放点积注意力：先算分数，再除以 \(\sqrt{d_k}\)，Softmax，再乘 V。
3. 打印注意力权重矩阵（`seq_len × seq_len`），确认每行和约为 1。
4. 把某一个 Query 改得和某个 Key 很像，观察权重是否集中到对应位置。
5. （可选）去掉缩放因子，把 `d_k` 调到 256，观察 Softmax 是否更“尖锐”。

**和已有知识的联系**：注意力权重就是一组归一化后的相似度。你在图形学里用点积判断朝向是否接近；这里用点积判断“当前 Token 该看哪些位置”。

**验收标准**

- [ ] 手写注意力能跑通，权重每行和约为 1。
- [ ] 能演示“相似的 Q/K 导致高权重”。
- [ ] 能解释为什么要除以 \(\sqrt{d_k}\)。

**自测题**

1. Q、K、V 各自扮演什么角色？
2. 为什么 Softmax 之前要缩放？
3. 注意力输出的形状由什么决定？

---

### 周三：Transformer 整体结构

**学习目标**

1. 能说出 Encoder / Decoder 里各有哪些子层。
2. 理解多头注意力、前馈层、残差、LayerNorm 各自干什么。
3. 知道数据在一层 Transformer 里怎么流。

**时间安排**：读资料 55 分钟，整理笔记 20 分钟，自测 15 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) | 英文 | 必读 | 全文，配图跟着走 |
| [动手学深度学习：Transformer](https://zh.d2l.ai/chapter_attention-mechanisms/transformer.html) | 中文 | 必读 | 对照英文文看，补中文术语 |
| [动手学深度学习：多头注意力](https://zh.d2l.ai/chapter_attention-mechanisms/multihead-attention.html) | 中文 | 必读 | 多头如何拼接与投影 |
| [Happy-LLM 第二章：Transformer 架构](https://datawhalechina.github.io/happy-llm/) | 中文 | 选读 | 想要更系统的中文讲解时看 |

**动手任务**

1. 按 Illustrated Transformer 的结构，自己画一张 Encoder 层和 Decoder 层的框图。
2. 在笔记里用表格列出：多头注意力、前馈网络、残差连接、LayerNorm——各解决什么问题。
3. 标出 Decoder 里三处注意力（自注意力、交叉注意力）分别看谁。

**验收标准**

- [ ] 有自己画的 Encoder / Decoder 框图。
- [ ] 能不看资料说出一层 Encoder 的计算顺序。
- [ ] 能区分 Decoder 自注意力和交叉注意力。

**自测题**

1. 为什么要用多头而不是单头注意力？
2. Transformer 里的前馈网络通常是几层？作用是什么？
3. 残差和 LayerNorm 在 Transformer 里通常怎么摆放（Pre-LN / Post-LN 知道有区别即可）？

---

### 周四：位置编码、掩码与 Multi-Head Attention 实现

**学习目标**

1. 理解为什么需要位置编码，正弦位置编码和可学习位置编码的区别。
2. 理解因果掩码和 Padding 掩码。
3. 手写一个简化的 Multi-Head Attention。

**时间安排**：读资料 30 分钟，动手 40 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：自注意力和位置编码](https://zh.d2l.ai/chapter_attention-mechanisms/self-attention-and-positional-encoding.html) | 中文 | 必读 | 全文 |
| [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | 英文 | 选读 | 只看第 3 节（模型）和第 3.2.3 节（位置编码） |
| [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) | 英文 | 必读 | 回顾掩码和多头部分 |

**动手任务**

1. 实现正弦位置编码，画前几个维度随位置变化的曲线。
2. 实现因果掩码：上三角为 `-inf`，加到注意力分数上，确认位置 i 看不到 j>i。
3. 手写 Multi-Head Attention：
   - 输入 `(batch, seq, d_model)`
   - 拆成 `num_heads` 个头
   - 每个头做缩放点积注意力
   - 拼接后再线性投影
4. 用小 batch 跑通，打印中间形状：拆头前后、注意力权重、输出。

**验收标准**

- [ ] 因果掩码生效（可视化权重上三角接近 0）。
- [ ] Multi-Head Attention 前向形状正确。
- [ ] 能解释位置编码解决了什么问题。

**自测题**

1. 如果没有位置编码，Transformer 对打乱顺序的句子会怎样？
2. 因果掩码用在什么模型里？为什么生成式模型需要它？
3. 多头注意力里 `d_k` 和 `d_model`、`num_heads` 通常是什么关系？

---

### 周五：BERT、GPT、T5 三种架构

**学习目标**

1. 分清 Encoder-only、Decoder-only、Encoder-Decoder。
2. 知道各自典型任务：理解 / 生成 / 序列到序列。
3. 明白现代生成式 LLM 为什么大多用 Decoder-only。

**时间安排**：读资料 45 分钟，整理对比表 25 分钟，自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：BERT](https://zh.d2l.ai/chapter_natural-language-processing-pretraining/bert.html) | 中文 | 必读 | BERT 在做什么（MLM、NSP 了解即可） |
| [The Illustrated GPT-2](https://jalammar.github.io/illustrated-gpt2/) | 英文 | 必读 | Decoder-only 怎么生成 |
| [Happy-LLM 第三章：预训练语言模型](https://datawhalechina.github.io/happy-llm/) | 中文 | 必读 | Encoder-only / Decoder-only / Encoder-Decoder 对比 |
| [3Blue1Brown：GPT](https://www.3blue1brown.com/lessons/gpt) | 英文，B 站有中文字幕 | 选读 | 生成过程的视觉讲解 |

**动手任务**

1. 做一张对比表，列至少这些行：架构、是否双向、典型预训练目标、典型下游任务、代表模型。
2. 写一段话：做 Embedding 检索（语义相似度）更倾向哪类模型？做聊天生成呢？做翻译呢？
3. 解释“现代 ChatGPT 类模型为什么是 Decoder-only”——从训练目标、工程复用、涌现能力里选你认同的理由，用自己的话写。

**验收标准**

- [ ] 对比表完成。
- [ ] 能根据任务选出更合适的架构类型。
- [ ] 能解释因果语言模型（next-token prediction）和 BERT 的掩码语言模型有何不同。

**自测题**

1. BERT 预训练时为什么可以双向看上下文？GPT 为什么不行？
2. T5 把所有任务都写成“文本到文本”是什么意思？
3. Embedding 模型常见是 Encoder 还是 Decoder？为什么？

---

### 周末：Transformer 数据流图与阶段复习

**学习目标**

1. 把本周概念串成一张带形状的数据流图。
2. 为 W8 从零实现小 GPT 做好形状直觉。

**时间安排**（约 3 小时）：画图 90 分钟，复习自测 40 分钟，补笔记 20 分钟，周复盘 30 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) | 英文 | 必读 | 对照自己的图查漏 |
| [李沐：Transformer 论文逐段精读](https://www.bilibili.com/video/BV1pu411o7BE/) | 中文 | 选读 | 约 1.5 小时，时间够就看 |
| [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | 英文 | 选读 | 对照图 1、图 2 |

**动手任务**

1. 假设：`batch=2`，`seq=8`，`d_model=32`，`num_heads=4`，`d_ff=64`。
2. 画 Decoder-only 一层的数据流，标出每一步张量形状：
   - 输入 Embedding + 位置编码
   - Q/K/V 投影、拆头
   - 注意力权重、加权输出、拼头、输出投影
   - 残差 + LayerNorm
   - 前馈网络上下两层
   - 再残差 + LayerNorm
3. 在图边注明哪里需要因果掩码。
4. 回头把本周所有自测题闭卷重做一遍，不会的标到遗留问题。

**验收标准**

- [ ] 数据流图形状前后一致（拆头、拼头对得上）。
- [ ] 本周验收清单全部打勾。
- [ ] 遗留问题不超过 3 个，且都有解决计划。

---

## 5. 本周验收

对应主计划 W7 的验收：

- [ ] 能解释 Q、K、V 各是什么，注意力权重怎么算出来。
- [ ] 能说出 Embedding 模型和生成模型在架构上的区别。

## 6. 周复盘

- **实际用时**：
- **验收通过情况**：
- **本周最重要的三个收获**：
- **哪些自测题答不上来**：
- **遗留问题**（带到下周）：
- **哪种学习方式效果好，哪种不好**：
- **下周要调整的地方**：

## 7. 容易踩的坑

- **把 RNN 公式抠太深**：本周目标是建立“为什么需要注意力”的动机，不是成为 RNN 专家。
- **多头形状搅在一起**：永远先写注释标形状，再写代码；`view` / `reshape` / `transpose` 的顺序错了就会静默算错。
- **掩码加错位置**：掩码加在 Softmax 之前的分数上，不是加在权重上。
- **只看动画不写代码**：周二、周四的手写不能跳；W8 会直接用到。

## 8. 下周预告

[W08](W08-从零实现小GPT.md) 跟 Karpathy 的 “Let's build GPT” 从零实现并训练一个字符级小 GPT。本周的注意力、多头、残差、LayerNorm、因果掩码会全部用上。

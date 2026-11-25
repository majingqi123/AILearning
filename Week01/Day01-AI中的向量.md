# 程序员的 AI 学习日志

**AI LEARNING · DAY 01**  
**主题：AI 中的向量**  
**学习记录 · 数学基础**

---

## 1. 今日学习目标

今天的核心任务是理解向量，以及向量为什么会成为 AI 中表示和处理信息的重要基础。

完成今天的学习后，应该能够：

1. 区分标量与向量，理解向量的维度和几何意义。
2. 掌握向量的加法、减法、数乘、模、单位向量和归一化。
3. 理解点积、夹角、余弦相似度，以及它们之间的关系。
4. 初步理解向量在 Embedding、语义搜索、RAG、推荐系统和 Transformer 中的用途。
5. 能够用 C++ 实现基础向量运算和余弦相似度计算。

## 2. 标量与向量

### 2.1 标量

标量只有大小，没有方向。例如：

- 温度：25℃
- 质量：80kg
- 时间：10s
- 一个数值：5

### 2.2 向量

向量既有大小，也有方向。

例如：

- 向东移动 5 米
- 物体以每秒 3 米的速度向上运动
- 摄像机朝某个方向观察

二维向量通常写成：

\[
\mathbf{a}=(x,y)
\]

三维向量通常写成：

\[
\mathbf{a}=(x,y,z)
\]

其中每个分量描述向量在对应坐标轴方向上的分量。

### 2.3 向量的起点与终点

如果向量从点 A 指向点 B，那么：

\[
\overrightarrow{AB}=B-A
\]

若 \(A=(1,2)\)，\(B=(4,6)\)，则：

\[
\overrightarrow{AB}=(4-1,6-2)=(3,4)
\]

注意：向量可以平移而不改变其大小和方向，因此在数学运算中，通常更关注向量的分量，而不是它画在坐标系中的具体位置。

## 3. 向量的基本运算

设：

\[
\mathbf{a}=(a_x,a_y),\quad \mathbf{b}=(b_x,b_y)
\]

### 3.1 向量加法

\[
\mathbf{a}+\mathbf{b}=(a_x+b_x,a_y+b_y)
\]

几何意义：将一个向量平移到另一个向量的终点处，得到合向量，也可以用平行四边形法则理解。

应用示例：物体原本有一个移动速度，又受到另一个方向的速度影响，最终速度可以通过向量相加得到。

### 3.2 向量减法

\[
\mathbf{a}-\mathbf{b}=(a_x-b_x,a_y-b_y)
\]

几何意义：如果两个向量起点相同，\(\mathbf{a}-\mathbf{b}\) 表示从 \(\mathbf{b}\) 的终点指向 \(\mathbf{a}\) 的终点。

应用示例：游戏或图形程序中，计算目标相对于摄像机或物体的位置方向：

```cpp
Vector3 direction = targetPosition - cameraPosition;
```

### 3.3 向量数乘

\[
k\mathbf{a}=(ka_x,ka_y)
\]

- \(k>1\)：向量长度变大，方向不变。
- \(0<k<1\)：向量长度变小，方向不变。
- \(k<0\)：方向反转，长度按 \(|k|\) 倍缩放。
- \(k=0\)：得到零向量。

## 4. 向量的模、单位向量与归一化

### 4.1 向量的模

向量的模就是向量的长度。

二维向量：

\[
|\mathbf{a}|=\sqrt{x^2+y^2}
\]

三维向量：

\[
|\mathbf{a}|=\sqrt{x^2+y^2+z^2}
\]

例如：

\[
\mathbf{a}=(3,4)
\]

则：

\[
|\mathbf{a}|=\sqrt{3^2+4^2}=5
\]

### 4.2 单位向量

单位向量的长度为 1，主要用于表示方向。

### 4.3 归一化

将一个非零向量除以它的模，就可以得到对应的单位向量：

\[
\hat{\mathbf{a}}=\frac{\mathbf{a}}{|\mathbf{a}|}
\]

例如：

\[
\mathbf{a}=(3,4),\quad |\mathbf{a}|=5
\]

归一化后：

\[
\hat{\mathbf{a}}=(0.6,0.8)
\]

注意：零向量的模为 0，不能直接归一化，否则会出现除以 0 的问题。工程代码中需要先检查向量长度是否足够大。

### 4.4 在图形学中的应用

- 摄像机朝向通常使用方向向量表示。
- 光照计算中常将法线和光照方向归一化。
- 移动方向归一化后再乘以速度和时间，可以避免斜向移动更快的问题。

## 5. 点积（Dot Product）

点积是向量运算中非常重要的概念，也是理解余弦相似度和 Transformer Attention 的基础之一。

### 5.1 分量形式

对于二维向量：

\[
\mathbf{a}\cdot\mathbf{b}=a_xb_x+a_yb_y
\]

对于三维向量：

\[
\mathbf{a}\cdot\mathbf{b}=a_xb_x+a_yb_y+a_zb_z
\]

### 5.2 几何形式

\[
\mathbf{a}\cdot\mathbf{b}
=|\mathbf{a}||\mathbf{b}|\cos\theta
\]

其中 \(\theta\) 是两个向量之间的夹角。

这个公式说明：点积不仅与向量长度有关，也与两个向量的方向关系有关。

### 5.3 点积的几何意义

当两个向量都不是零向量时：

| 点积结果 | 夹角关系 | 含义 |
|---|---|---|
| 大于 0 | 小于 90° | 方向大致相同 |
| 等于 0 | 等于 90° | 两个向量垂直 |
| 小于 0 | 大于 90° | 方向大致相反 |

如果已知两个向量的点积和模，可以计算夹角：

\[
\theta=\arccos\left(
\frac{\mathbf{a}\cdot\mathbf{b}}
{|\mathbf{a}||\mathbf{b}|}
\right)
\]

计算时需要确保两个向量都不是零向量，并在浮点计算中将余弦值限制在 \([-1,1]\) 范围内，避免数值误差导致反余弦计算异常。

### 5.4 点积在图形学中的应用

**判断目标是否位于前方**

设摄像机前方向量为 `forward`，摄像机到目标的方向为 `toTarget`。归一化后计算：

```cpp
float d = Dot(forward, toTarget);
if (d > 0.0f)
{
    // 目标位于摄像机前方的半空间
}
```

若还需要判断目标是否位于视野角范围内，则需要将点积结果与视野角对应的余弦阈值比较。

**Lambert 漫反射光照**

常见的漫反射强度计算：

\[
I=\max(0,\mathbf{N}\cdot\mathbf{L})
\]

其中：

- \(\mathbf{N}\)：归一化后的表面法线。
- \(\mathbf{L}\)：归一化后的指向光源方向。
- \(I\)：漫反射强度系数。

## 6. 叉积（Cross Product）

叉积主要用于三维向量。

\[
\mathbf{a}\times\mathbf{b}
=
\begin{pmatrix}
a_yb_z-a_zb_y\\
a_zb_x-a_xb_z\\
a_xb_y-a_yb_x
\end{pmatrix}
\]

### 6.1 几何意义

叉积的结果是一个同时垂直于原来两个向量的新向量。

其长度为：

\[
|\mathbf{a}\times\mathbf{b}|
=|\mathbf{a}||\mathbf{b}|\sin\theta
\]

这个长度也等于以两个向量为邻边的平行四边形面积。

### 6.2 方向：右手定则

右手四指从第一个向量的方向弯向第二个向量，拇指所指方向就是叉积结果的方向。

叉积不满足交换律：

\[
\mathbf{a}\times\mathbf{b}
=-(\mathbf{b}\times\mathbf{a})
\]

### 6.3 在图形学中的应用

- 通过三角形的两条边计算三角形法线。
- 构建摄像机坐标系中的右方向、上方向。
- 判断三维空间中的朝向关系。

## 7. 向量投影

向量投影用于计算一个向量在另一个向量方向上的分量。

设向量 \(\mathbf{a}\) 在非零向量 \(\mathbf{b}\) 上的标量投影为：

\[
\operatorname{comp}_{\mathbf{b}}(\mathbf{a})
=\frac{\mathbf{a}\cdot\mathbf{b}}{|\mathbf{b}|}
\]

对应的向量投影为：

\[
\operatorname{proj}_{\mathbf{b}}(\mathbf{a})
=\frac{\mathbf{a}\cdot\mathbf{b}}{|\mathbf{b}|^2}\mathbf{b}
\]

如果 \(\mathbf{b}\) 是单位向量，则公式可以简化为：

\[
\operatorname{proj}_{\mathbf{b}}(\mathbf{a})
=(\mathbf{a}\cdot\mathbf{b})\mathbf{b}
\]

应用示例：将角色速度分解为沿斜坡方向的速度和垂直斜坡方向的速度。

## 8. 线性组合、基与维度

### 8.1 线性组合

若：

\[
\mathbf{v}=a\mathbf{x}+b\mathbf{y}
\]

那么称 \(\mathbf{v}\) 是 \(\mathbf{x}\) 和 \(\mathbf{y}\) 的线性组合。

### 8.2 线性相关与线性无关

如果一组向量中，至少有一个向量可以由其他向量线性组合得到，那么这组向量线性相关。

如果只有所有系数都为 0 时，线性组合才能得到零向量，那么这组向量线性无关。

直观理解：线性相关意味着存在冗余方向；线性无关意味着每个向量都提供了新的独立方向。

### 8.3 基（Basis）

一组向量如果既线性无关，又能张成整个空间，就构成这个空间的一组基。

二维平面的一组常见基为：

\[
\mathbf{e}_1=(1,0),\quad \mathbf{e}_2=(0,1)
\]

任意二维向量都可以表示为：

\[
(x,y)=x(1,0)+y(0,1)
\]

### 8.4 维度（Dimension）

空间的维度可以理解为描述该空间所需的独立基向量数量。

- 一维空间：1 个独立方向。
- 二维空间：2 个独立方向。
- 三维空间：3 个独立方向。
- n 维空间：n 个独立方向。

## 9. 从几何向量到 AI Embedding

前面学习的向量通常只有 2 维或 3 维，便于画图和理解。AI 中的向量往往有几百甚至几千个维度。

例如，一个文本经过 Embedding 模型后，可能得到如下形式的向量：

```text
[0.12, -0.35, 0.87, 0.06, ..., -0.21]
```

这里的每个数字通常不是一个可以直接解释的单独属性，而是模型学习到的特征表示。整个向量共同编码了文本的语义信息。

向量维度取决于具体模型，例如 384、768、1024、1536 或 3072 维都可能出现；不能认为所有 Embedding 模型的维度都相同。

### 9.1 为什么 AI 需要向量

计算机擅长处理数字，而自然语言、图片等原始信息并不天然适合直接进行数学运算。

Embedding 模型可以把文本、图片等信息映射到向量空间中。这样就可以利用向量运算比较信息之间的关系。

例如：

- “我喜欢苹果”
- “我爱吃苹果”
- “今天的天气很好”

前两句话语义接近，Embedding 模型通常会让它们在向量空间中表现出较高的相似度；第三句话与前两句的语义关系通常较弱。

注意：向量不是简单地把每个词映射成固定含义的数字。Embedding 的语义表示是模型从大量数据中学习得到的。

## 10. 余弦相似度（Cosine Similarity）

余弦相似度是比较两个向量方向接近程度的常见方法。

### 10.1 公式

\[
\operatorname{sim}(\mathbf{A},\mathbf{B})
=\frac{\mathbf{A}\cdot\mathbf{B}}
{|\mathbf{A}||\mathbf{B}|}
=\cos\theta
\]

其中 \(\theta\) 是两个向量的夹角。两个向量都不能是零向量。

余弦相似度的理论取值范围为：

\[
[-1,1]
\]

| 结果 | 含义 |
|---|---|
| 接近 1 | 方向接近，通常表示相似 |
| 接近 0 | 方向近似垂直 |
| 接近 -1 | 方向相反 |

实际 Embedding 模型的数据分布与训练目标有关，实际检索结果不一定覆盖整个区间，也不能简单假设所有文本相似度都在 0 到 1 之间。

### 10.2 计算示例

设：

\[
\mathbf{A}=(1,2),\quad \mathbf{B}=(2,1)
\]

点积：

\[
\mathbf{A}\cdot\mathbf{B}=1\times2+2\times1=4
\]

模：

\[
|\mathbf{A}|=\sqrt{1^2+2^2}=\sqrt{5}
\]

\[
|\mathbf{B}|=\sqrt{2^2+1^2}=\sqrt{5}
\]

因此：

\[
\operatorname{sim}(\mathbf{A},\mathbf{B})
=\frac{4}{\sqrt{5}\sqrt{5}}
=\frac{4}{5}=0.8
\]

这表示两个向量的方向比较接近。

### 10.3 点积与余弦相似度的区别

| 对比 | 点积 | 余弦相似度 |
|---|---|---|
| 计算 | \(\mathbf{A}\cdot\mathbf{B}\) | \(\frac{\mathbf{A}\cdot\mathbf{B}}{|\mathbf{A}||\mathbf{B}|}\) |
| 是否受向量长度影响 | 是 | 归一化后不受长度影响 |
| 主要衡量 | 长度与方向共同影响的匹配程度 | 方向接近程度 |
| 常见用途 | 几何计算、神经网络计算 | 语义相似度、向量检索 |

如果两个向量已经归一化为单位向量，那么它们的点积就等于余弦相似度。

## 11. 向量在 AI 中的典型应用

### 11.1 语义搜索（Semantic Search）

传统关键词搜索主要依赖词语匹配。例如搜索“汽车维修”，可能难以找到只写了“车辆保养”的内容。

语义搜索会将查询和文档转换为向量，再根据向量相似度检索语义接近的内容。

基本流程：

1. 使用 Embedding 模型把文档转换为向量。
2. 将向量和文档信息保存起来。
3. 把用户查询转换为向量。
4. 计算查询向量与文档向量之间的相似度。
5. 返回最相关的文档。

### 11.2 RAG（检索增强生成）

RAG 是向量应用中非常重要的场景。

典型流程：

```text
知识文档
   |
   v
文档切分（Chunk）
   |
   v
Embedding 模型
   |
   v
文档向量 + 原文存储
   |
   |                         用户问题
   |                            |
   |                            v
   |                       Embedding 模型
   |                            |
   |                            v
   +--------------------> 向量检索
                                |
                                v
                         相关内容 Top-K
                                |
                                v
                         拼接到 Prompt
                                |
                                v
                              LLM
                                |
                                v
                              回答
```

实际 RAG 系统还可能结合关键词检索、元数据过滤和 Reranker（重排序模型），不一定只依赖向量相似度。

### 11.3 推荐系统

用户和商品都可以表示为向量。通过比较用户向量与商品向量，可以估计用户对商品的兴趣程度，并用于候选商品召回或排序。

### 11.4 多模态检索

例如 CLIP 一类模型可以把图片和文本映射到可比较的向量空间中，从而实现：

- 输入一段文字，搜索匹配的图片。
- 输入一张图片，搜索语义接近的图片或文本。

### 11.5 聚类与分类

向量可以作为机器学习算法的输入：

- 聚类：把相似向量自动分组。
- 分类：根据向量特征判断样本所属类别。

## 12. Transformer 中的点积

Transformer 的 Attention 机制也使用向量点积。

Scaled Dot-Product Attention 的公式为：

\[
\operatorname{Attention}(Q,K,V)
=
\operatorname{softmax}
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)V
\]

其中：

- \(Q\)：Query，查询向量。
- \(K\)：Key，键向量。
- \(V\)：Value，值向量。
- \(d_k\)：Key 向量的维度。

直观理解：

1. Query 与各个 Key 做点积。
2. 点积结果经过缩放和 Softmax，得到注意力权重。
3. 使用权重对 Value 加权求和，得到输出。

这说明点积是 Transformer 中衡量 Query 与 Key 匹配程度的重要计算。

需要特别区分：Attention 中的缩放点积不等同于余弦相似度。Q 和 K 通常不要求预先归一化，因此点积仍会受到向量长度影响。

## 13. C++ 实践：实现向量与余弦相似度

对于 C++ 开发者，可以先不依赖第三方库，手动实现基础运算。

### 13.1 Vector3 示例

```cpp
#include <cmath>

struct Vector3
{
    float x = 0.0f;
    float y = 0.0f;
    float z = 0.0f;

    Vector3 operator+(const Vector3& rhs) const
    {
        return { x + rhs.x, y + rhs.y, z + rhs.z };
    }

    Vector3 operator-(const Vector3& rhs) const
    {
        return { x - rhs.x, y - rhs.y, z - rhs.z };
    }

    Vector3 operator*(float k) const
    {
        return { x * k, y * k, z * k };
    }

    float Dot(const Vector3& rhs) const
    {
        return x * rhs.x + y * rhs.y + z * rhs.z;
    }

    float Length() const
    {
        return std::sqrt(Dot(*this));
    }

    Vector3 Normalized() const
    {
        const float len = Length();
        if (len < 1e-6f)
            return { 0.0f, 0.0f, 0.0f };

        return { x / len, y / len, z / len };
    }
};
```

### 13.2 计算余弦相似度

```cpp
#include <cmath>
#include <vector>
#include <stdexcept>

float CosineSimilarity(
    const std::vector<float>& a,
    const std::vector<float>& b)
{
    if (a.size() != b.size() || a.empty())
        throw std::invalid_argument("Vector dimensions do not match");

    float dot = 0.0f;
    float normA = 0.0f;
    float normB = 0.0f;

    for (size_t i = 0; i < a.size(); ++i)
    {
        dot += a[i] * b[i];
        normA += a[i] * a[i];
        normB += b[i] * b[i];
    }

    if (normA < 1e-12f || normB < 1e-12f)
        throw std::invalid_argument("Cosine similarity is undefined for zero vectors");

    return dot / (std::sqrt(normA) * std::sqrt(normB));
}
```

实际工程中还应考虑浮点误差、异常输入、性能以及是否提前对向量归一化。

## 14. 向量数据库与检索工具

当向量数量变大时，逐个计算相似度会越来越慢，因此需要专门的向量检索工具。

| 工具 | 定位 | 适合了解的方向 |
|---|---|---|
| FAISS | 向量相似度搜索库 | 本地检索、索引算法、C++ 集成 |
| Milvus | 向量数据库 | 大规模向量存储与检索 |
| Qdrant | 向量数据库 | 向量检索、过滤与服务化 |
| Chroma | 面向 Embedding 应用的向量数据库 | 快速搭建 RAG 原型 |

对于 C++ 开发者，FAISS 值得优先了解，因为它提供 C++ 接口，适合从底层检索算法和工程集成入手。

参考资料：

- FAISS：<https://github.com/facebookresearch/faiss>
- Milvus：<https://milvus.io/>
- Qdrant：<https://qdrant.tech/>
- Chroma：<https://www.trychroma.com/>

## 15. 今日练习与验收

### 15.1 基础练习

1. 已知 \(\mathbf{a}=(3,4)\)，计算它的模和单位向量。
2. 已知 \(\mathbf{a}=(1,2)\)、\(\mathbf{b}=(3,4)\)，计算 \(\mathbf{a}+\mathbf{b}\)、\(\mathbf{a}-\mathbf{b}\) 和点积。
3. 两个非零向量的点积为 0，说明它们有什么关系？
4. 余弦相似度为 1、0、-1 时，分别表示什么？
5. 为什么余弦相似度比普通点积更适合衡量某些文本向量的方向相似程度？
6. RAG 为什么需要把文档和用户问题转换为向量？
7. Transformer Attention 中的 Q 和 K 为什么要做点积？

### 15.2 编程练习

建议使用 C++ 完成：

- 实现二维或三维向量结构。
- 实现向量加法、减法、数乘。
- 实现 Dot、Length、Normalize。
- 实现 CosineSimilarity。
- 构造若干测试向量，验证相似度计算结果。
- 尝试实现 Top-K：给定一个查询向量和一组候选向量，找出相似度最高的 K 个。

### 15.3 今日验收标准

完成以下内容，就算掌握了今天的核心知识：

- [ ] 能够解释标量和向量的区别。
- [ ] 能够计算向量的模、归一化和点积。
- [ ] 能够解释点积与夹角的关系。
- [ ] 能够说清楚余弦相似度的公式和含义。
- [ ] 能够解释 Embedding 如何把文本转换为向量。
- [ ] 能够描述向量检索在 RAG 中的作用。
- [ ] 能够用 C++ 实现余弦相似度。

## 16. 今日总结

今天从线性代数中的向量出发，学习了向量的基本运算、模、归一化、点积、叉积和投影，并进一步理解了余弦相似度。

最重要的认知是：

**AI 中的向量并不只是数学课本中的坐标，它还是模型表示文本、图片等信息的一种方式。**

Embedding 模型负责把信息转换为向量；向量检索通过相似度计算找到相关内容；RAG 将检索到的内容提供给大语言模型，帮助模型基于外部知识回答问题。

对于 C++ 工程师来说，理解向量运算不仅能衔接已有的图形学知识，也能进一步理解 Embedding、向量数据库和 RAG 的底层工作方式。

---

## 17. 学习记录

- **学习日期：** 
- **实际学习时长：** 
- **完成情况：** 
- **仍然不理解的知识点：** 
- **明天需要复习的内容：** 
- **个人总结：** 


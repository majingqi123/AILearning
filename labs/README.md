# 实验目录约定

本目录放 [AI 学习计划（23 周）](../docs/AI学习计划-23周.md) 的全部实验代码，一周一个子目录。

## 命名

- 子目录用 `w两位周次-主题`，如 `w01-topk`、`w08-gpt`。
- 跨周的实验沿用第一次创建时的目录，不重复建。
- 每个要对外展示的作品在自己的目录里放一份 `README.md`，写清楚怎么运行。

## 目录一览

| 目录 | 周次 | 内容 | 语言 |
|---|---|---|---|
| `w01-topk` | W1-W2 | 向量 Top-K 检索命令行工具 | C++ |
| `w03-math` | W3 | 梯度下降、线性回归 | Python |
| `w04-ml` | W4 | scikit-learn 分类、聚类、降维 | Python |
| `w05-nn` | W5 | 自动求导、手写 MLP | Python |
| `w06-cnn` | W6 | MNIST 上的 MLP 与 CNN | Python |
| `w08-gpt` | W8-W9 | 字符级小 GPT、采样参数、KV Cache | Python |
| `w09-llm` | W9 | 简化 BPE | Python |
| `w10-hf` | W10 | 开源模型加载、性能测量、QLoRA 微调 | Python |
| `w11-deploy` | W11 | llama.cpp 与 ONNX Runtime 的 C++ 调用 | C++ |
| `w12-api` | W12 | 流式聊天与工具调用 | Python、C++ |
| `w13-rag` | W13 | 本地文档问答 | Python |
| `w14-code-rag` | W14 | C++ 代码库问答 | Python |
| `w15-agent` | W15 | 最小编码 Agent | Python |
| `w16-mcp` | W16 | 代码检索 MCP Server | Python |
| `w17-framework` | W17 | LangGraph 版 Agent | Python |
| `w18-service` | W18 | RAG HTTP 服务、评测与成本统计 | Python |
| `w19-multimodal` | W19 | CLIP、Whisper、扩散模型 | Python |
| `w22-capstone` | W22-W23 | 知识库 + 代码生成 Agent | Python、C++ |

## 注意

- 模型权重、数据集、向量索引、构建产物不要提交到 git，用 `.gitignore` 排除。
- API Key 放环境变量或本地配置文件里，不要写进代码。
- Python 实验共用工作区根目录下的 `.venv`；某个包在当前 Python 版本下装不上时，换一个 Python 版本另建环境，并在该实验的 README 里注明。
- 换电脑时 `.venv` 不用迁移，在新电脑上重建即可。

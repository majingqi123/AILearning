# W06 周计划：PyTorch 训练实践与 CNN

- **所属阶段**：阶段三 深度学习（W5-W8）
- **起止日期**：
- **计划用时**：工作日 5 × 约 1.5 小时 + 周末约 3 小时，共约 10 小时
- **硬件要求**：有 NVIDIA 显卡更快，不是必需；没有就用 CPU 训练，MNIST 也能跑完
- **主计划**：[AI 学习计划（23 周）](../AI学习计划-23周.md)
- **上一周**：[W05](W05-神经网络与反向传播.md)
- **笔记位置**：`notes/W06/`，每天两个文件：学习笔记用 [学习笔记模板](../../notes/学习笔记模板.md)，自我总结用 [自我总结模板](../../notes/自我总结模板.md)

## 1. 本周目标

学完这一周，你应该能做到：

1. 用 `nn.Module`、`Dataset`、`DataLoader` 写出规范的训练和验证循环。
2. 在 MNIST 上训练 MLP 和 CNN，会保存、加载权重。
3. 讲清卷积、池化、padding、stride、通道和感受野。
4. 理解 BatchNorm、Dropout 的作用，并能用实验对比。
5. 讲清 ResNet 残差连接为什么有效，会用预训练 ResNet 做图片分类。

## 2. 本周产出

| 产出 | 位置 | 对应日 |
|---|---|---|
| 训练循环模板 | `labs/w06-cnn/01-train-loop.ipynb` | 周一 |
| MNIST MLP 准确率与权重文件 | `labs/w06-cnn/02-mnist-mlp.ipynb`、`checkpoints/` | 周二 |
| 卷积特征图可视化 | `labs/w06-cnn/03-conv-viz.ipynb` | 周三 |
| MNIST CNN，含 BatchNorm / Dropout 对比 | `labs/w06-cnn/04-mnist-cnn.ipynb` | 周四 |
| CNN 演进与残差笔记 | `notes/W06/` | 周五 |
| 预训练 ResNet 分类实验 | `labs/w06-cnn/05-resnet-ft.ipynb` | 周末 |

## 3. 开始前的准备

- [ ] 新建目录 `notes/W06/` 和 `labs/w06-cnn/`。
- [ ] 确认 W5 装好的 PyTorch 还能用：`python -c "import torch; print(torch.__version__, torch.cuda.is_available())"`。
- [ ] 第一次跑 MNIST 会下载数据，提前确认网络能访问；不行就手动下载到 `~/.pytorch` 或改 `torchvision.datasets.MNIST` 的路径。

---

## 4. 每日安排

### 周一：nn.Module、DataLoader 与训练循环

**学习目标**

1. 会用 `nn.Module` 定义网络，知道 `forward`、`parameters`、`state_dict` 是什么。
2. 会用 `Dataset` 和 `DataLoader` 按 batch 取数据。
3. 写出“前向 → 损失 → 反向 → 更新 → 清零梯度”的标准训练循环。

**时间安排**：读资料 40 分钟，动手 30 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [PyTorch Basics：构建模型](https://docs.pytorch.org/tutorials/beginner/basics/buildmodel_tutorial.html) | 英文 | 必读 | `nn.Module`、`nn.Linear`、`Sequential` |
| [PyTorch Basics：数据](https://docs.pytorch.org/tutorials/beginner/basics/data_tutorial.html) | 英文 | 必读 | `Dataset`、`DataLoader` |
| [PyTorch Basics：优化](https://docs.pytorch.org/tutorials/beginner/basics/optimization_tutorial.html) | 英文 | 必读 | 损失、优化器、完整训练循环 |
| [动手学深度学习：深度学习计算](https://zh.d2l.ai/chapter_deep-learning-computation/index.html) | 中文 | 选读 | “层和块”“参数管理”两节 |

**动手任务**

1. 用 `nn.Module` 写一个两层 MLP（输入 784、隐藏 128、输出 10），打印 `model` 和参数量。
2. 用 `torchvision.datasets.MNIST` 建训练集和测试集，包一层 `DataLoader`（`batch_size=64`，训练集 `shuffle=True`）。
3. 写一个不训练、只跑一个 epoch 前向的循环，打印每个 batch 的输入形状和输出形状。
4. 整理成一个可复用的训练循环模板：`train_one_epoch`、`evaluate`、主循环。先不追求准确率，保证能跑通。

**和已有知识的联系**：W5 周末的 PyTorch 版是手写张量；这一周改用 `nn.Module`，等价于把参数管理和前向计算封装成类，和你在引擎里封装渲染对象的思路一样。

**验收标准**

- [ ] 能打印出模型结构和参数量。
- [ ] DataLoader 能正确产出 `(B, 1, 28, 28)` 和标签。
- [ ] 有一份可复用的训练循环模板。

**自测题**

1. `nn.Module` 的 `forward` 和直接调用 `model(x)` 有什么关系？
2. `DataLoader` 的 `shuffle`、`batch_size`、`num_workers` 各影响什么？
3. 训练循环里 `optimizer.zero_grad()`、`loss.backward()`、`optimizer.step()` 的顺序为什么是这样？

---

### 周二：MNIST 上训练 MLP

**学习目标**

1. 在真实数据集上完整训练一个分类模型。
2. 区分 `train()` 和 `eval()`，会做验证集评估。
3. 会保存和加载权重。

**时间安排**：读资料 20 分钟，动手 50 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [PyTorch Basics：保存与加载](https://docs.pytorch.org/tutorials/beginner/basics/saveloadrun_tutorial.html) | 英文 | 必读 | `state_dict`、`torch.save`、`torch.load` |
| [动手学深度学习：使用 GPU](https://zh.d2l.ai/chapter_deep-learning-computation/use-gpu.html) | 中文 | 必读 | 设备和张量/模型的 `.to(device)` |
| [动手学深度学习：读写文件](https://zh.d2l.ai/chapter_deep-learning-computation/read-write.html) | 中文 | 选读 | 参数保存的细节 |

**动手任务**

1. 把周一的 MLP 搬到 GPU（有的话）：`device = "cuda" if torch.cuda.is_available() else "cpu"`，模型和数据都 `.to(device)`。
2. 训练 5–10 个 epoch：优化器用 `Adam`，损失用 `CrossEntropyLoss`，学习率 `1e-3`。每个 epoch 记录训练损失和测试准确率。
3. 画损失曲线和准确率曲线。
4. 用 `torch.save(model.state_dict(), ...)` 保存最好的权重，新开一个进程加载，确认测试准确率一致。
5. 故意在 `eval()` 模式下漏掉 `torch.no_grad()`，观察显存或速度有没有变化（有 Dropout/BN 时差别更明显，周四再体会）。

**验收标准**

- [ ] 测试准确率达到 95% 以上。
- [ ] 有训练曲线图。
- [ ] 能成功保存并加载权重，准确率一致。

**自测题**

1. `model.train()` 和 `model.eval()` 分别影响哪些层？
2. 为什么验证时要用 `torch.no_grad()`？
3. 保存整个模型对象和只保存 `state_dict` 各有什么优缺点？

---

### 周三：卷积、池化与特征图

**学习目标**

1. 理解卷积核、padding、stride、通道、感受野。
2. 会算卷积输出的空间尺寸。
3. 能把卷积和图形学里的图像滤波对应起来。

**时间安排**：读资料 40 分钟，动手 30 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：卷积神经网络](https://zh.d2l.ai/chapter_convolutional-neural-networks/index.html) | 中文 | 必读 | 卷积层、填充和步幅、多输入多输出通道、汇聚层 |
| [CNN Explainer](https://poloclub.github.io/cnn-explainer/) | 英文界面 | 必读 | 交互式看卷积怎么滑动，不用读很多文字 |
| [动手学深度学习：填充和步幅](https://zh.d2l.ai/chapter_convolutional-neural-networks/padding-and-strides.html) | 中文 | 必读 | 输出尺寸公式 |
| [动手学深度学习：通道](https://zh.d2l.ai/chapter_convolutional-neural-networks/channels.html) | 中文 | 必读 | 多通道卷积 |

**动手任务**

1. 手算：输入 `28×28`，`3×3` 卷积、`padding=1`、`stride=1`，输出空间尺寸是多少？`stride=2` 呢？
2. 用 `nn.Conv2d(1, 8, kernel_size=3, padding=1)` 对一张 MNIST 图片做卷积，把 8 个输出通道画成特征图。
3. 再接一个 `MaxPool2d(2)`，观察特征图尺寸怎么变。
4. 用一张自己的照片（转灰度或用 RGB）试 `Conv2d`，感受“滤波器”和图形学里模糊、锐化、边缘检测核的相似性。

**和已有知识的联系**：卷积核就是一组可学习的图像滤波器。传统图像处理里核是人手设计的；CNN 里核是训练出来的。多通道卷积可以想成对多个纹理/特征图同时滤波再线性组合。

**验收标准**

- [ ] 输出尺寸公式能手算，和 `nn.Conv2d` 实际输出一致。
- [ ] 有特征图可视化。
- [ ] 能用一句话解释 padding 和 stride 各自解决什么问题。

**自测题**

1. 卷积输出空间尺寸的公式是什么？
2. 输入通道 3、输出通道 16、`kernel_size=3` 的卷积层有多少个参数（含偏置）？
3. 池化层为什么几乎没有可学习参数？它的作用是什么？
4. 感受野是什么？堆更多层或加大 stride 对感受野有什么影响？

---

### 周四：MNIST 上训练 CNN，对比 BatchNorm 与 Dropout

**学习目标**

1. 搭一个简单 CNN 并在 MNIST 上训到高准确率。
2. 理解 BatchNorm 和 Dropout 各自解决什么问题。
3. 用对照实验看出它们对训练曲线的影响。

**时间安排**：读资料 25 分钟，动手 45 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：BatchNorm](https://zh.d2l.ai/chapter_convolutional-modern/batch-norm.html) | 中文 | 必读 | 原理和代码 |
| [动手学深度学习：Dropout](https://zh.d2l.ai/chapter_multilayer-perceptrons/dropout.html) | 中文 | 必读 | 回顾即可，W4/W5 接触过 |
| [动手学深度学习：LeNet](https://zh.d2l.ai/chapter_convolutional-neural-networks/lenet.html) | 中文 | 必读 | 经典小 CNN 结构，可直接当参考 |

**动手任务**

1. 搭一个小 CNN：两到三个 `Conv-ReLU-Pool` 块，再接全连接分类头。参考 LeNet，不必一模一样。
2. 训练到测试准确率 ≥ 98%。记录训练时间、最终准确率。
3. 做三组对照（其他超参尽量不变）：
   - 基线 CNN
   - 基线 + BatchNorm（卷积后、激活前）
   - 基线 + Dropout（全连接前，`p=0.5`）
4. 画三组的训练/测试曲线，写三句话结论：谁收敛更快、谁过拟合更轻、谁最终准确率更高。

**验收标准**

- [ ] CNN 测试准确率 ≥ 98%。
- [ ] 有三组对照的曲线和结论。
- [ ] 能说明 `train()` / `eval()` 对 BatchNorm 和 Dropout 的影响。

**自测题**

1. BatchNorm 在训练和推理时行为有什么不同？
2. Dropout 为什么能减轻过拟合？推理时要怎么处理？
3. 为什么卷积层后面常接 BatchNorm，而不是只在全连接层用？

---

### 周五：从 LeNet 到 ResNet，残差连接

**学习目标**

1. 知道 LeNet、AlexNet、VGG、ResNet 各自解决了什么问题。
2. 理解梯度消失/爆炸，以及残差连接为什么能训练更深的网络。
3. 会读懂一个 ResNet 基本块的结构。

**时间安排**：读资料 50 分钟，动手 20 分钟，笔记和自测 20 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [动手学深度学习：现代卷积神经网络](https://zh.d2l.ai/chapter_convolutional-modern/index.html) | 中文 | 必读 | AlexNet、VGG、ResNet 几节；NiN/GoogLeNet 可略读 |
| [动手学深度学习：残差网络](https://zh.d2l.ai/chapter_convolutional-modern/resnet.html) | 中文 | 必读 | 残差块、为什么有效 |
| [动手学深度学习：数值稳定性和模型初始化](https://zh.d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html) | 中文 | 选读 | 和梯度消失/爆炸相关的部分 |
| [跟李沐学 AI：ResNet 论文精读](https://space.bilibili.com/1567748478) | 中文 | 选读 | 在空间里搜 “ResNet”，时间不够就看上集 |

**动手任务**

1. 画一张时间线：LeNet → AlexNet → VGG → ResNet，每项写一句话“解决了什么问题”。
2. 用 PyTorch 手写一个残差基本块：`Conv-BN-ReLU-Conv-BN`，再与输入相加，最后 ReLU。输入输出通道相同、stride=1 的情况即可。
3. 打印 `torchvision.models.resnet18(weights=None)` 的结构，找到几个残差块，对照你写的基本块。
4. 在笔记里用自己的话解释：如果没有残差，网络变深时梯度会发生什么；有了残差，反向传播多了一条什么路径。

**验收标准**

- [ ] 有演进时间线笔记。
- [ ] 自己写的残差块能前向跑通，输出形状与输入相同。
- [ ] 能不看资料讲清残差连接的作用。

**自测题**

1. VGG 相比 AlexNet 的主要设计思想是什么？
2. 残差连接里的“捷径”在反向传播时起了什么作用？
3. 当残差块要改变通道数或空间尺寸时，捷径上通常要加什么？

---

### 周末：预训练 ResNet 图片分类与阶段整理

**学习目标**

1. 会加载预训练权重，做迁移学习或特征提取。
2. 整理本周 MNIST 实验，形成可展示的结果。

**时间安排**（约 3 小时）：ResNet 实验 90 分钟，整理实验 40 分钟，补笔记 20 分钟，周复盘 30 分钟。

**学习资源**

| 资料 | 语言 | 要求 | 看什么 |
|---|---|---|---|
| [torchvision models](https://docs.pytorch.org/vision/stable/models.html) | 英文 | 必读 | 如何加载预训练权重、预处理 |
| [动手学深度学习：微调](https://zh.d2l.ai/chapter_computer-vision/fine-tuning.html) | 中文 | 必读 | 冻结骨干、替换分类头 |
| [CIFAR10 教程（结构参考）](https://docs.pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html) | 英文 | 选读 | 数据增强和训练循环写法 |

**动手任务**

1. 准备 20–50 张自己的图片，分 2–3 类（例如桌面物品、户外、截图），或用 CIFAR-10 的一个子集。
2. 加载 `resnet18` 预训练权重，替换最后的全连接层为你的类别数。
3. 先冻结骨干只训分类头，再（可选）解冻后几层做微调。记录两种做法的准确率。
4. 整理本周结果到 `labs/w06-cnn/README.md`：MLP 准确率、CNN 准确率、三组对照结论、ResNet 实验结论。
5. 完成周复盘，更新 [进度追踪](../进度追踪.md)。

**验收标准**

- [ ] 预训练 ResNet 在你的数据上能分类，有准确率数字。
- [ ] `labs/w06-cnn/README.md` 写清了怎么复现。
- [ ] 本周验收清单全部打勾。

---

## 5. 本周验收

对应主计划 W6 的验收：

- [ ] 不看教程能写出完整的训练和验证循环。
- [ ] MNIST 上 CNN 测试准确率达到 98% 以上。

## 6. 周复盘

- **实际用时**：
- **验收通过情况**：
- **本周最重要的三个收获**：
- **哪些自测题答不上来**：
- **遗留问题**（带到下周）：
- **哪种学习方式效果好，哪种不好**：
- **下周要调整的地方**：

## 7. 容易踩的坑

- **数据和模型不在同一设备**：报错 `Expected all tensors to be on the same device`。记住 `inputs.to(device)` 和 `model.to(device)`。
- **忘记 `model.eval()`**：验证准确率异常偏低，常见原因是 Dropout 还开着。
- **只看训练准确率**：训练很高、测试一般，就是过拟合；周四的对照就是为这个准备的。
- **一上来就微调全部 ResNet**：数据很少时先冻结骨干，只训分类头更稳。

## 8. 下周预告

[W07](W07-序列建模注意力与Transformer.md) 进入序列建模：RNN/LSTM 的思路、注意力机制、Transformer 结构，以及 BERT / GPT / T5 的区别。这是后面 LLM 阶段的直接前置。

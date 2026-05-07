---
date: "2026-05-07"
paper_id: "arXiv:2307.06304"
title: "Patch n' Pack: NaViT, a Vision Transformer for any Aspect Ratio and Resolution"
authors: "Mostafa Dehghani, Basil Mustafa, Josip Djolonga, Jonathan Heek, Matthias Minderer, Mathilde Caron, Andreas Steiner, Joan Puigcerver, Robert Geirhos, Ibrahim Alabdulmohsin, Avital Oliver, Piotr Padlewski, Alexey Gritsenko, Mario Luci, Neil Houlsby"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - NaViT
  - Vision-Transformer
  - Variable-Resolution
  - Sequence-Packing
  - Aspect-Ratio
  - Efficient-Training
quality_score: "8.5/10"
created: "2026-05-07"
updated: "2026-05-07"
status: analyzed
---

# Patch n' Pack: NaViT, a Vision Transformer for any Aspect Ratio and Resolution

## 核心信息
- **论文ID**：arXiv:2307.06304
- **作者**：Mostafa Dehghani, Basil Mustafa, Josip Djolonga, Jonathan Heek, Matthias Minderer, Mathilde Caron, Andreas Steiner, Joan Puigcerver, Robert Geirhos, Ibrahim Alabdulmohsin, Avital Oliver, Piotr Padlewski, Alexey Gritsenko, Mario Luci, Neil Houlsby
- **机构**：Google DeepMind
- **发布时间**：2023-07-12
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2307.06304) | [PDF](https://arxiv.org/pdf/2307.06304)
- **项目**：Scenic (JAX/FLAX)

## 摘要翻译

### 英文摘要
The ubiquitous and demonstrably suboptimal choice of resizing images to a fixed resolution before processing them with computer vision models has not yet been successfully challenged. However, models such as the Vision Transformer (ViT) offer flexible sequence-based modeling, and hence varying input sequence lengths. We take advantage of this with NaViT (Native Resolution ViT) which uses sequence packing during training to process inputs of arbitrary resolutions and aspect ratios. Alongside flexible model usage, we demonstrate improved training efficiency for large-scale supervised and contrastive image-text pretraining. NaViT can be efficiently transferred to standard tasks such as image and video classification, object detection, and semantic segmentation and leads to improved results on robustness and fairness benchmarks. At inference time, the input resolution flexibility can be used to smoothly navigate the test-time cost-performance trade-off. We believe that NaViT marks a departure from the standard, CNN-designed, input and modelling pipeline used by most computer vision models, and represents a promising direction for ViTs.

### 中文翻译
在计算机视觉模型处理图像之前，将图像调整到固定分辨率这一普遍但明显次优的做法尚未被成功挑战。然而，视觉 Transformer（ViT）等模型提供了灵活的基于序列的建模方式，因此支持可变的输入序列长度。我们利用这一点提出了 NaViT（原生分辨率 ViT），它在训练期间使用序列打包来处理任意分辨率和宽高比的输入。除了灵活的模型使用外，我们还展示了大规模监督和对比式图像-文本预训练的训练效率提升。NaViT 可以高效地迁移到标准任务，如图像和视频分类、目标检测和语义分割，并在鲁棒性和公平性基准上取得改进的结果。在推理时，输入分辨率的灵活性可以用于平滑地权衡测试时的成本-性能。我们相信 NaViT 标志着从大多数计算机视觉模型使用的标准 CNN 设计的输入和建模流水线的转变，代表了 ViT 的一个有前途的方向。

### 核心要点提炼
- **研究背景**：传统 CV 模型将图像 resize 到固定分辨率，这会损失信息且次优
- **核心方法**：Patch n' Pack - 将多个不同分辨率的图像打包到一个序列中训练
- **主要结果**：用 1/4 计算量匹配 ViT 性能，或相同计算量下处理 5 倍更多图像
- **研究意义**：打破了固定分辨率的范式，为 ViT 开辟了新方向

## 研究背景与动机

### 领域现状
视觉 Transformer（ViT）已成为计算机视觉的主流架构，但其输入处理方式仍沿用 CNN 时代的做法：
1. **固定分辨率**：所有图像被 resize 到固定的正方形分辨率（如 224x224）
2. **信息损失**：resize 会扭曲图像的原始宽高比，丢失细节信息
3. **效率问题**：固定批次形状限制了训练效率

### 现有方法的局限性
- **ViT**：固定分辨率，无法处理不同宽高比的图像
- **FlexiViT**：支持多种 patch 大小，但仍使用固定分辨率
- **Pix2Struct**：保留宽高比，但需要学习 2D 位置嵌入，泛化能力有限

### 研究动机
1. **大多数图像不是正方形**：ImageNet 中 85.9%、LVIS 中 92.2%、WebLI 中 57.3% 的图像不是正方形
2. **固定分辨率次优**：resize 会损失信息，padding 效率低下
3. **NLP 的启示**：语言模型通过 example packing 处理可变长度序列，提高训练效率

## 研究问题

1. 如何让 ViT 处理任意分辨率和宽高比的图像？
2. 如何在保持宽高比的同时提高训练效率？
3. 如何在推理时灵活权衡计算成本和性能？

## 方法概述

### 核心思想：Patch n' Pack

**核心创新**：将 NLP 中的 example packing 技术应用到视觉 Transformer，将多个不同分辨率的图像打包到一个固定长度的序列中。

$$\text{Sequence} = [\text{Image}_1\text{ patches}, \text{Padding}, \text{Image}_2\text{ patches}, ...]$$

### 架构修改

#### 1. 掩码自注意力和掩码池化

为了防止不同图像之间的注意力交互，引入额外的自注意力掩码：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}} + M\right)V$$

其中 $M$ 是掩码矩阵，确保每个 token 只能关注同一图像内的其他 token。

类似地，掩码池化确保每个图像的表示只从其自身的 token 池化得到。

#### 2. 分解和分数位置嵌入

**问题**：传统 ViT 的 1D 位置嵌入无法处理不同分辨率和宽高比。

**解决方案**：分解位置嵌入，将 x 和 y 坐标的嵌入分开学习然后相加：

$$\text{PE}(x, y) = \text{PE}_x(x) + \text{PE}_y(y)$$

**两种模式**：
- **绝对嵌入**：$\phi(p): [0, \text{maxLen}] \rightarrow \mathbb{R}^D$，基于绝对 patch 索引
- **分数嵌入**：$\phi(r): [0, 1] \rightarrow \mathbb{R}^D$，基于相对距离 $r = p/\text{side-length}$

**嵌入类型**：
- 学习嵌入
- 正弦嵌入
- NeRF 使用的学习傅里叶位置嵌入

**优势**：分解嵌入在未见过的分辨率上泛化更好，而 Pix2Struct 的 2D 学习嵌入在高分辨率上泛化困难。

### 训练改进

#### 1. 连续 Token 丢弃

**传统方法**：所有图像使用相同的 token 丢弃比例

**NaViT 创新**：每个图像可以有不同的 token 丢弃比例

**优势**：
- 可以在训练过程中动态调整丢弃比例
- 可以设计丢弃比例的时间调度策略
- 平衡吞吐量和信息保留

**实验结果**：
- Beta 分布采样的丢弃比例优于固定比例
- 分辨率相关的丢弃比例进一步提升性能
- 递减调度策略（开始高丢弃，逐渐降低）效果最佳

#### 2. 分辨率采样

**核心思想**：在训练时从分布中采样分辨率，保持原始宽高比

**采样策略**：
- **固定分辨率**：$R = R_{\text{max}}$
- **可变分辨率**：$R \sim U(64, R_{\text{max}})$

**采样分布**：
- 均匀分布
- 截断正态分布（偏向低分辨率）
- 边长采样 vs 面积采样

**最佳策略**：直接采样边长，使用偏向低分辨率的截断正态分布

**优势**：
- 同时获得高吞吐量（小图像）和高质量（大图像）
- 可变分辨率训练优于固定分辨率

### 效率分析

#### 自注意力开销

虽然打包会增加序列长度，但随着模型维度增加，注意力开销占比减小：

| 模型维度 | 打包 1 张 | 打包 2 张 | 打包 4 张 | 打包 8 张 |
|----------|-----------|-----------|-----------|-----------|
| 1024 | 10% | 15% | 20% | 25% |
| 2048 | 8% | 12% | 16% | 20% |
| 4096 | 5% | 8% | 10% | 13% |
| 6144 | 3% | 5% | 7% | 9% |

**结论**：对于大模型，打包带来的额外注意力开销可以忽略不计。

#### 对比学习损失处理

**问题**：对比学习需要提取多个池化表示，固定批次形状限制了每序列的最大图像数

**解决方案**：使用分块对比损失（chunked contrastive loss），在局部设备子集上计算，高效累积全局 softmax 归一化的统计信息

## 实验结果

### 实验设置

**预训练数据集**：
- JFT-4B：40 亿图像的监督分类数据集
- WebLI：大规模图像-文本对比学习数据集

**评估任务**：
- ImageNet 线性评估
- 零样本 ImageNet 分类
- COCO 图像-文本检索
- 语义分割（ADE20k）
- 目标检测（COCO）

### 主要结果

#### 训练效率提升

| 方法 | 计算量 | ImageNet Top-1 |
|------|--------|----------------|
| ViT-B/16 | 1x | 78.5% |
| NaViT-B/16 | 1x | 79.5% |
| ViT-L/16 | 4x | 80.2% |
| NaViT-L/16 | 1x | 80.0% |

**关键发现**：
- NaViT 用 1/4 计算量匹配 ViT-L 的性能
- 最轻量的 NaViT 比等效 ViT 高效 5 倍

#### 可变分辨率训练

| 训练策略 | 评估分辨率 | Top-1 准确率 |
|----------|------------|--------------|
| 固定 R=128 | 128 | 75.2% |
| 固定 R=128 | 256 | 68.3% |
| 可变 R~U(64,128) | 128 | 76.1% |
| 可变 R~U(64,128) | 256 | 72.5% |

**关键发现**：
- 可变分辨率训练在所有评估分辨率上都优于固定分辨率
- 单一可变分辨率模型可以替代多个固定分辨率模型

#### 位置嵌入对比

| 位置嵌入类型 | 10-shot 准确率 | 高分辨率泛化 |
|--------------|----------------|--------------|
| 学习 1D (ViT) | 63.5% | 差 |
| 学习 2D (Pix2Struct) | 64.9% | 差 |
| 分解 (+) | 71.3% | 好 |
| 分解 (堆叠) | 71.5% | 中等 |
| 分解 (乘积) | 71.5% | 中等 |
| 正弦 | 71.3% | 好 |
| 傅里叶 | 71.4% | 好 |

**关键发现**：
- 分解位置嵌入在性能和泛化上都优于传统方法
- 加法组合策略最佳

#### Token 丢弃策略

| 策略 | 平均丢弃率 | Top-1 准确率 |
|------|------------|--------------|
| 固定 0.25 | 25% | 78.0% |
| Beta 分布 | 25% | 78.5% |
| 分辨率相关 | 25% | 79.0% |
| 递减调度 | 可变 | 76.5% |

**关键发现**：
- 可变 token 丢弃优于固定比例
- 分辨率相关的丢弃策略效果最佳
- 训练过程中的调度策略可以进一步提升性能

### 下游任务迁移

#### 语义分割（ADE20k）

| 方法 | mIoU |
|------|------|
| ViT-L/16 | 52.5 |
| NaViT-L/16 | 53.0 |

#### 目标检测（COCO）

| 方法 | mAP |
|------|-----|
| ViT-L/16 | 54.2 |
| NaViT-L/16 | 55.1 |

#### 公平性标注

| 方法 | FairFace 性别 | FairFace 种族 |
|------|---------------|---------------|
| ViT-L/16 | 95.2% | 68.5% |
| NaViT-L/16 | 96.1% | 70.2% |

**关键发现**：
- NaViT 的迁移能力与 ViT 相当或更好
- 在公平性标注任务上，NaViT 显著优于 ViT
- 使用原始宽高比（而非 resize 到正方形）进一步提升公平性标注准确率

## 深度分析

### 研究价值评估

#### 理论贡献
- **贡献 1：Patch n' Pack 范式**
  - 创新点：将 NLP 的 example packing 应用到视觉 Transformer
  - 学术价值：打破了固定分辨率的范式，为 ViT 设计开辟新方向
  - 影响范围：计算机视觉、多模态学习、模型效率

- **贡献 2：分解位置嵌入**
  - 创新点：将 x/y 坐标的位置嵌入分解学习
  - 学术价值：解决了可变分辨率下的位置编码问题
  - 影响范围：Transformer 位置编码设计

#### 实际应用价值
- **应用场景 1：高效视觉模型训练**
  - 适用性：大规模图像预训练
  - 优势：4 倍计算节省或 5 倍吞吐量提升
  - 潜在影响：降低大模型训练成本

- **应用场景 2：灵活推理**
  - 适用性：需要权衡计算成本和性能的场景
  - 优势：单一模型支持多种分辨率
  - 潜在影响：简化部署，降低存储需求

- **应用场景 3：公平性敏感应用**
  - 适用性：人脸分析、公平性审计
  - 优势：保持原始宽高比，减少信息损失
  - 潜在影响：提高公平性标注准确性

#### 领域影响
- **短期影响**：提高 ViT 训练效率，减少计算成本
- **中期影响**：改变视觉模型的输入处理范式
- **长期影响**：推动原生分辨率视觉模型成为标准
- **潜在变革**：从"模型适应数据"到"数据适应模型"

### 方法优势详解

#### 优势 1：训练效率大幅提升
- **描述**：通过打包和可变分辨率，处理更多训练图像
- **技术基础**：序列打包 + 可变分辨率采样 + Token 丢弃
- **实验验证**：NaViT-B/16 用 1/4 计算量匹配 ViT-L 性能
- **对比分析**：比 ViT 高效 4-5 倍

#### 优势 2：推理灵活性
- **描述**：单一模型支持多种分辨率，平滑权衡成本-性能
- **技术基础**：分解位置嵌入 + 分数嵌入
- **实验验证**：在 64-512 分辨率范围内表现稳定
- **对比分析**：无需为不同分辨率训练多个模型

#### 优势 3：更好的宽高比保持
- **描述**：保持图像原始宽高比，避免信息损失
- **技术基础**：Patch n' Pack + 分解位置嵌入
- **实验验证**：在非正方形图像上性能提升显著
- **对比分析**：比 resize 到正方形的 ViT 更好

### 局限性分析

#### 局限 1：注意力开销增加
- **描述**：打包多个图像会增加序列长度，增加注意力计算
- **表现**：对于小模型，打包 8 张图像的注意力开销可达 25%
- **原因**：自注意力的 O(n²) 复杂度
- **影响**：小模型的效率提升可能被注意力开销抵消
- **可能的解决方案**：使用高效注意力机制（如 FlashAttention）

#### 局限 2：实现复杂性
- **描述**：需要修改注意力掩码、池化、位置嵌入等多个组件
- **表现**：实现比标准 ViT 更复杂
- **原因**：需要处理变长序列和跨图像交互
- **影响**：增加了工程实现难度
- **可能的解决方案**：提供开源实现和详细文档

#### 局限 3：批次构造开销
- **描述**：需要智能地打包图像到固定长度序列
- **表现**：约 2% 的 padding token 浪费
- **原因**：贪心打包策略无法完美填充
- **影响**：轻微的效率损失
- **可能的解决方案**：使用更复杂的装箱算法

### 适用性与场景分析

#### 适用场景
- **场景 1：大规模预训练**
  - 适用原因：效率提升显著，数据量大时收益更高
  - 预期效果：4-5 倍训练加速
  - 注意事项：需要适配数据加载管道

- **场景 2：多分辨率部署**
  - 适用原因：单一模型支持多种分辨率
  - 预期效果：简化部署，降低存储成本
  - 注意事项：需要权衡不同分辨率的性能

- **场景 3：宽高比敏感任务**
  - 适用原因：保持原始宽高比，避免信息损失
  - 预期效果：提升非正方形图像的性能
  - 注意事项：需要重新设计数据预处理

#### 不适用场景
- **场景 1：小规模微调**
  - 不适用原因：效率提升不明显，实现复杂
  - 替代方案：使用标准 ViT + 固定分辨率

- **场景 2：实时推理**
  - 不适用原因：可变序列长度可能影响硬件优化
  - 替代方案：使用固定分辨率的量化模型

## 与相关论文对比

### 对比论文选择依据
选择与 NaViT 最相关的 3 篇论文：
1. **ViT**：基础视觉 Transformer
2. **FlexiViT**：支持多种 patch 大小的 ViT
3. **Pix2Struct**：保留宽高比的视觉 Transformer

### [[An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale]] - ViT

#### 基本信息
- **作者**：Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, Jakob Uszkoreit, Neil Houlsby
- **发表时间**：2020 年
- **会议/期刊**：ICLR 2021
- **核心方法**：将图像分割为 patch，用 Transformer 处理

#### 方法对比
| 对比维度 | ViT | NaViT |
|----------|-----|-------|
| 输入处理 | 固定分辨率，resize 到正方形 | 可变分辨率，保持宽高比 |
| 位置嵌入 | 1D 学习嵌入 | 分解 2D 嵌入 |
| 训练效率 | 固定批次形状 | 序列打包 |
| 推理灵活性 | 固定分辨率 | 多分辨率支持 |

#### 性能对比
| 模型 | 计算量 | ImageNet Top-1 |
|------|--------|----------------|
| ViT-B/16 | 1x | 78.5% |
| NaViT-B/16 | 1x | 79.5% |
| ViT-L/16 | 4x | 80.2% |
| NaViT-L/16 | 1x | 80.0% |

#### 关系分析
- **关系类型**：改进
- **本文改进**：解决了固定分辨率的限制，提升了训练效率
- **优势**：更高的效率和灵活性
- **劣势**：实现更复杂
- **互补性**：NaViT 建立在 ViT 基础上，是其自然扩展

### [[FlexiViT: One Model is All You Need for Unlocking Resolution Efficiency in Vision Transformers]] - FlexiViT

#### 基本信息
- **作者**：Lucas Beyer, Pavel Izmailov, Alexander Kolesnikov, Mathilde Caron, Simon Kornblith, Michael Tschannen, Josip Djolonga, Mario Lucic, Neil Houlsby, Sylvain Gelly, Mostafa Dehghani, Xiaohua Zhai
- **发表时间**：2023 年
- **会议/期刊**：--
- **核心方法**：在单个模型中支持多种 patch 大小

#### 方法对比
| 对比维度 | FlexiViT | NaViT |
|----------|----------|-------|
| 灵活性 | 多种 patch 大小 | 任意分辨率和宽高比 |
| 位置嵌入 | 插值 1D 嵌入 | 分解 2D 嵌入 |
| 训练方式 | 随机采样 patch 大小 | 序列打包 + 可变分辨率 |
| 效率提升 | 中等 | 显著 |

#### 性能对比
| 任务 | FlexiViT | NaViT |
|------|----------|-------|
| 训练效率 | 1.5-2x | 4-5x |
| 推理灵活性 | 多分辨率 | 任意分辨率 |

#### 关系分析
- **关系类型**：改进/扩展
- **本文改进**：不仅支持多种 patch 大小，还支持任意宽高比
- **优势**：更全面的灵活性，更高的效率
- **劣势**：实现更复杂
- **互补性**：FlexiViT 的 patch 大小采样可以与 NaViT 结合

### [[Pix2Struct: Screenshot Parsing as Pretraining for Visual Language Understanding]] - Pix2Struct

#### 基本信息
- **作者**：Kenton Lee, Mandar Joshi, Iulia Turc, Hexiang Hu, Fangyu Liu, Julian Eisenschlos, Urvashi Khandelwal, Peter Shaw, Ming-Wei Chang, Kristina Toutanova
- **发表时间**：2022 年
- **会议/期刊**：ICML 2023
- **核心方法**：保留宽高比的 patch 提取，用于文档和图表理解

#### 方法对比
| 对比维度 | Pix2Struct | NaViT |
|----------|------------|-------|
| 宽高比处理 | 保留宽高比 | 保留宽高比 |
| 位置嵌入 | 学习 2D 嵌入 | 分解 2D 嵌入 |
| 分辨率支持 | 有限（maxLen²） | 任意 |
| 训练效率 | 标准 | 序列打包 |

#### 性能对比
| 指标 | Pix2Struct | NaViT |
|------|------------|-------|
| 高分辨率泛化 | 差 | 好 |
| 训练效率 | 标准 | 高 |

#### 关系分析
- **关系类型**：改进
- **本文改进**：解决了 Pix2Struct 2D 嵌入泛化差的问题
- **优势**：更好的泛化能力，更高的效率
- **劣势**：Pix2Struct 专注于文档理解，NaViT 更通用
- **互补性**：NaViT 可以替代 Pix2Struct 的视觉编码器

### 对比总结

| 方法 | 宽高比保持 | 分辨率灵活性 | 训练效率 | 推理灵活性 | 实现复杂度 |
|------|------------|--------------|----------|------------|------------|
| ViT | ❌ | ❌ | 标准 | 低 | 低 |
| FlexiViT | ❌ | 中等 | 中等 | 中等 | 中等 |
| Pix2Struct | ✅ | 有限 | 标准 | 中等 | 中等 |
| **NaViT** | **✅** | **任意** | **高** | **高** | **高** |

## 技术路线定位

### 所属技术路线
本文属于 **高效视觉 Transformer**技术路线，该技术路线的核心特点是：
- 提升 ViT 的训练和推理效率
- 增强 ViT 的灵活性和泛化能力
- 保持或提升模型性能

### 技术路线发展历程
```
ViT (2020) → DeiT (2021) → FlexiViT (2023) → NaViT (2023) → 未来方向
   ↑            ↑              ↑                ↑
 基础架构     高效训练       多分辨率         原生分辨率
```

### 本文在技术路线中的位置
- **承上**：继承了 ViT 的架构，借鉴了 FlexiViT 的多分辨率思想
- **启下**：为后续原生分辨率视觉模型奠定基础
- **关键节点**：首次实现了真正的任意分辨率和宽高比支持

### 具体子方向
本文主要关注 **可变分辨率视觉 Transformer**，该子方向的研究重点是：
- 如何处理不同分辨率和宽高比的图像
- 如何提升训练效率
- 如何在推理时灵活权衡成本和性能

## 未来工作建议

### 作者建议的未来工作
1. **扩展到更多任务**
   - 建议：将 NaViT 应用到视频理解、3D 视觉等任务
   - 可行性：高，视频天然具有可变长度
   - 价值：提升视频理解的效率和灵活性
   - 难度：中等，需要处理时间维度

2. **结合高效注意力机制**
   - 建议：将 NaViT 与 FlashAttention 等高效注意力结合
   - 可行性：高，两者正交
   - 价值：进一步降低计算开销
   - 难度：低

3. **探索更智能的打包策略**
   - 建议：使用学习的打包策略替代贪心算法
   - 可行性：中等
   - 价值：减少 padding 浪费，提升效率
   - 难度：中等

### 基于分析的未来方向
1. **多模态 NaViT**
   - 动机：图像-文本对的长度差异大，打包可以提升效率
   - 可能的方法：将 NaViT 与文本 Transformer 结合
   - 预期成果：提升多模态预训练效率
   - 挑战：跨模态注意力的设计

2. **自适应分辨率选择**
   - 动机：不同图像可能需要不同分辨率
   - 可能的方法：学习一个轻量级网络预测最优分辨率
   - 预期成果：进一步提升效率-性能权衡
   - 挑战：分辨率选择网络的设计

3. **硬件感知优化**
   - 动机：不同硬件对序列长度的偏好不同
   - 可能的方法：根据目标硬件优化打包策略
   - 预期成果：最大化实际部署效率
   - 挑战：需要硬件建模

## 我的综合评价

### 价值评分

#### 总体评分
**8.5/10** - 提出了 Patch n' Pack 范式，解决了视觉 Transformer 的固定分辨率限制，效率提升显著，实用价值高。

#### 分项评分

| 评分维度 | 分数 | 评分理由 |
|----------|------|----------|
| 创新性 | 9/10 | 首次将 NLP 的 example packing 应用到视觉 Transformer，思想新颖 |
| 技术质量 | 8/10 | 方法设计合理，实验充分，分析深入 |
| 实验充分性 | 8/10 | 在多个任务和数据集上验证，消融实验完整 |
| 写作质量 | 9/10 | 清晰易懂，图表丰富，示例详细 |
| 实用性 | 9/10 | 效率提升显著，可以直接应用，实用价值高 |

### 重点关注

#### 值得关注的技术点
1. **序列打包**：将多个图像打包到一个序列的实现细节
2. **分解位置嵌入**：x/y 坐标分别编码的设计
3. **连续 Token 丢弃**：可变丢弃比例和调度策略

#### 需要深入理解的部分
1. **掩码自注意力**：如何防止跨图像注意力
2. **分辨率采样策略**：不同分布的效果差异
3. **对比学习损失处理**：分块对比损失的设计

## 我的笔记

%% 用户可以在这里添加个人阅读笔记 %%

## 相关论文

### 直接相关
- [[An Image is Worth 16x16 Words|ViT]] - 基础视觉 Transformer，NaViT 的基础
- [[FlexiViT: One Model is All You Need|FlexiViT]] - 多 patch 大小的 ViT，NaViT 的前身
- [[Pix2Struct: Screenshot Parsing as Pretraining|Pix2Struct]] - 保留宽高比的 ViT，NaViT 的重要对比

### 背景相关
- [[Attention Is All You Need|Transformer]] - Transformer 基础架构
- [[BERT: Pre-training of Deep Bidirectional Transformers|BERT]] - NLP 中的 example packing 启示

### 后续工作
- [[SigLIP: Sigmoid Loss for Language Image Pre-Training|SigLIP]] - 可能受益于 NaViT 的多模态模型
- [[PaliGemma: A versatile 3B VLM for transfer|PaliGemma]] - 原生分辨率视觉编码器的应用

## 外部资源
- **Scenic 库**：https://github.com/google-research/scenic
- **JAX/FLAX**：https://github.com/google/jax

> [!tip] 关键启示
> 固定分辨率是 CNN 时代的遗留物——通过序列打包，视觉 Transformer 可以像语言模型一样处理任意长度的输入，同时提升效率和灵活性。

> [!warning] 注意事项
> - 实现比标准 ViT 更复杂，需要修改注意力和池化
> - 对于小模型，打包的注意力开销可能抵消效率提升
> - 需要重新设计数据加载管道

> [!success] 推荐指数
> ⭐⭐⭐⭐⭐ 强烈推荐阅读！这是视觉 Transformer 效率优化的重要工作，Patch n' Pack 范式可能成为未来视觉模型的标准做法。

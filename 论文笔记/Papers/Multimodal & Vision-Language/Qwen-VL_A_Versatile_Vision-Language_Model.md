---
date: "2026-05-06"
paper_id: "arXiv:2308.12966"
title: "Qwen-VL: A Versatile Vision-Language Model for Understanding, Localization, Text Reading, and Beyond"
authors: "Jinze Bai, Shuai Bai, Shusheng Yang, Shijie Wang, Sinan Tan, Peng Wang, Junyang Lin, Chang Zhou, Jingren Zhou"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen-VL
  - Vision-Language-Model
  - Visual-Grounding
  - OCR
  - Cross-Attention
  - Multimodal
quality_score: "7.5/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen-VL: A Versatile Vision-Language Model

## 核心信息
- **论文ID**：arXiv:2308.12966
- **作者**：Jinze Bai, Shuai Bai, Shusheng Yang, Shijie Wang, Sinan Tan, Peng Wang, Junyang Lin, Chang Zhou, Jingren Zhou（阿里通义千问团队）
- **机构**：Alibaba Group
- **发布时间**：2023-08（v3: 2023-10）
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2308.12966)
- **模型**：Qwen-VL 和 Qwen-VL-Chat（对话版）

## 摘要翻译

### 英文摘要
We introduce the Qwen-VL series, a set of large-scale vision-language models designed to perceive and understand both text and images. Qwen-VL is built upon a 7B-parameter LLM, Qwen-7B, augmented with a visual encoder and a position-aware vision-language adapter. The model is designed to handle tasks including image captioning, question answering, visual localization, and text reading. Qwen-VL-Chat supports flexible multi-turn multimodal interaction through alignment training. The model achieves strong performance across a wide range of benchmarks compared to similarly sized models.

### 中文翻译
Qwen-VL 系列是通义千问团队推出的首批大规模视觉-语言模型。基于 7B LLM（Qwen-7B），配备视觉编码器和位置感知的跨模态适配器，支持图像描述、视觉问答、视觉定位和文字阅读等任务。Qwen-VL-Chat 通过对齐训练支持灵活的多轮多模态交互，在同等规模模型上取得领先性能。

### 核心要点提炼
- **研究背景**：2023 年初，GPT-4V 尚未发布，开源多模态大模型稀缺
- **核心方法**：ViT 视觉编码 + 单层交叉注意力适配器 + Qwen-7B LLM，三段式训练
- **主要结果**：在视觉定位、OCR、few-shot 多模态理解上达到 SOTA
- **研究意义**：为后续 Qwen2-VL、Qwen2.5-VL 奠定架构基础

## 研究背景与动机

### 领域现状
2023 年是多模态大模型的元年。BLIP-2、LLaVA、InstructBLIP 已展示了 LLM + 视觉编码器的范式，但多数模型在**细粒度视觉定位**（如指代 grounding）和**文字阅读**（OCR）上表现有限。

### Qwen-VL 的定位
Qwen-VL 瞄准三个差异化能力：
1. **细粒度视觉定位**：不只是问答，还要能指认"图中的人在哪个位置"
2. **强文字理解**：文档、表格、路标的 OCR 阅读
3. **多轮多模态对话**：Qwen-VL-Chat 支持交互式多模态对话

## 研究问题

1. 如何让 LLM 获得高精度的视觉定位能力（输出 bounding box 坐标）？
2. 如何在一次前向传播中同时完成图像理解和文字阅读？
3. 如何高效地将视觉特征注入 LLM 而不破坏其语言能力？

## 方法概述

### 前置知识：Cross-Attention vs Q-Former vs MLP Projector

当时主流的多模态融合方案有三种：
- **LLaVA**：MLP 投影器，最简单，视觉 token 直接拼到文本序列前
- **BLIP-2**：Q-Former，用可学习的 query token 压缩视觉信息
- **Flamingo**：Cross-Attention，在 LLM 每层插入交叉注意力接收视觉特征

Qwen-VL 选择了类似 Flamingo 的方案，但做了简化。

### 整体架构

![[qwenvl_page1.png|800]]

> 图1：Qwen-VL 整体架构。视觉编码器 + 位置感知适配器 + Qwen-7B LLM

Qwen-VL 由三部分组成：

**1. 视觉编码器（Vision Transformer）**
- 使用 ViT-G（约 1.9B 参数），以 OpenCLIP 预训练权重初始化
- 输入图像被 resize 到 $224 \times 224$ 分辨率
- 输出 $16 \times 16$ 的 patch 特征序列（257 tokens，含 CLS）

**2. 位置感知跨模态适配器（Position-aware Vision-Language Adapter）**
- **单层交叉注意力**：将视觉 token 压缩为固定长度的可学习 query 向量
- 引入 **2D 位置编码**（正弦编码），让适配器知道每个视觉 token 的空间位置
- 输出 256 个压缩后的视觉 token
- 设计目的：压缩视觉序列长度（256 vs 原始 257），减少 LLM 的计算开销，同时保留空间信息

**3. 大语言模型（Qwen-7B）**
- 通义千问 7B 基础模型，32 层 Transformer
- 视觉 token 拼接到文本 token 序列前，一起输入 LLM
- 输出可以是文本、bounding box 坐标（`<box>(x1,y1),(x2,y2)</box>` 格式），或两者混合

### 三段式训练流程

| 阶段 | 目标 | 数据 | 冻结 |
|------|------|------|------|
| **Stage 1**：预训练 | 对齐视觉和语言表征 | 1.4B 图文对（LAION、DataComp 等） | LLM 冻结，仅训练适配器+视觉编码器 |
| **Stage 2**：多任务微调 | 获得多模态能力 | 高质量标注数据（定位、OCR、VQA、Caption） | LLM 解冻 |
| **Stage 3**：对话对齐 | 实现多轮对话 | 人工标注的多轮多模态对话 | 全模型 |

**设计直觉**：分阶段训练确保视觉编码器先学会"看懂"，再让 LLM 学会"根据看的东西回答"，最后学会"对话"。一步到位会导致视觉能力被语言能力淹没。

#### 关键设计：Bounding Box Tokenization

Qwen-VL 将 bounding box 坐标表示为特殊 token 序列：

$$\text{<box>}(x_1, y_1), (x_2, y_2)\text{</box>}$$

所有坐标归一化到 $[0, 1000)$ 区间。这种设计让 ground truth 和预测都共享同一套 token 空间，LLM 可以直接生成定位坐标。

### 方法关键创新点

- **位置感知适配器**：单层交叉注意力 + 2D 位置编码，用少数可学习 query 压缩视觉信息，同时保留空间位置。比 Q-Former 更轻量，比 MLP 投影器更"理解"视觉特征
- **Bounding Box 作为自然语言**：将定位框表示为特殊 token，无需额外的检测头或回归网络，统一了理解和定位任务
- **三段式训练**：预训练（对齐）→ 多任务微调（能力）→ RLHF（对话），每阶段有针对性的数据配比和参数冻结策略

## 实验结果

### 关键基准

| 任务 | 基准 | Qwen-VL | 对比 |
|------|------|---------|------|
| 图像描述 | Flickr30K | SOTA | 超 LLaVA-1.5 |
| 视觉问答 | VQAv2 | 78.8 | 同规模领先 |
| 视觉定位 | RefCOCO | 89.4 | 显著领先 |
| 文字阅读 | TextVQA | 63.8 | SOTA |
| Few-shot | MME | 领先 | 强泛化 |

![[radar1_page1.png|700]]

> 图2：Qwen-VL 多维度能力雷达图

![[figure_fewshot_compare.png|700]]

> 图3：Few-shot 能力对比

## 深度分析

### Qwen-VL 在 VL 发展史中的位置

Qwen-VL 是 Qwen 多模态系列的开山之作，在 2023 年确立了 **ViT + Cross-Attention + LLM + Bounding Box Token** 的基本范式。这个范式的核心设计（bbox tokenization、位置感知适配器）被后续 Qwen2-VL 继承和大幅改进。

### 局限性
- **固定 224² 分辨率**：无法处理高分辨率文档、大表格
- **单层交叉注意力**虽轻量但信息瓶颈明显——256 个 query 难以保留细粒度视觉信息
- **视频能力缺失**：仅支持图像，不支持视频
- 适配器虽有 2D 位置编码，但面对任意分辨率的动态图像仍不够灵活

## 我的综合评价

### 总体评分：7.5/10

| 维度 | 分数 | 理由 |
|------|------|------|
| 创新性 | 7/10 | 架构多是已有组件的组合，但 bbox tokenization 有创新 |
| 技术质量 | 7/10 | 三阶段训练设计合理 |
| 实验充分性 | 7/10 | 覆盖多个基准 |
| 写作质量 | 8/10 | 清晰完整 |
| 实用性 | 8/10 | 直接可用的开源多模态模型 |

> [!tip] 关键启示
> Bounding box 不需要专门的检测头——用特殊 token 表示坐标，LLM 天生就能学会"指认位置"。

> [!warning] 注意事项
> - 架构已被 Qwen2-VL 完全超越，仅供历史参考
> - 固定分辨率是最主要的瓶颈

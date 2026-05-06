---
date: "2026-05-06"
paper_id: "arXiv:2511.21631"
title: "Qwen3-VL Technical Report"
authors: "Shuai Bai et al. (Qwen Team, 64 authors)"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen3-VL
  - MoE
  - DeepStack
  - Interleaved-MRoPE
  - Long-Context
  - Multimodal-Reasoning
quality_score: "9.0/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen3-VL Technical Report

## 核心信息
- **论文ID**：arXiv:2511.21631
- **作者**：Shuai Bai 等（Qwen Team，64 位作者）
- **机构**：Alibaba Group
- **发布时间**：2025-11
- **链接**：[arXiv](https://arxiv.org/abs/2511.21631)
- **模型**：Dense 2B/4B/8B/32B + MoE 30B-A3B/235B-A22B

## 摘要翻译

Qwen3-VL 是 Qwen VL 系列的最新版本，引入密集（Dense）和混合专家（MoE）两种架构。三大核心支柱：**超越纯文本骨干的纯文本理解能力**、**原生 256K 多模态长上下文**、**领先的多模态推理**。架构关键升级包括增强的 Interleaved-MRoPE（时空建模）、DeepStack（多层 ViT 特征集成）和基于文本的视频时间对齐（T-RoPE 演进）。

## 方法概述

### 整体架构

![[qwen3vl_arc.jpg|800]]

> 图1：Qwen3-VL 整体架构

### 核心创新1：Dense + MoE 双架构

Qwen3-VL 是首个同时发布 Dense 和 MoE 版本的 VL 系列：

| 架构 | 参数 | 激活参数 | 适用场景 |
|------|------|----------|----------|
| Dense | 2B/4B/8B/32B | 全部 | 低延迟、边缘部署 |
| MoE | 30B-A3B | 3B | 推理能力 vs 效率平衡 |
| MoE | 235B-A22B | 22B | 最强性能 |

**MoE 设计**：用 Mixture-of-Experts 替换 FFN 层，每个 token 只激活部分专家，实现"大模型的能力，小模型的推理成本"。

### 核心创新2：Interleaved-MRoPE

Qwen2-VL 的 M-RoPE 将位置编码分解为 $(t, h, w)$ 三维。Qwen3-VL 进一步升级为 **Interleaved-MRoPE**：

- M-RoPE 的三维分量是连续排列的（前 d/3 给时间，中间 d/3 给高度，后 d/3 给宽度）
- Interleaved-MRoPE 将三个分量**交错排列**：$[t_1, h_1, w_1, t_2, h_2, w_2, ...]$
- 效果：每个注意力头同时接收三种位置信息，而非不同头专注于不同维度

### 核心创新3：DeepStack

![[qwen3vl_head.png|600]]

Qwen2-VL 只用 ViT 的最后一层特征。Qwen3-VL 的 **DeepStack** 从 ViT 的多层提取特征并融合：

- 浅层特征：边缘、纹理等低级视觉信息
- 中层特征：物体部件、形状等中级语义
- 深层特征：完整物体、场景等高级语义

融合方式：将多层特征通过可学习的权重加权求和后送入 LLM。这使得模型能同时利用不同粒度的视觉信息——对精细定位和 OCR 特别有帮助。

### 核心创新4：256K 原生长上下文

Qwen3-VL 将上下文窗口扩展到 256K token，支持：
- 数十张高清图片 + 超长文本交错的输入
- 小时级视频的逐帧理解
- 长文档的全量解析（无需分页）

![[heatmap_comparison_qwen-vl_needle_in_a_haystack_performance_page1.png|600]]

> 图2：Needle-in-a-Haystack 长上下文检索性能

### 核心创新5：多模态推理

Qwen3-VL 在多模态推理基准上达到领先水平，主要通过：
- 增强的 Chain-of-Thought 训练数据（图文交错推理链）
- 数学推理的视觉化能力（看懂几何图形、函数图像、数据图表）
- 32B Dense 版本在 MMMU、MathVista、MathVision 等基准上超越同级模型

## 实验结果

| 基准 | Qwen3-VL-235B | Qwen3-VL-32B | Qwen2.5-VL-72B |
|------|:---:|:---:|:---:|
| MMMU | **75.0** | 68.1 | 65.2 |
| MathVista | **78.5** | 74.9 | 70.1 |
| MathVision | **42.8** | 39.0 | -- |
| MMBench | 89.4 | **89.7** | 88.5 |
| OCRBench | **92.3** | 90.5 | 89.6 |

![[multi_lan_ocr_support.png|600]]

> 图3：多语言 OCR 支持

## 深度分析

### VL 系列演进关键节点

| 代际 | 核心升级 | 旗舰尺寸 |
|------|----------|----------|
| Qwen-VL | Cross-Attn + Bbox Token | 7B |
| Qwen2-VL | NDR + M-RoPE + 视频 | 72B Dense |
| Qwen2.5-VL | Window Attn + 绝对时间 + Agent | 72B Dense |
| **Qwen3-VL** | MoE + DeepStack + Interleaved-MRoPE + 256K | **235B MoE** |

### 局限性
- 235B MoE 虽激活仅 22B，但总参数量巨大，部署门槛高
- Interleaved-MRoPE 的交错设计缺乏充分消融实验
- 视频 Agent 能力未量化为 benchmark

## 我的综合评价

### 总体评分：9.0/10

| 维度 | 分数 |
|------|:---:|
| 创新性 | 8/10 |
| 技术质量 | 9/10 |
| 实验充分性 | 8/10 |
| 写作质量 | 9/10 |
| 实用性 | 10/10 |

> [!tip] 关键启示
> MoE + DeepStack + 256K 构成 VL 模型的新三角——效率、细粒度感知、长上下文各占一角。

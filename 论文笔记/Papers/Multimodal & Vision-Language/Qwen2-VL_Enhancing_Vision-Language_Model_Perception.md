---
date: "2026-05-06"
paper_id: "arXiv:2409.12191"
title: "Qwen2-VL: Enhancing Vision-Language Model's Perception of the World at Any Resolution"
authors: "Peng Wang, Shuai Bai, Sinan Tan, Shijie Wang, Zhihao Fan, Jinze Bai, Keqin Chen, Xuejing Liu, Jialin Wang, Wenbin Ge, Yang Fan, Kai Dang, Mengfei Du, Xuancheng Ren, Rui Men, Dayiheng Liu, Chang Zhou, Jingren Zhou, Junyang Lin"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen2-VL
  - Dynamic-Resolution
  - M-RoPE
  - Video-Understanding
  - Vision-Language-Model
quality_score: "8.5/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen2-VL: Enhancing Vision-Language Model's Perception of the World at Any Resolution

## 核心信息
- **论文ID**：arXiv:2409.12191
- **作者**：Peng Wang, Shuai Bai 等（阿里通义千问团队）
- **机构**：Alibaba Group
- **发布时间**：2024-09（v2: 2024-10）
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2409.12191)
- **模型**：Qwen2-VL 2B / 8B / 72B

## 摘要翻译

### 英文摘要
We present Qwen2-VL, the latest version of our vision-language models based on Qwen2 in the Qwen model family. Qwen2-VL introduces Naive Dynamic Resolution (NDR), allowing the model to process images of varying resolutions natively, and Multimodal Rotary Position Embedding (M-RoPE), which effectively fuses positional information across text, images, and videos. Qwen2-VL can perceive and understand both images and videos at arbitrary resolutions natively, achieving performance comparable to or surpassing GPT-4o and Claude 3.5 Sonnet at the 72B scale. The model is released at 2B, 8B, and 72B parameter sizes.

### 中文翻译
Qwen2-VL 基于 Qwen2 LLM 构建，引入了两大关键创新：Naive Dynamic Resolution（原生动态分辨率，让模型处理任意分辨率图像）和 Multimodal Rotary Position Embedding（M-RoPE，有效融合文本、图像和视频的位置信息）。模型可在任意分辨率下感知图像和视频，72B 版本性能媲美或超越 GPT-4o 和 Claude 3.5 Sonnet。

### 核心要点提炼
- **研究背景**：Qwen-VL 固定 224² 分辨率严重限制了细粒度理解
- **核心方法**：动态分辨率（NDR）+ M-RoPE + 统一视频处理
- **主要结果**：72B 版本匹配 GPT-4o/Claude 3.5 Sonnet
- **研究意义**：首次系统研究了 LVLM 的 scaling law，开源最强 VL 系列之一

## 研究背景与动机

### Qwen-VL 的三大瓶颈
1. **固定分辨率**：所有图像缩放到 224²，丢失细粒度信息（文档文字、小物体）
2. **位置编码不统一**：视觉和文本使用独立的位置编码，多模态位置关系无法建模
3. **无视频能力**：Qwen-VL 仅支持静态图像

### Qwen2-VL 的解决方案
- **NDR**：不再固定分辨率，而是将任意尺寸图像按 ViT patch 大小切成小块，保持原生分辨率
- **M-RoPE**：用统一的旋转位置编码表示文本（1D）、图像（2D）、视频（3D）的位置关系
- **统一视觉表示**：图像和视频统一为"视觉帧序列"，视频只是多了时间维度

## 方法概述

### 整体架构

![[qwen2_vl.png|800]]

> 图1：Qwen2-VL 整体架构，核心变化是动态分辨率处理和 M-RoPE

Qwen2-VL 保留 Qwen-VL 的三件套（ViT + Adapter + LLM），但在视觉处理流程上做了根本性改造。

### 核心创新1：Naive Dynamic Resolution (NDR)

QL2-VL 处理图像的流程：

**步骤1：移除固定缩放的瓶颈**
- 旧：所有图 resize到 224² → 丢失信息
- 新：保持原图宽高比，只在必要时 resize 到不超过 $28 \times N_{max}$ 像素（$28$ 是 ViT patch size）

**步骤2：动态分块策略**
- 将图像按 ViT 的 patch grid 网格化
- 如果图像分辨率超过预定义上限，切分为多个子图
- 每个子图独立经过 ViT 编码后，拼接为视觉 token 序列

**步骤3：全局+局部融合**
- 一张缩略图提供全局上下文
- 多个高清子图提供细粒度信息
- 两者的 token 序列拼接输入 LLM

NDR 的核心思想类似 **"放大镜"**——先看一眼全局（缩略图），再仔细看每个局部（子图）。这使得模型在不显著增加计算量的前提下获得了"任意分辨率"的能力。

### 核心创新2：Multimodal Rotary Position Embedding (M-RoPE)

![[mrope.png|700]]

> 图2：M-RoPE 将 RoPE 从 1D 扩展到多模态的 3D 位置编码

**背景**：RoPE（Rotary Position Embedding）在纯文本 LLM 中非常成功。但视觉和视频输入引入的是 2D/3D 位置关系。

**M-RoPE 的设计**：
- 将 RoPE 的旋转维度拆分为三个分量：
  - $\frac{d}{3}$ 维给**时间**位置（视频帧序号）
  - $\frac{d}{3}$ 维给**高度**位置（空间 Y 坐标）
  - $\frac{d}{3}$ 维给**宽度**位置（空间 X 坐标）
- 纯文本：三个分量相同（退化为标准 RoPE）
- 图像：时间分量固定为0，高度和宽度分量编码 2D patch 位置
- 视频：三组全部激活，编码 $(t, h, w)$

**为什么 M-RoPE 重要**：它让 LLM 首次同时"理解"文本的先后顺序、图像的上下左右关系、视频的时序和空间变化。一个统一的数学框架处理所有模态的位置关系。

数学上，M-RoPE 的旋转变换：

$$q_{(t,h,w)} = R_t \cdot R_h \cdot R_w \cdot q$$

其中 $R_t$, $R_h$, $R_w$ 分别为时间、高度、宽度维度的旋转矩阵，作用于 query 向量的对应子空间。

### 核心创新3：统一图像和视频处理

Qwen2-VL 将图像和视频统一为**"视觉帧序列"**：
- 图像 = 1 帧
- 视频 = $N$ 帧（$N$ 由动态 FPS 采样决定）

每个视觉帧使用相同的 NDR 处理流程，帧间通过 M-RoPE 的时间分量建立时序关系。

**视频 token 计算量控制**：
- 视频总 token 数 = $N_{frames} \times T_{per\_frame}$
- 对于长视频，$T_{per\_frame}$ 可能达到数千 token
- Qwen2-VL 通过 NDR 自适应调整每帧的分辨率来平衡总计算量

### Scaling Law 研究

![[scale.jpg|600]]

> 图3：Qwen2-VL 的 Scaling Law 研究——视觉编码器和 LLM 的规模关系

Qwen2-VL 首次系统性研究了 LVLM 的 scaling law：
- 视觉编码器大小和 LLM 大小需要**协调增长**：小 LLM 配大 ViT 收益递减
- 达到某阈值后，加大 LLM 比加大 ViT 更有效
- 这直接催生了 Qwen2-VL 的三个尺寸：2B（小 LLM + 小 ViT）、8B（中 LLM + 中 ViT）、72B（大 LLM + 大 ViT）

## 实验结果

### 关键基准（72B vs GPT-4o/Claude 3.5 Sonnet）

| 基准 | Qwen2-VL-72B | GPT-4o | Claude 3.5 Sonnet |
|------|:---:|:---:|:---:|
| MMBench | **86.1** | 85.0 | 84.5 |
| MME-Perception | **1744** | 1730 | -- |
| DocVQA | 94.5 | **94.7** | 89.1 |
| MathVista | **71.2** | 65.5 | 65.2 |
| Video-MME | **74.7** | 71.9 | 53.7 |

![[qwen2_vl_example.jpg|600]]

> 图4：Qwen2-VL 多模态能力示例

### 视频理解

![[qwen2_vl_frame.jpg|600]]

> 图5：视频帧采样和时序理解

Qwen2-VL 在视频理解上大幅领先——因为 M-RoPE 的时间分量提供了原生时序建模，而不是像其他模型用"帧序号文本标签"这种非原生方案。

## 深度分析

### Qwen2-VL 相对于 Qwen-VL 的飞跃

| 维度 | Qwen-VL | Qwen2-VL |
|------|---------|----------|
| 分辨率 | 固定 224² | 动态，任意分辨率 |
| 位置编码 | 2D 正弦（仅视觉适配器） | M-RoPE（多模态统一） |
| 视频 | 不支持 | 原生支持 |
| 模型尺寸 | 仅 7B | 2B/8B/72B |
| 视觉编码器 | ViT-G | 多尺寸 ViT |

### 局限性
- NDR 的动态分块策略在高分辨率下仍会显著增加视觉 token 数（每个子图 = 256+ token）
- M-RoPE 的维度划分是硬编码的（各 1/3），可能不是最优分配
- 视频帧的采样策略相对简单（均匀采样），对于"关键事件定位"不够智能
- 训练和推理成本高（72B 版本需要大量计算资源）

## 我的综合评价

### 总体评分：8.5/10

| 维度 | 分数 | 理由 |
|------|------|------|
| 创新性 | 8/10 | NDR 和 M-RoPE 都是原创新方法 |
| 技术质量 | 9/10 | 系统工程极为扎实，scaling law 研究加分 |
| 实验充分性 | 8/10 | 覆盖多模态主要基准 |
| 写作质量 | 8/10 | 技术报告结构清晰 |
| 实用性 | 9/10 | 开源 2B/8B/72B，直接可用 |

> [!tip] 关键启示
> M-RoPE 用统一的数学框架处理文本（1D）、图像（2D）、视频（3D）的位置关系——多模态的位置编码不该是各自独立的。

> [!warning] 注意事项
> - 动态分辨率增加推理延迟
> - 小模型（2B）在复杂任务上能力有限

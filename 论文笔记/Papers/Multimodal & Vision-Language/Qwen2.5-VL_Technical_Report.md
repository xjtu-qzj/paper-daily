---
date: "2026-05-06"
paper_id: "arXiv:2502.13923"
title: "Qwen2.5-VL Technical Report"
authors: "Shuai Bai, Keqin Chen, Xuejing Liu, Jialin Wang, Wenbin Ge, Sibo Song, Kai Dang, Peng Wang, Shijie Wang, Jun Tang 等"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen2.5-VL
  - Window-Attention
  - Visual-Agent
  - Dynamic-Resolution
  - Video-Understanding
  - Document-Parsing
quality_score: "9.0/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen2.5-VL Technical Report

## 核心信息
- **论文ID**：arXiv:2502.13923
- **作者**：Shuai Bai, Keqin Chen, Xuejing Liu 等（阿里通义千问团队）
- **机构**：Alibaba Group
- **发布时间**：2025-02
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2502.13923)
- **模型**：Qwen2.5-VL 3B / 7B / 72B

## 摘要翻译

### 中文翻译
Qwen2.5-VL 在 Qwen2-VL 基础上实现了重大飞跃。核心升级包括：从零训练的**原生动态分辨率 ViT**（Window Attention 实现高效任意分辨率处理）、**绝对时间编码**（支持超长视频的事件级时间定位）、增强的**精细定位**（bounding box + point coords）、鲁棒的**文档解析**（发票、表格、图表）、以及**视觉 Agent** 能力（操作计算机和移动设备）。预训练数据从 1.2T 扩展到约 4T token。72B 版本匹配 GPT-4o 和 Claude 3.5 Sonnet 的综合表现。

### 核心要点提炼
- **研究背景**：Qwen2-VL 的 ViT 仍从固定分辨率预训练模型初始化，限制动态分辨率效率
- **核心方法**：从零训练的 Window Attention ViT + 绝对时间编码 + 动态 FPS
- **主要结果**：匹配 GPT-4o/Claude 3.5 Sonnet，在视觉 Agent 和文档解析上建立新标杆
- **研究意义**：从"理解"到"行动"的跨越——VL 模型首次具备可用的 Agent 能力

## 研究背景与动机

### Qwen2-VL 的遗留问题
1. **ViT 非原生的动态分辨率**：Qwen2-VL 的 ViT 从固定 224² 的预训练模型初始化，虽然 NDR 可以处理任意分辨率，但 ViT 骨干本身没有为动态分辨率优化
2. **视频时序定位不够精确**：M-RoPE 的时间分量提供了帧间位置关系，但缺少"第几分钟发生了某事"的绝对时间感
3. **文档/图表解析不鲁棒**：对于复杂表格、流程图、科学图表的结构化提取仍有困难
4. **缺少行动能力**：模型能"看懂"屏幕但不能"操作"屏幕

### Qwen2.5-VL 的核心解决思路
- **从零训练的 Window Attention ViT**：原生支持任意分辨率和长序列
- **绝对时间编码**：以秒为单位标注事件，实现"3分15秒发生了什么"级别的时间定位
- **视觉 Agent 能力**：模型输出操作指令（点击、滑动、键入），能控制 GUI 界面

## 方法概述

### 整体架构

![[qwen2.5vl_arc.jpeg|800]]

> 图1：Qwen2.5-VL 整体架构。核心变化：Window Attention ViT + 绝对时间编码 + 动态 FPS

### 核心创新1：原生动态分辨率 ViT（Window Attention）

**问题**：标准 ViT 的 self-attention 复杂度为 $O(N^2)$，$N$ 是 patch 数。动态分辨率下 $N$ 可能达到数万，导致计算爆炸。

**方案**：**Window Attention**——将 ViT 的全局自注意力替换为滑动窗口注意力。

![[head.jpg|600]]

> 图2：Window Attention 机制示意

**Window Attention 的设计**：

标准 ViT 层：
$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}}\right)V$$

Window Attention 将 $K$ 和 $V$ 限制在 $Q$ 的局部窗口内：

$$\text{WindowAttn}(Q, K, V) = \text{softmax}\left(\frac{QK_{window}^T}{\sqrt{d}}\right)V_{window}$$

- 窗口大小：$W \times W$（如 $14 \times 14$）
- 不同层使用不同窗口偏移，确保信息跨窗口传播
- 复杂度从 $O(N^2)$ 降至 $O(N \cdot W^2)$

**为什么从零训练**：Window Attention 改变了 ViT 的信息流模式，用预训练权重初始化会产生分布不匹配。从零训练虽然贵，但保证了 ViT 和动态分辨率的原生兼容。

![[bullet_introduction.png|700]]

> 图3：文档解析和图表理解能力展示

### 核心创新2：绝对时间编码与动态 FPS

**Qwen2-VL 的局限**：M-RoPE 只提供帧间的**相对**位置关系，模型无法回答"第5分钟发生了什么"这类问题。

**Qwen2.5-VL 的绝对时间编码**：
- 每个视频帧附加上距视频起点的绝对秒数
- 秒数被量化为时间 token，和视觉 token 一起输入 LLM
- 模型能学习"t=195 秒 = 3分15秒"的事件-时间绑定

**动态 FPS 采样**：
- Qwen2-VL：固定帧间隔采样（如每秒1帧）
- Qwen2.5-VL：根据视频内容动态调整采样频率：
  - 镜头切换密集段 → 高 FPS
  - 静态长镜头段 → 低 FPS
- 效果：用相同的总帧数预算，覆盖更长的视频（小时级），同时不错过关键事件

### 核心创新3：精细定位升级

在 Qwen-VL 的 bounding box tokenization 基础上，Qwen2.5-VL 新增：

- **点定位**：`<point>(x, y)</point>` — 精确定位到像素级
- **多目标同时定位**：一次回答可以同时定位多个物体
- **层级定位**：大框包含小框，支持文档的段落→行→词层级结构

这对文档解析尤为重要——先定位表格区域（大框），再定位每个单元格（小框），再提取每个单元格的文字。

### 核心创新4：视觉 Agent 能力

Qwen2.5-VL 可以直接作为 GUI Agent：

**输入**：当前屏幕截图 + 操作历史 + 自然语言指令
**输出**：操作指令（点击坐标、滑动方向、键入文本、等待）

模型被微调来理解 GUI 元素——按钮、文本框、下拉菜单的位置和功能。这使 Qwen2.5-VL 能：
- 操作计算机完成多步任务（如"帮我订一张北京到上海的机票"）
- 操作手机完成 App 内任务
- 理解错误提示并重试

这项能力的训练数据来自大量人类操作 GUI 的轨迹标注。

## 实验结果

### 关键基准

| 基准 | Qwen2.5-VL-72B | GPT-4o | Claude 3.5 Sonnet |
|------|:---:|:---:|:---:|
| MMBench | **88.5** | 85.5 | 85.0 |
| MathVista | **76.7** | 65.5 | 67.1 |
| DocVQA | 95.2 | **96.1** | 91.5 |
| MME-Perception | **1804** | 1740 | 1725 |
| OCRBench | **89.6** | 82.3 | 79.8 |
| Video-MME | **76.8** | 71.9 | 55.2 |

### 视觉 Agent 基准

| 任务 | Qwen2.5-VL-72B |
|------|:---:|
| AndroidWorld | 34.7% task success |
| OSWorld | 22.3% |
| WebVoyager | 42.1% |

> 注：视觉 Agent 是全新方向，绝对数字较低但代表了 2025 年初的 SOTA。

## 深度分析

### Qwen VL 系列演进总结

| 能力 | Qwen-VL (2023) | Qwen2-VL (2024) | Qwen2.5-VL (2025) |
|------|:---:|:---:|:---:|
| 图像分辨率 | 固定 224² | 动态（NDR） | 动态（Native） |
| 位置编码 | 2D 正弦 | M-RoPE | M-RoPE + 绝对时间 |
| 视频 | 不支持 | 相对时序 | 绝对时间 + 动态 FPS |
| ViT | 预训练 ViT-G | 预训练多尺度 ViT | 从零训练的 Window Attn ViT |
| 定位 | Bbox | Bbox + 类别 | Bbox + Point + 层级 |
| Agent | 不支持 | 不支持 | 原生支持 |
| 文档 | 基础 OCR | -- | 结构化解析 |

### Qwen2.5-VL vs Qwen2-VL 的核心差异

- **Window Attention** 是最被低估的创新：它让动态分辨率从"能用"变为"高效用"，减少了长视觉序列的平方级计算
- **绝对时间编码**看似一个小改动，但解决了视频 QA 中"when"类问题的根本瓶颈
- **视觉 Agent** 是从"感知智能"到"行动智能"的质变——这不是 benchmark 上的几个百分点，而是能力的品类扩展

### 局限性
- 从零训练 ViT 成本极高（4T token 预训练）
- Window Attention 虽然高效，但窗口间的信息交换靠多层叠加，可能丢失长距离依赖
- 视觉 Agent 的成功率（22-42%）离实用还有距离
- 72B 推理成本高，3B/7B 在复杂任务上降级明显

## 我的综合评价

### 总体评分：9.0/10

| 维度 | 分数 | 理由 |
|------|------|------|
| 创新性 | 8/10 | Window Attn + 绝对时间 + Agent 三线并进 |
| 技术质量 | 9/10 | 系统工程质量极高 |
| 实验充分性 | 8/10 | 覆盖常规 + Agent 基准 |
| 写作质量 | 8/10 | 技术报告清晰 |
| 实用性 | 10/10 | 开源 3B/7B/72B，覆盖边缘到云端全场景 |

> [!tip] 关键启示
> VL 模型的下一步不是更大的 benchmark 分数，而是从"看"到"做"——视觉 Agent 是 VL 技术走向实际应用的关键一步。

> [!warning] 注意事项
> - Agent 能力目前仅适用于 GUI 场景
> - 3B 模型在 Agent 任务上能力显著不足
> - 从零训练 ViT 的门槛极高，个人/小团队难以复现

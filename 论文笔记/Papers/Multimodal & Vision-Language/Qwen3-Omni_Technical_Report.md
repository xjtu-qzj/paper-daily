---
date: "2026-05-06"
paper_id: "arXiv:2509.17765"
title: "Qwen3-Omni Technical Report"
authors: "Qwen Team"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen3-Omni
  - MoE
  - AuT
  - Multi-Codebook
  - Speech-Generation
  - Audio-Transformer
quality_score: "8.5/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen3-Omni Technical Report

## 核心信息
- **论文ID**：arXiv:2509.17765
- **作者**：Qwen Team
- **机构**：Alibaba Group
- **发布时间**：2025-09
- **链接**：[arXiv](https://arxiv.org/abs/2509.17765)
- **许可**：Apache 2.0

## 摘要翻译

Qwen3-Omni 在 Qwen2.5-Omni 的 Thinker-Talker 架构基础上进行五项关键升级：(1) Thinker 和 Talker 均升级为 MoE 设计；(2) 用 AuT（Audio Transformer）替代 Whisper 作为音频编码器，在 20M 小时音频上训练；(3) 多码本语音生成搭配轻量因果 ConvNet 解码器；(4) 支持 40+ 分钟音频输入，119 种书面语言和 19 种口语；(5) 冷启动流式端到端延迟仅 234ms。在 36 个音频/视听基准中的 32 个上达到开源 SOTA。

## 方法概述

### 整体架构

![[q3o_page1.png|800]]

> 图1：Qwen3-Omni 系统架构总览

### 升级1：MoE Thinker + MoE Talker

Qwen2.5-Omni 使用 7B Dense 模型作为 Thinker。Qwen3-Omni 将 Thinker 和 Talker 都升级为 MoE：

- **Thinker**：MoE LLM，更大总参数但激活参数更少 → 更强的理解能力，相近推理速度
- **Talker**：MoE 语音解码器 → 更自然的韵律和情感表达

### 升级2：AuT（Audio Transformer）

![[aut.png|700]]

> 图2：AuT 音频编码器架构

Qwen2.5-Omni 使用 Whisper 作为音频编码器。Qwen3-Omni 替换为 **AuT**：

- 从零训练的 Audio Transformer（而非基于 Whisper 的 encoder-decoder）
- 训练数据：**20M 小时**音频（相比 Whisper 的 680K 小时）
- 原生支持多语言（119 种书面语言，19 种口语）
- 编码质量在嘈杂环境、远场语音、方言口音上显著提升

### 升级3：多码本语音生成

Qwen2.5-Omni 的语音生成使用单码本（single codebook），音质受限。

Qwen3-Omni 升级为 **多码本（Multi-codebook）**：
- 使用多个 VQ 码本分别编码语音的不同频带
- 低频码本：基频、音素结构
- 高频码本：音色细节、情感表达
- 解码器从自回归 Transformer 替换为**轻量因果 ConvNet（Code2Wav）**——更快、更适合流式输出

### 升级4：超长音频和实时流式

- 支持 **40+ 分钟**连续音频输入（vs Qwen2.5-Omni 的几分钟）
- 冷启动端到端延迟 **234ms**（音频输入 → 首个语音输出包）
- 119 种书面语言理解，19 种语言的语音生成

### 升级5：全双工交互增强

相比 Qwen2.5-Omni 的简单能量阈值打断检测，Qwen3-Omni 的打断机制更智能：
- 基于语义的打断判断（不是听到声音就停，而是判断是否在对自己说话）
- 支持**自然停顿**——用户思考时的"嗯..."不会触发打断

## 实验结果

| 基准 | Qwen3-Omni | Qwen2.5-Omni | Gemini 2.0 Flash |
|------|:---:|:---:|:---:|
| SpeechBench | **82.1** | 78.2 | 80.5 |
| AudioCaps | **SOTA** | SOTA | -- |
| SEEDBench-Audio | **79.3** | 76.5 | -- |
| MMLU | **73.5** | 68.3 | -- |

## 深度分析

### Qwen-Omni 系列对比

| 特性 | Qwen2.5-Omni | Qwen3-Omni | Qwen3.5-Omni |
|------|:---:|:---:|:---:|
| Thinker | 7B Dense | MoE | Hybrid Attn MoE |
| Talker | Non-AR Dec | MoE + Code2Wav | Hybrid Attn MoE |
| Audio Enc | Whisper | **AuT (20M hr)** | AuT |
| Max Audio | 几分钟 | **40+ 分钟** | 10+ 小时 |
| 语言 | 中英为主 | **119+19** | 10 语言 |
| 延迟 | ~500ms | **234ms** | ~200ms |
| Codebook | 单码本 | **多码本** | 多码本 |

### 局限性
- Code2Wav 的因果 ConvNet 是轻量设计，极限音质不如扩散模型
- MoE Talker 在极端压缩比下可能出现韵律不稳定
- 19 种口语仍以英语和中文为主，其他语言训练数据不足

## 我的综合评价

### 总体评分：8.5/10

| 维度 | 分数 |
|------|:---:|
| 创新性 | 8/10 |
| 技术质量 | 9/10 |
| 实验充分性 | 8/10 |
| 写作质量 | 8/10 |
| 实用性 | 9/10 |

> [!tip] 关键启示
> 全模态模型的两个关键杠杆：音频编码器质量（AuT > Whisper）和语音生成码本数量（多码本 >> 单码本）。

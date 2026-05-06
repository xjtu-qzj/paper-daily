---
date: "2026-05-05"
paper_id: "arXiv:2605.01913"
title: "RefusalGuard: Geometry-Preserving Fine-Tuning for Safety in LLMs"
authors: "Sadia Asif, Mohammad Mohammadi Amiri"
domain: "Foundation Models & LLM"
tags:
  - 论文笔记
  - Foundation-Models-&-LLM
  - LLM-Safety
  - Fine-Tuning
  - RefusalGuard
  - Representation-Learning
  - Adversarial-Safety
  - Qwen
  - LLaMA
  - Gemma
quality_score: "7.5/10"
created: "2026-05-05"
updated: "2026-05-05"
status: analyzed
---

# RefusalGuard: Geometry-Preserving Fine-Tuning for Safety in LLMs

## 核心信息
- **论文ID**：arXiv:2605.01913
- **作者**：Sadia Asif, Mohammad Mohammadi Amiri
- **机构**：--
- **发布时间**：2026-05-03
- **会议/期刊**：--
- **链接**：[arXiv](http://arxiv.org/abs/2605.01913v1) | [PDF](https://arxiv.org/pdf/2605.01913v1)
- **引用**：--

## 摘要翻译

### 英文摘要
Fine-tuning safety-aligned language models for downstream tasks often leads to substantial degradation of refusal behavior, making models vulnerable to adversarial misuse. While prior work has shown that safety-relevant features are encoded in structured representations within the model's activation space, how these representations change during fine-tuning and why alignment degrades remains poorly understood. In this work, we investigate the representation-level mechanisms underlying alignment degradation. Our analysis shows that standard fine-tuning induces systematic drift in safety-relevant representations, distorts their geometric structure, and introduces interference between task optimization and safety features. These effects collectively lead to increased harmful compliance. Motivated by these findings, we introduce REFUSALGUARD, a representation-level fine-tuning framework that preserves safety-relevant structure during model adaptation. Our approach constrains updates in hidden representation space, ensuring that safety-mediating components remain stable while allowing task-specific learning in complementary directions. We evaluate REFUSALGUARD across multiple model families, including LLaMA, Gemma, and Qwen, on adversarial safety benchmarks such as AdvBench, DirectHarm4, and JailbreakBench, as well as downstream utility tasks. Our approach achieves attack success rates comparable to base safety-aligned models while maintaining competitive task performance, significantly outperforming baselines.

### 中文翻译
对安全对齐语言模型进行下游任务微调通常会导致拒绝行为的严重退化，使模型容易受到对抗性滥用。尽管先前工作表明安全相关特征以结构化表示编码在模型的激活空间中，但这些表示在微调过程中如何变化以及对齐为何退化仍然知之甚少。本文研究了对齐退化背后的表示层面机制。分析表明，标准微调会导致安全相关表示的系统性漂移、扭曲其几何结构，并在任务优化和安全特征之间引入干扰。这些效应共同导致有害合规的增加。基于这些发现，我们提出了 REFUSALGUARD，一个在模型适应过程中保持安全相关结构的表示级微调框架。我们的方法约束隐藏表示空间中的更新，确保安全中介组件保持稳定，同时允许在互补方向上进行任务特定学习。我们在 LLaMA、Gemma 和 Qwen 等多个模型家族上评估 REFUSALGUARD，测试包括 AdvBench、DirectHarm4 和 JailbreakBench 等对抗安全基准，以及下游实用任务。方法达到与基础安全对齐模型相当的攻击成功率，同时保持竞争性的任务性能，显著优于基线。

### 核心要点提炼
- **研究背景**：安全对齐的 LLM 在微调后安全性显著退化，缺乏表示层面的机制理解
- **研究动机**：从表示几何的角度理解和解决安全退化问题
- **核心方法**：分析安全表示的几何变化，提出约束表示更新的 REFUSALGUARD 框架
- **主要结果**：攻击成功率与基础安全对齐模型持平，远超现有安全微调方法
- **研究意义**：首次从表示几何角度系统解释和解决 LLM 微调安全退化

## 研究背景与动机

### 领域现状
LLM 的安全对齐（如 RLHF、DPO）在预训练后有效，但在下游任务微调（SFT）后安全性往往会崩溃。现有解决方案（如安全数据混合、正则化）效果有限，且缺乏对退化机制的深入理解。

### 现有方法的局限性
- 数据层面的方法（混入安全样本）无法从根本上阻止退化
- 缺乏对安全特征在表示空间中如何变化的机制理解
- 现有防御方法在强对抗攻击下效果不佳

### 研究动机
作者发现安全相关特征在激活空间中以特定几何结构存在，标准微调会破坏这种结构。理解并保护这种几何结构是解决安全退化的关键。

## 研究问题

### 核心研究问题
1. 标准微调如何在表示层面导致安全对齐退化？
2. 能否设计一个表示级约束框架，在微调中保持安全几何结构？

## 方法概述

### 核心思想
REFUSALGUARD 的核心思想是：安全相关特征在激活空间中具有特定的几何结构，微调时约束这些表示的更新方向，使其保持在安全区域内，同时允许其他维度自由更新以适应下游任务。

### 方法框架

#### 整体架构

![[mechanistic_panel_harmful.png|800]]

> 图1：安全表示在微调过程中的几何变化机制分析。标准微调导致安全表示的漂移和结构扭曲。

**REFUSALGUARD 的三步分析-干预框架**：

1. **识别安全相关表示**：在基础安全对齐模型的激活空间中定位安全相关维度
2. **约束更新方向**：在微调时限制安全维度的更新幅度和方向
3. **自由任务学习**：允许非安全维度的正常梯度更新

![[02_asr_reduction_vs_lora.png|600]]

> 图2：不同 LoRA rank 下攻击成功率（ASR）的降低效果对比

#### 关键创新
- **表示级约束而非数据级约束**：直接在表示空间施加几何约束，而非修改训练数据
- **选择性更新**：区分安全维度和任务维度，实现安全与效用的解耦
- **理论分析**：提供安全退化机制的表示级理论解释

## 实验结果

### 实验设置
- **模型**：LLaMA、Gemma、Qwen 系列
- **安全基准**：AdvBench、DirectHarm4、JailbreakBench
- **下游任务**：MMLU、HellaSwag 等标准基准
- **基线**：标准 SFT、SafeRLHF、数据混合等方法

### 主要结果

#### 安全性能
- 攻击成功率（ASR）接近基础安全对齐模型，远低于标准 SFT 模型
- 在多个攻击类型（GCG、AutoDAN 等）下均有效

#### 实用性保持
- 下游任务性能与标准 SFT 竞争
- 安全-效用 trade-off 显著优于所有基线

![[09_lambda_tradeoff_panel.png|600]]

> 图3：安全-效用 trade-off 的 λ 参数调节面板

## 深度分析

### 研究价值评估

#### 理论贡献
- **表示级机制分析**：首次系统揭示微调中安全退化的表示几何机制
- **选择性更新理论**：证明安全维度和任务维度可以在表示空间中解耦

#### 实际应用价值
- 可嵌入任何微调流程，无需修改训练数据
- 跨模型家族泛化（LLaMA、Gemma、Qwen）
- 对模型发布者/部署者有直接价值

### 方法优势详解
- **根本性解决**：从表示层面解决问题而非治标
- **高兼容性**：与 LoRA/全量微调均兼容
- **强泛化性**：跨模型家族和攻击类型有效

### 局限性分析
- 需要基础安全对齐模型作为起点（依赖于初始对齐质量）
- 安全维度识别可能需要针对不同模型调整
- 未能完全消除所有越狱攻击

## 我的综合评价

### 价值评分

#### 总体评分
**7.5/10** - 研究视角新颖（表示几何），方法具有理论和实用双重价值

#### 分项评分

| 评分维度 | 分数 | 评分理由 |
|----------|------|----------|
| 创新性 | 8/10 | 首次从表示几何角度解决安全微调退化 |
| 技术质量 | 8/10 | 机制分析严谨，方法设计有理论支撑 |
| 实验充分性 | 7/10 | 多模型、多基准，但消融实验可更丰富 |
| 写作质量 | 7/10 | 逻辑清晰，结构完整 |
| 实用性 | 8/10 | 高实用价值，可嵌入现有流程 |

> [!tip] 关键启示
> 安全对齐的退化源于表示几何的破坏，而非简单的"遗忘"——在表示空间施加几何约束是比修改数据更根本的解决路径。

> [!warning] 注意事项
> - 需要基础安全对齐模型，对未对齐模型无效
> - λ 超参数需针对任务调节
> - 未探索多轮对话的安全性退化

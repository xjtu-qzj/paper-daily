---
date: "2026-05-05"
paper_id: "arXiv:2603.00436"
title: "ROKA: Robust Knowledge Unlearning against Adversaries"
authors: "--"
domain: "unlearning backdoor attack"
tags:
  - 论文笔记
  - unlearning-backdoor-attack
  - Machine-Unlearning
  - Knowledge-Contamination
  - Neural-Healing
  - Backdoor-Attack
  - Indirect-Attack
  - Robust-Unlearning
quality_score: "8.0/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# ROKA: Robust Knowledge Unlearning against Adversaries

## 核心信息
- **论文ID**：arXiv:2603.00436
- **作者**：--
- **机构**：--
- **发布时间**：2026-02-28
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2603.00436) | [Semantic Scholar](https://www.semanticscholar.org/paper/e323001f48903d3163a77dc73a1eb69227c97e69)
- **引用**：--

## 摘要翻译

### 英文摘要
The need for machine unlearning is critical for data privacy, yet existing methods often cause Knowledge Contamination by unintentionally damaging related knowledge. Such a degraded model performance after unlearning has been recently leveraged for new inference and backdoor attacks. Most studies design adversarial unlearning requests that require poisoning or duplicating training data. In this study, we introduce a new unlearning-induced attack model, namely indirect unlearning attack, which does not require data manipulation but exploits the consequence of knowledge contamination to perturb the model accuracy on security-critical predictions. To mitigate this attack, we introduce a theoretical framework that models neural networks as Neural Knowledge Systems. Based on this, we propose ROKA, a robust unlearning strategy centered on Neural Healing. Unlike conventional unlearning methods that only destroy information, ROKA constructively rebalances the model by nullifying the influence of forgotten data while strengthening its conceptual neighbors. To the best of our knowledge, our work is the first to provide a theoretical guarantee for knowledge preservation during unlearning. Evaluations on various large models, including vision transformers, multi-modal models, and large language models, show that ROKA effectively unlearns targets while preserving, or even enhancing, the accuracy of retained data, thereby mitigating the indirect unlearning attacks.

### 中文翻译
机器遗忘对于数据隐私至关重要，但现有方法常常因意外损害相关知识而导致知识污染。遗忘后模型性能的退化最近被利用于新的推理和后门攻击。大多数研究设计需要中毒或复制训练数据的对抗性遗忘请求。本文引入了一种新的遗忘诱导攻击模型——间接遗忘攻击，它不需要数据操纵，而是利用知识污染的后果来扰动模型在安全关键预测上的准确性。为缓解此攻击，我们引入了一个将神经网络建模为神经知识系统的理论框架。基于此，我们提出 ROKA，一种以神经修复为核心的鲁棒遗忘策略。与仅销毁信息的传统遗忘方法不同，ROKA 通过消除被遗忘数据的影响同时强化其概念邻居，建设性地重新平衡模型。据我们所知，这是首次为遗忘过程中的知识保持提供理论保证的工作。在视觉 Transformer、多模态模型和 LLM 上的评估表明，ROKA 能有效遗忘目标，同时保持甚至增强保留数据的准确性，从而缓解间接遗忘攻击。

### 核心要点提炼
- **研究背景**：机器遗忘过程中会产生知识污染（Knowledge Contamination），破坏相关但不该被遗忘的知识
- **研究动机**：现有攻击利用遗忘后的性能退化进行推理/后门攻击，但缺乏理论理解和鲁棒防御
- **核心方法**：Neural Knowledge Systems 理论框架 + Neural Healing（神经修复）策略
- **主要结果**：首个提供遗忘中知识保持理论保证的工作；在 ViT、多模态、LLM 上验证
- **研究意义**：将遗忘从"破坏"范式升级为"修复-重平衡"范式

## 研究背景与动机

### 领域现状
机器遗忘（Machine Unlearning）旨在从已训练模型中移除特定数据的影响。现有方法（如梯度上升、Fisher 遗忘、影响函数）的核心思路是"破坏"——直接消除目标数据对模型参数的影响。

### 知识污染问题
传统方法的问题在于，遗忘操作不仅移除了目标知识，还会波及其"概念邻居"（conceptual neighbors）。例如：
- 遗忘"飞机"类别可能损害"鸟类"分类能力（共享翅膀、天空等特征）
- 遗忘某人的数据可能损害同人群其他人的模型表现

### 间接遗忘攻击
攻击者不需要访问训练数据，只需利用知识污染的连锁反应：
- 发起一个看似合法的遗忘请求
- 利用遗忘造成的相关概念退化
- 在安全关键预测上造成模型失效

## 研究问题

### 核心研究问题
1. 如何在遗忘特定数据的同时保证相关知识不被污染？
2. 能否为遗忘过程中的知识保持提供理论保证？
3. 如何防御利用知识污染的间接遗忘攻击？

## 方法概述

### 核心思想
ROKA 的核心洞见是：遗忘不应该是"删除"操作，而应该是"重新平衡"操作。Neural Healing（神经修复）在消除目标数据影响的同时，主动强化其概念邻居，维持整体知识结构的完整性。

### 方法框架

#### 整体架构

![[threat_model_A.png|800]]

> 图1：威胁模型A — 展示了间接遗忘攻击的工作机制，攻击者通过合法遗忘请求触发知识污染链式反应

**ROKA 三步框架**：

| 步骤 | 操作 | 目的 |
|------|------|------|
| 1. 神经知识建模 | 将神经网络参数化为知识图谱结构 | 识别概念间的依赖关系 |
| 2. 影响量化 | 计算遗忘目标对每个概念邻居的影响 | 精确定位污染风险 |
| 3. 神经修复 | 消除目标影响 + 强化邻居概念 | 建设性重平衡 |

![[threat_model_B.png|600]]

> 图2：威胁模型B — ROKA 的防御策略

#### 关键创新

**Neural Knowledge Systems 理论**：
- 将神经网络形式化为知识系统，其中神经元/参数组对应概念
- 知识污染 = 概念间依赖关系的非预期断裂
- 提供可证明的知识保持界

**Neural Healing 策略**：

$$\theta_{new} = \theta - \eta \cdot \nabla_\theta L_{forget} + \lambda \cdot \nabla_\theta L_{reinforce}$$

其中 $L_{forget}$ 消除目标数据影响，$L_{reinforce}$ 强化概念邻居。

![[threat_model_C.png|600]]

> 图3：威胁模型C — 完整攻防框架

## 实验结果

### 实验设置
- **模型**：Vision Transformer (ViT)、多模态模型、LLM
- **数据集**：CIFAR-100、Tiny-ImageNet 等
- **遗忘任务**：类别遗忘（class-wise unlearning）
- **评估指标**：遗忘成功率、保留类别准确率、遗忘后攻击防御率

### 主要结果

#### 遗忘效果与知识保持

![[stability_CIFAR100.png|600]]

> 图4：CIFAR-100 上 ROKA 的稳定性表现

![[stability_Tiny-Imagenet.png|600]]

> 图5：Tiny-ImageNet 上 ROKA 的稳定性表现

![[contribution_reallocation_1__2.png|600]]

> 图6：贡献重新分配 — Neural Healing 将遗忘目标的贡献转移到概念邻居

**核心发现**：
- ROKA 在成功遗忘目标的同时，**保持甚至增强**了保留类别的准确率
- 传统方法（梯度上升、Fisher）遗忘后保留准确率显著下降
- ROKA 有效防御间接遗忘攻击

## 深度分析

### 研究价值评估

#### 理论贡献（8/10）
- **首个知识保持理论保证**：此前的方法只有经验性结果
- Neural Knowledge Systems 框架通用性强，可应用于多种遗忘场景
- 概念邻居的识别和强化机制有理论支撑

#### 实际应用价值（7/10）
- 对需要合规遗忘（GDPR 被遗忘权）的场景有直接价值
- 适用于 ViT、多模态、LLM 等多种模型
- 计算开销相对传统方法增加不多

### 方法优势详解
- **正向修复而非负向破坏**：范式转变，从"删除"到"重平衡"
- **理论保证**：不是启发式方法
- **防御能力**：被动防御间接遗忘攻击

### 局限性分析
- 概念邻居的识别依赖一定的先验知识
- 在极端遗忘比例下（如遗忘 50% 以上类别）效果可能下降
- LLM 上的评估相对有限

## 我的综合评价

### 价值评分

#### 总体评分
**8.0/10** — 遗忘领域的理论突破，首次提供知识保持保证

#### 分项评分

| 评分维度 | 分数 | 评分理由 |
|----------|------|----------|
| 创新性 | 9/10 | 范式转变：从破坏到修复，理论创新强 |
| 技术质量 | 8/10 | 理论框架严谨，方法合理 |
| 实验充分性 | 7/10 | 多模型验证，但 LLM 实验可更丰富 |
| 写作质量 | 7/10 | 技术内容清晰 |
| 实用性 | 7/10 | 合规场景直接可用 |

> [!tip] 关键启示
> 遗忘不是删除，而是重新平衡。保护好概念邻居，才能真正安全地遗忘。

> [!warning] 注意事项
> - 概念邻居识别需要领域知识
> - 理论保证在极端遗忘比例下可能松弛
> - 需要在实际合规场景（GDPR）中进一步验证

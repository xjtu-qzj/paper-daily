---
date: "2026-05-05"
paper_id: "arXiv:2510.13322"
title: "Injection, Attack and Erasure: Revocable Backdoor Attacks via Machine Unlearning"
authors: "--"
domain: "unlearning backdoor attack"
tags:
  - 论文笔记
  - unlearning-backdoor-attack
  - Revocable-Backdoor
  - Bilevel-Optimization
  - PCGrad
  - Trigger-Optimization
  - Backdoor-Attack
  - Machine-Unlearning
quality_score: "7.5/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Injection, Attack and Erasure: Revocable Backdoor Attacks via Machine Unlearning

## 核心信息
- **论文ID**：arXiv:2510.13322
- **作者**：--
- **机构**：--
- **发布时间**：2025-10-15
- **会议/期刊**：--
- **链接**：[arXiv](https://arxiv.org/abs/2510.13322)
- **引用**：--

## 摘要翻译

### 英文摘要
Backdoor attacks pose a persistent security risk to deep neural networks (DNNs) due to their stealth and durability. While recent research has explored leveraging model unlearning mechanisms to enhance backdoor concealment, existing attack strategies still leave persistent traces that may be detected through static analysis. In this work, we introduce the first paradigm of revocable backdoor attacks, where the backdoor can be proactively and thoroughly removed after the attack objective is achieved. We formulate the trigger optimization in revocable backdoor attacks as a bilevel optimization problem: by simulating both backdoor injection and unlearning processes, the trigger generator is optimized to achieve a high attack success rate (ASR) while ensuring that the backdoor can be easily erased through unlearning. To mitigate the optimization conflict between injection and removal objectives, we employ a deterministic partition of poisoning and unlearning samples to reduce sampling-induced variance, and further apply the Projected Conflicting Gradient (PCGrad) technique to resolve the remaining gradient conflicts. Experiments on CIFAR-10 and ImageNet demonstrate that our method maintains ASR comparable to state-of-the-art backdoor attacks, while enabling effective removal of backdoor behavior after unlearning. This work opens a new direction for backdoor attack research and presents new challenges for the security of machine learning systems.

### 中文翻译
后门攻击因其隐蔽性和持久性对深度神经网络构成持续安全风险。尽管近期研究探索了利用模型遗忘机制增强后门隐藏，但现有攻击策略仍留下可被静态分析检测的持久痕迹。本文首次引入可撤销后门攻击范式——后门可在攻击目标达成后被主动、彻底地移除。我们将可撤销后门攻击中的触发器优化建模为双层优化问题：通过同时模拟后门注入和遗忘过程，优化触发器生成器在保持高攻击成功率的同时确保后门可被轻易遗忘。为缓解注入和移除目标之间的优化冲突，我们使用中毒和遗忘样本的确定性划分来减少采样方差，并进一步应用投影冲突梯度（PCGrad）技术解决剩余的梯度冲突。CIFAR-10 和 ImageNet 上的实验表明，该方法保持了与 SOTA 后门攻击相当的 ASR，同时能在遗忘后有效移除后门行为。本工作为后门攻击研究开辟了新方向，对机器学习系统安全提出了新挑战。

### 核心要点提炼
- **研究背景**：现有后门攻击在模型中留下持久痕迹，可被静态分析和防御检测
- **研究动机**：设计一种"用后即弃"的后门——攻击后可自动清除，不留下法理痕迹
- **核心方法**：双层优化（同时优化 ASR 和可遗忘性）+ PCGrad 解决梯度冲突
- **主要结果**：ASR 与 SOTA 后门攻击相当，遗忘后后门行为被有效清除
- **研究意义**：开创可撤销后门攻击新范式，对 ML 安全防御提出新挑战

## 研究背景与动机

### 领域现状
传统后门攻击的核心目标是**持久性**——后门一旦植入，应难以被检测和移除。但持久性是一把双刃剑：
- 攻击者同样无法主动清除后门
- 后门痕迹可能被静态分析捕获
- 模型被多个攻击者利用后，互相干扰

### 为什么需要可撤销后门？
- **操作安全**：攻击完成后不留证据
- **独占性**：攻击者可以"锁住"后门，防止他人利用
- **隐蔽性升级**：可遗忘 = 更难被取证

### 核心挑战
攻击成功率和可遗忘性是矛盾的目标：
- 更好的攻击 → 后门更深地嵌入模型 → 更难遗忘
- 更容易遗忘 → 后门不够牢固 → 攻击容易失败

## 研究问题

### 核心研究问题
1. 能否设计一个既能高效攻击、又能被轻易遗忘的后门？
2. 如何在优化中平衡注入和移除这两个冲突目标？
3. 可遗忘后门能否通过现有的后门防御检测？

## 方法概述

### 核心思想
将后门注入和遗忘作为**联合优化问题**——不是在事后考虑如何移除后门，而是在设计后门时就内置"自毁开关"。

### 方法框架

#### 整体架构

![[framework_page1.png|800]]

> 图1：可撤销后门攻击框架 — 双层优化同时模拟注入和遗忘

**双层优化公式**：

**上层（攻击目标）**：
$$\max_{trigger} \ ASR(model(trigger), target)$$

**下层（遗忘能力）**：
$$\min_{model} \ L_{unlearn}(model, poisoned\_data)$$

**约束**：遗忘后 ASR → 0，清洁准确率保持不变

![[motivation_page1.png|600]]

> 图2：研究动机 — 常规后门 vs 可撤销后门的攻击-遗忘曲线对比

#### 解决优化冲突

**问题**：攻击目标（最大化 ASR）与遗忘目标（最小化遗忘后 ASR）梯度方向冲突

**方案1 — 确定性样本划分**：
- 将中毒样本固定分为两组：注入组和遗忘组
- 减少随机采样导致的梯度方差

**方案2 — PCGrad（投影冲突梯度）**：
- 检测两个目标梯度的冲突方向
- 将冲突梯度投影到彼此的正交空间

$$g_{proj} = g_{inject} - \frac{g_{inject} \cdot g_{unlearn}}{||g_{unlearn}||^2} \cdot g_{unlearn}$$

![[First-Order_page1.png|600]]

> 图3：First-Order 方法 — 使用一阶近似的双层优化

![[UnrollSGD_page1.png|600]]

> 图4：UnrollSGD 方法 — 展开 SGD 步骤以获得更精确的梯度估计

## 实验结果

### 实验设置
- **数据集**：CIFAR-10、ImageNet-10
- **攻击类型**：BadNets、Blended、WaNet 等 SOTA 方法作为对比
- **评估指标**：ASR（攻击成功率）、清洁准确率（CA）、遗忘后 ASR（Post-Unlearning ASR）

### 主要结果

#### 攻击性能

![[cifar10_clean_page1.png|600]]

> 图5：CIFAR-10 清洁准确率保持

![[cifar10_first_order_page1.png|600]]

> 图6：CIFAR-10 First-Order 方法的 ASR 和遗忘后 ASR

#### 可遗忘性验证

![[output_cifar10_first_order_page1.png|600]]

> 图7：CIFAR-10 First-Order 输出可视化

![[output_imagenet10_unrollsgd_page1.png|600]]

> 图8：ImageNet-10 UnrollSGD 输出可视化

#### 抗防御能力

![[CIFAR10_First-Order_STRIP_page1.png|600]]

> 图9：STRIP 防御检测 — 可撤销后门在 STRIP 检测下的表现

**核心结果**：
- ASR 与 SOTA 后门攻击（BadNets、WaNet）**持平或略优**
- 遗忘后 ASR 降至接近 0%，清洁准确率完全恢复
- 现有防御方法难以检测可撤销后门（因注入和遗忘模式交替）

## 深度分析

### 研究价值评估

#### 理论贡献（7/10）
- 首次将"可撤销性"引入后门攻击设计空间
- 双层优化 + PCGrad 解决攻防目标冲突
- 但 PCGrad 本身是已有技术

#### 实际应用价值（6/10）
- 攻击视角：为高级持续性威胁（APT）提供新工具
- 防御视角：提醒防御方关注"可撤销后门"这一新攻击面
- 目前仅在图像分类上验证

### 方法优势详解
- **新颖范式**：可撤销后门为全新攻击类别
- **优雅的优化框架**：双层优化自然捕获注入-遗忘的权衡
- **与现有后门兼容**：方法可与多种触发器类型结合

### 局限性分析
- 遗忘需要已知的中毒样本（需攻击者保存）
- PCGrad 增加训练复杂度
- 未在更大模型（LLM、ViT）上验证
- "可撤销"本身可能成为防御方的检测特征

## 我的综合评价

### 价值评分

#### 总体评分
**7.5/10** — 开辟新攻击范式，优化框架优雅，但验证范围有限

#### 分项评分

| 评分维度 | 分数 | 评分理由 |
|----------|------|----------|
| 创新性 | 8/10 | 可撤销后门全新范式 |
| 技术质量 | 8/10 | 双层优化 + PCGrad 设计合理 |
| 实验充分性 | 6/10 | 仅图像分类，未扩展到 LLM/VLM |
| 写作质量 | 7/10 | 动机清晰，技术细节完整 |
| 实用性 | 6/10 | 攻击工具价值高，防御启发有限 |

> [!tip] 关键启示
> 后门不一定是永久的——"用后即弃"的后门既是攻击者的新武器，也是防御研究必须面对的新挑战。

> [!warning] 注意事项
> - 可撤销性可能成为检测后门的新信号
> - 仅在中小规模图像任务上验证
> - 需要攻击者保存中毒样本用于遗忘

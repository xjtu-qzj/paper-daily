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

### 前置知识：LRP 与 Input×Gradient

ROKA 的贡献量化依赖 **Layer-wise Relevance Propagation (LRP)** 的简化形式——**Input×Gradient** 方法。核心思想：将模型输出对某个输入的敏感度反向传播到隐藏层，得到每个神经元对该输出的"贡献分数"。

对于输入 $x$ 和目标类别 $c$，隐藏层神经元 $h_i$ 的贡献定义为：

$$R(h_i) = h_i \cdot \frac{\partial f_c(x)}{\partial h_i}$$

即神经元激活值与其对目标输出梯度的乘积。这回答了"哪些神经元对当前预测贡献最大"的问题。ROKA 使用此方法计算 batch 中每个样本的贡献向量，作为识别"概念邻居"的依据。

### 理论框架：Neural Knowledge System

ROKA 将深度神经网络形式化为一个**层级知识系统** $\mathcal{S}$：

$$\mathcal{S} = (\mathbf{X}, \mathbf{K}, \mathcal{F})$$

- $\mathbf{X}$：原始输入空间
- $\mathbf{K}$：内部抽象知识表示空间，按层次组织为 $(\mathbf{K}_0, \mathbf{K}_1, ..., \mathbf{K}_L)$，类似 Markov 链 $\mathbf{K}_{l-1} \to \mathbf{K}_l$
- $\mathcal{F}$：变换函数集，包括 Encode $\mathcal{E}: \mathbf{X} \to \mathbf{K}$ 和 Decode $\mathcal{D}: \mathbf{K} \to \mathbf{Y}$

#### 静态属性：Contribution（贡献度）

低层组件 $k_{l-1,i}$ 对高层状态 $\mathbf{K}_{l,j}$ 的**贡献度**即其条件概率：

$$C(k_{l-1,i} \to \mathbf{K}_{l,j}) = \frac{w_{l-1,i}}{W_{l,j}} = p(k_{l-1,i}|\mathbf{K}_{l,j})$$

其中 $w$ 为组件权重，$W$ 为父组总权重。

#### 动态属性：Leverage（杠杆率）

**Influence**（影响力）衡量整个系统熵对单个组件变化的敏感度：

$$\text{Influence}(w_i \to \mathbf{K}_l) = -\log(W_{dest})$$

**Leverage** 是影响力按其自身权重缩放后的值，量化了"小变化引发大效应"的潜力：

$$\text{Leverage}(w_i \to \mathbf{K}_l) = \frac{-\log(W_{dest})}{w_i}$$

Leverage 在低层基础组件上最高，随聚合层级升高而递减。这解释了为什么破坏底层共享参数会通过层级链放大，导致高层知识的灾难性崩塌。

#### 知识破坏的边界条件

**知识破坏（Knowledge Destruction）** 指低层组件的小扰动经高 Leverage 放大后导致高层知识集的剧烈变化。其边界由 KL 散度界定：

$$D_{KL}(P(\mathbf{Y}|\mathbf{K'}_l) \ || \ P(\mathbf{Y}|\mathbf{K}_l)) \ \propto \ \text{Leverage}(w_{l-1,i} \to \mathbf{K}_l) \cdot \left| \frac{\Delta w_{l-1,i}}{w_{l-1,i}} \right|$$

当此值超过阈值 $\tau$ 时，知识破坏发生。**知识污染（Knowledge Contamination）** 则是遗忘更新 $\Delta w_{shared}$ 跨越了保留知识的破坏边界：

$$C \cdot \text{Leverage}(w_{shared} \to K_{retain}) \cdot \left| \frac{\Delta w_{shared}}{w_{shared}} \right| > \tau_{retain}$$

这个不等式揭示了遗忘的根本困境：一个足以遗忘目标概念的更新，如果作用于共享的高 Leverage 参数，就会污染相关概念。

### Neural Healing：贡献重分配

![[contribution_reallocation_1__2.png|700]]

> 图：贡献重分配过程 — 消除遗忘目标的贡献，按比例重分配给 sibling 组件

ROKA 的核心策略是**贡献重分配（Contribution Re-allocation）**——不是简单地删除目标知识，而是将被遗忘组件的权重按比例转移给其"兄弟姐妹"组件，保证父级知识结构的总权重守恒。

**三步过程**：

**1. Nullification（置零）**：消除 $k_{l-1,i}$ 的贡献，产生权重赤字 $w_{l-1,i}$。

**2. Identification of Siblings（兄弟识别）**：找到同层中与 $k_{l-1,i}$ 结构相关的所有组件集合 $\mathbf{S}_i$（通常共享同一父聚合节点）。

**3. Proportional Re-allocation（按比重分配）**：权重赤字按各 sibling 的原贡献比例重新分配：

$$W_{siblings} = \sum_{k_{l-1, k} \in \mathbf{S}_i} w_{l-1, k}$$

$$\Delta w_j = w_{l-1,i} \cdot \frac{w_{l-1,j}}{W_{siblings}}$$

$$w'_{l-1, j} = w_{l-1,j} + \Delta w_j$$

**理论保证**：该过程**保持任意两个 sibling 之间的贡献比率不变**（Sibling Knowledge Preservation theorem），因此整体知识结构不会因遗忘而扭曲。这是首个为遗忘提供知识保持理论保证的方法。

### 实际实现：随机化神经修复（Stochastic Unlearning with Neural Healing）

纯理论的重分配在真实神经网络中不可行——因为单个数据点的知识分散在数百万参数中，且精确计算每个参数的贡献成本极高（接近重新训练的代价）。因此 ROKA 提出了一种**迭代随机化近似**。

#### 变体1：有目标遗忘（Targeted Stochastic Unlearning）

适用于遗忘目标是**明确类别标签**的场景（如"遗忘飞机类别"）。

![[threat_model_C.png|700]]

> 图：ROKA 完整有目标遗忘流程

**算法流程**（详见 Algorithm 1）：

```
输入：模型 M, 数据集 D, 目标标签 l_target, 迭代次数 N, 修复因子 α, 学习率 η
输出：遗忘后模型 M'

For i = 1 to N:
    1. 从 D 中采样一个 batch B
    
    2. 选择遗忘样本 x_forget：
       - 计算 B 中每个样本被预测为 l_target 的概率
       - 按概率加权采样（模型越"确信"是目标类的样本越可能被选中）
       → 保证遗忘努力始终集中在最具代表性的样本上
    
    3. 识别兄弟样本 B_siblings：
       - 用 Input×Gradient 计算 B 中每个样本的贡献向量 R
       - 取 r_forget = R[x_forget]
       - 在贡献空间中找 x_forget 的 k 近邻 → B_siblings
    
    4. 计算复合损失：
       L_forget = CrossEntropy(M(x_forget), l_target)
       y_pred = argmax(M(B_siblings))        # 模型对兄弟样本的当前预测
       L_heal = CrossEntropy(M(B_siblings), y_pred)   # 自我蒸馏
       L_unlearn = L_forget - α · L_heal
    
    5. 梯度上升：θ = θ + η · ∇_θ L_unlearn
       （等价于梯度下降 -L_unlearn）
```

**损失函数设计**是 ROKA 最核心的技术：

- **$L_{forget}$**：最大化遗忘样本对目标标签的损失 → 推动模型"忘记"
- **$L_{heal}$**：最小化兄弟样本对自身预测的损失（自我蒸馏）→ 固化模型的原有行为
- **$-\alpha \cdot L_{heal}$**：$\alpha$ 控制修复强度。足够大的 $\alpha$ 确保保留损失下降

单个更新步骤同时执行遗忘和修复——梯度上升方向推远目标概念，同时拉近兄弟概念。这就是"贡献重分配"在优化空间中的具体实现。

**如何理解"修复"机制**：假设遗忘 $l_{\text{target}}$=飞机。$x_{\text{forget}}$ 是一张被模型高置信度判为飞机的图。B_siblings 包含贡献向量相似的样本（可能是鸟类——共享"翅膀、天空"特征）。$L_{\text{heal}}$ 告诉模型："这些样本的预测不要改变"，从而保护鸟类分类不受遗忘飞机的连锁影响。

#### 变体2：无目标遗忘（Non-Targeted Stochastic Unlearning）

适用于遗忘目标**未明确标注**的场景（如"删除这组用户数据，但不知道哪些类别受影响"）。

与变体1的关键差异：
- **预计算阶段**：用初始模型为整个 $D_{forget}$ 生成伪标签 $Y_{pseudo}$ 和贡献质心 $r_{centroid}$（固定不变，防止漂移）
- **遗忘样本选择**：每轮选 batch 中贡献向量与 $r_{centroid}$ 余弦相似度最高的样本
- **兄弟识别**：同变体1，在贡献空间中找近邻
- **损失函数**：同变体1，但使用伪标签而非真实标签

伪标签和质心的**预计算-固定**策略是本变体的关键设计——如果每轮重新计算，遗忘目标会"漂移"，导致优化震荡。

#### 理论保证

ROKA 提供了 **Unlearning Trajectory Guarantee**：在每次参数更新中，只要 $\alpha$ 足够大，$L_{\text{forget}}$ 的期望增加（遗忘推进），$L_{\text{heal}}$ 的期望减少（知识保持）。这保证了模型参数始终沿着"遗忘目标 + 保护邻居"的方向移动。

此外，有目标遗忘的**收敛速度远快于**无目标遗忘。原因：无目标场景中遗忘样本和兄弟样本来自同一分布，梯度高度对齐（$E[\cos(g_f, g_h)]$ 高），每次更新的有效幅度被削弱。有目标场景中二者来自不同类别，梯度近似正交，更新更高效。

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

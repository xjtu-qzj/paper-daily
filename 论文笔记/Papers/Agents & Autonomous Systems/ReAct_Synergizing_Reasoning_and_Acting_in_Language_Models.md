---
date: "2026-05-07"
paper_id: "arXiv:2210.03629"
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors: "Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao"
domain: "Agents & Autonomous Systems"
tags:
  - 论文笔记
  - Agents-&-Autonomous-Systems
  - ReAct
  - Reasoning
  - Acting
  - Prompting
  - Chain-of-Thought
  - Few-Shot-Learning
  - ICLR-2023
quality_score: "9.0/10"
created: "2026-05-07"
updated: "2026-05-07"
status: analyzed
---

# ReAct: Synergizing Reasoning and Acting in Language Models

## 核心信息
- **论文ID**：arXiv:2210.03629
- **作者**：Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao
- **机构**：Princeton University, Google Research (Brain team)
- **发布时间**：2022-10（v3: 2023-03）
- **会议/期刊**：ICLR 2023
- **链接**：[arXiv](https://arxiv.org/abs/2210.03629) | [PDF](https://arxiv.org/pdf/2210.03629)
- **项目主页**：https://react-lm.github.io/

## 摘要翻译

### 英文摘要
While large language models (LLMs) have demonstrated impressive performance across tasks in language understanding and interactive decision making, their abilities for reasoning (e.g. chain-of-thought prompting) and acting (e.g. action plan generation) have primarily been studied as separate topics. In this paper, we explore the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner, allowing for greater synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with and gather additional information from external sources such as knowledge bases or environments. We apply our approach, named ReAct, to a diverse set of language and decision making tasks and demonstrate its effectiveness over state-of-the-art baselines in addition to improved human interpretability and trustworthiness.

### 中文翻译
尽管大语言模型（LLMs）在语言理解和交互式决策等任务中展现了令人印象深刻的性能，但其推理能力（如思维链提示）和行动能力（如动作计划生成）一直被作为独立主题研究。本文探索了一种让 LLMs 以交错方式生成推理轨迹和任务特定动作的方法，实现两者之间的更大协同：推理轨迹帮助模型归纳、跟踪和更新动作计划，以及处理异常情况；而动作则使其能够与外部知识库或环境等外部源进行交互并获取额外信息。我们将此方法命名为 ReAct，并在多种语言和决策任务上验证了其有效性，在性能上超越了最先进的基线方法，同时提升了人类可解释性和可信度。

### 核心要点提炼
- **研究背景**：LLMs 的推理（CoT）和行动能力一直被分开研究，但人类智能的特征是将两者无缝结合
- **核心方法**：让 LLMs 在同一个轨迹中交替生成推理（Thought）和动作（Action），实现推理-行动协同
- **主要结果**：在 HotpotQA、Fever、ALFWorld、WebShop 等任务上超越纯推理或纯行动的基线
- **研究意义**：开创了 LLM Agent 推理-行动协同范式，为后续 Agent 研究奠定基础

## 研究背景与动机

### 领域现状
2022 年，LLMs 展现了两种关键能力：
1. **推理能力**：通过 Chain-of-Thought (CoT) prompting，LLMs 能够生成推理链来解决算术、常识和符号推理任务。但 CoT 是"静态黑盒"——模型使用内部表征生成思维，没有与外部世界交互，容易出现事实幻觉和错误传播。

2. **行动能力**：LLMs 被用于交互式环境中的规划和行动（如 WebGPT、SayCan），但这些方法没有显式建模推理过程，依赖昂贵的人类反馈进行强化学习。

### 现有方法的局限性
- **CoT 的问题**：纯推理容易产生幻觉事实，推理过程不基于外部信息，错误会逐步传播
- **Act-only 的问题**：纯行动缺乏高层次的目标分解和规划能力，难以处理复杂任务中的异常情况
- **两者割裂**：推理和行动一直被分开研究，没有探索它们的协同效应

### 研究动机
人类智能的独特特征是在任务导向行动和语言推理之间无缝切换。例如在厨房做饭时：
- 在两个具体动作之间进行推理来跟踪进度
- 根据情况处理异常或调整计划
- 当需要外部信息时采取行动（查食谱、查冰箱）

这种"行动"和"推理"的紧密协同使人类能够快速学习新任务，即使在前所未见的情况下也能进行稳健的决策。

## 研究问题

1. 如何让 LLMs 在同一个任务解决轨迹中同时生成推理轨迹和行动？
2. 推理和行动的协同是否能带来比单独推理或行动更好的性能？
3. 这种协同如何提升模型的可解释性和可信度？

## 方法概述

### 核心思想：扩展动作空间

ReAct 的核心思想是**扩展 LLM 的动作空间**：

$$\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$$

其中：
- $\mathcal{A}$ 是传统的任务特定动作空间（如搜索、点击、购买）
- $\mathcal{L}$ 是语言空间（即"思维"或"推理轨迹"）

在这个扩展的动作空间中：
- **动作 $a_t \in \mathcal{A}$**：与外部环境交互，产生观察反馈
- **思维 $\hat{a}_t \in \mathcal{L}$**：不直接影响外部环境，而是通过推理更新上下文，支持未来的推理或行动

### 推理轨迹的作用

思维（Thought）可以有多种形式：
1. **分解任务目标**：创建行动计划
2. **注入常识知识**：与任务解决相关的背景知识
3. **提取重要信息**：从观察中提取关键部分
4. **跟踪进度**：记录当前状态和已完成的子目标
5. **处理异常**：调整行动计划
6. **指导搜索重构**：当搜索失败时尝试其他查询

### 两种推理-行动模式

根据任务类型，ReAct 采用两种不同的模式：

**模式 1：密集思维（用于知识密集型推理任务）**
```
Thought 1 → Action 1 → Observation 1 → Thought 2 → Action 2 → Observation 2 → ...
```
- 推理和行动交替进行
- 适用于 HotpotQA、Fever 等需要多跳推理的任务

**模式 2：稀疏思维（用于决策任务）**
```
Action 1 → Observation 1 → Thought 1 → Action 2 → Observation 2 → Action 3 → Thought 2 → ...
```
- 思维异步出现，由 LLM 自行决定何时需要推理
- 适用于 ALFWorld、WebShop 等长序列决策任务

### 算法流程

**输入**：任务描述（问题/指令）
**输出**：任务解决方案（答案/完成动作）

**步骤 1：初始化**
- 准备少量上下文示例（1-6 个），展示推理-行动-观察的轨迹格式

**步骤 2：生成轨迹**
对于每个时间步 $t$：
1. LLM 接收上下文 $c_t = (o_1, a_1, \hat{a}_1, \ldots, o_{t-1}, a_{t-1}, \hat{a}_{t-1}, o_t)$
2. 根据策略 $\pi(\hat{a}_t | c_t)$ 或 $\pi(a_t | c_t)$ 生成思维或动作
3. 如果是动作 $a_t$，执行并获得观察 $o_{t+1}$
4. 如果是思维 $\hat{a}_t$，更新上下文 $c_{t+1} = (c_t, \hat{a}_t)$

**步骤 3：终止**
- 当生成终止动作（如 `finish[answer]`）或达到最大步数时停止

### 方法关键创新点

1. **推理-行动协同**：首次在 LLM 中将推理和行动整合到同一个轨迹中，实现双向增强
   - 推理帮助行动：思维帮助模型规划、跟踪进度、处理异常
   - 行动帮助推理：外部观察提供真实信息，减少幻觉

2. **灵活的思维-行动格式**：
   - 对于推理密集任务：交替生成思维和行动
   - 对于决策任务：让 LLM 自行决定何时需要思维
   - 这种灵活性使 ReAct 能适应不同类型的任务

3. **人类可解释性**：
   - 推理轨迹是自然语言，人类可以直接理解模型的决策过程
   - 可以区分来自模型内部知识 vs 外部环境的信息
   - 支持人类通过编辑思维来控制或纠正代理行为

4. **通用性和灵活性**：
   - 适用于 QA、事实验证、文本游戏、网页导航等不同任务
   - 设计简单，只需在动作空间中添加语言空间
   - 无需复杂的格式设计或示例选择

## 实验结果

### 实验目标
验证 ReAct 在知识密集型推理任务和交互式决策任务上的有效性，以及推理-行动协同的必要性。

### 数据集

| 数据集 | 任务类型 | 评估指标 | 说明 |
|--------|----------|----------|------|
| HotpotQA | 多跳问答 | Exact Match (EM) | 需要推理多个 Wikipedia 段落 |
| Fever | 事实验证 | Accuracy | 判断声明是否被 Wikipedia 支持/反驳 |
| ALFWorld | 文本游戏 | 成功率 | 模拟家庭环境中的物品操作任务 |
| WebShop | 网页导航 | 成功率/得分 | 在线购物网站的产品选择任务 |

### 实验设置

#### 基线方法
- **Standard Prompting**：标准提示，无推理或行动
- **CoT (Chain-of-Thought)**：纯推理基线
- **CoT-SC**：带自一致性的 CoT（采样 21 次，多数投票）
- **Act**：纯行动基线（无思维）
- **BUTLER**：模仿学习方法（ALFWorld）
- **IL / IL+RL**：模仿学习 + 强化学习（WebShop）

#### 评估环境
- 基础模型：PaLM-540B
- 提示示例：1-6 个上下文示例
- 解码策略：贪心解码

### 主要结果

#### 知识密集型推理任务（HotpotQA & Fever）

| 方法 | HotpotQA (EM) | Fever (Acc) |
|------|---------------|-------------|
| Standard | 28.7 | 57.1 |
| CoT | 29.4 | 56.3 |
| CoT-SC | 33.4 | 60.4 |
| Act | 25.7 | 58.9 |
| **ReAct** | **27.4** | **60.9** |
| CoT-SC → ReAct | -- | 64.6 |
| ReAct → CoT-SC | 35.1 | 62.0 |

**关键发现**：
1. ReAct 在 Fever 上超越 CoT（60.9 vs 56.3），在 HotpotQA 上略低于 CoT
2. ReAct + CoT-SC 组合效果最佳，结合内部知识和外部知识
3. ReAct 显著减少幻觉问题（6% vs 14% 的假阳性率）

#### 交互式决策任务（ALFWorld & WebShop）

**ALFWorld 结果**：

| 方法 | Pick | Clean | Heat | Cool | Look | Pick2 | All |
|------|------|-------|------|------|------|-------|-----|
| Act | 88 | 42 | 65 | 39 | 92 | 58 | 41 |
| ReAct (avg) | 74 | 67 | 72 | 83 | 76 | 55 | 45 |
| ReAct (best) | 96 | 86 | 78 | 78 | 78 | 71 | 71 |
| BUTLER | 46 | 39 | 74 | 100 | 22 | 33 | 22 |

**WebShop 结果**：

| 方法 | Score | Success Rate |
|------|-------|--------------|
| Act | 62.3 | 30.1 |
| **ReAct** | **66.6** | **40.0** |
| IL | 59.9 | 29.1 |
| IL+RL | 62.4 | 28.7 |

**关键发现**：
1. ReAct 在 ALFWorld 上取得 71% 成功率，显著超越 Act（41%）和 BUTLER（22%）
2. ReAct 在 WebShop 上取得 40% 成功率，超越 IL+RL 方法（28.7%）
3. 甚至最差的 ReAct 尝试（48%）也超越了 Act 和 BUTLER 的最佳尝试

### 消融实验

#### 推理 vs 行动的贡献
通过人工分析 200 个轨迹的成败模式：

| 类型 | ReAct | CoT |
|------|-------|-----|
| **成功** | | |
| - 正确推理和事实 | 94% | 86% |
| - 幻觉推理或事实 | 6% | 14% |
| **失败** | | |
| - 推理错误 | 47% | 16% |
| - 搜索结果错误 | 23% | -- |
| - 幻觉 | -- | 56% |
| - 标签歧义 | 29% | 28% |

**关键洞察**：
- ReAct 通过外部知识库显著减少幻觉（6% vs 14%）
- 但结构化约束也降低了推理灵活性（47% vs 16% 的推理错误）
- 搜索质量对 ReAct 成功率至关重要（23% 的失败源于搜索结果不佳）

#### 内部推理 vs 外部反馈
对比 ReAct 和 Inner Monologue (IM) 风格提示：

| 方法 | 总体成功率 |
|------|------------|
| ReAct | 71% |
| ReAct-IM | 53% |

**关键洞察**：
- ReAct 的稀疏、灵活思维显著优于 IM 的密集外部反馈
- ReAct 能更好地分解目标和运用常识推理

#### 微调效果

| 方法 | HotpotQA EM |
|------|-------------|
| PaLM-62B Prompting ReAct | 27.4 |
| PaLM-8B Finetune ReAct | 30.2 |
| PaLM-62B Finetune ReAct | 35.1 |

**关键洞察**：
- 微调 3,000 个 ReAct 轨迹就能显著提升性能
- PaLM-8B 微调的 ReAct 超越所有 62B 提示方法
- PaLM-62B 微调的 ReAct 超越所有 540B 提示方法

## 深度分析

### 研究价值评估

#### 理论贡献
- **贡献 1：推理-行动协同范式**
  - 创新点：首次在 LLM 中将推理和行动整合到同一个轨迹
  - 学术价值：开创了 Agent 推理-行动协同的研究方向
  - 影响范围：LLM Agent、具身智能、工具使用

- **贡献 2：扩展动作空间理论**
  - 创新点：将语言空间纳入动作空间 $\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$
  - 学术价值：为 Agent 设计提供了理论框架
  - 影响范围：Agent 架构设计、强化学习与 LLM 结合

#### 实际应用价值
- **应用场景 1：知识密集型问答**
  - 适用性：需要多跳推理和外部知识检索的 QA 任务
  - 优势：减少幻觉，提高答案的事实准确性
  - 潜在影响：搜索引擎、智能助手、知识图谱

- **应用场景 2：交互式决策系统**
  - 适用性：需要长期规划和异常处理的决策任务
  - 优势：提高任务成功率，增强可解释性
  - 潜在影响：自动化系统、机器人控制、游戏 AI

- **应用场景 3：可解释 AI 系统**
  - 适用性：需要决策透明度的应用
  - 优势：推理轨迹是自然语言，人类可直接理解
  - 潜在影响：医疗诊断、金融决策、法律咨询

#### 领域影响
- **短期影响**：推动 LLM Agent 研究热潮，大量后续工作基于 ReAct 范式
- **中期影响**：Agent 系统从纯推理或纯行动转向推理-行动协同
- **长期影响**：成为 Agent 系统设计的基础范式，影响具身智能、自动化等领域
- **潜在变革**：从"LLM 作为生成器"到"LLM 作为 Agent"的范式转变

### 方法优势详解

#### 优势 1：减少幻觉，提高事实准确性
- **描述**：通过与外部知识库交互，ReAct 能够获取真实信息，减少基于内部记忆的幻觉
- **技术基础**：动作空间包含搜索、查询等与外部环境交互的能力
- **实验验证**：在 HotpotQA 上，ReAct 的假阳性率仅 6%，远低于 CoT 的 14%
- **对比分析**：相比纯 CoT，ReAct 的事实错误率降低 57%

#### 优势 2：增强可解释性和可信度
- **描述**：推理轨迹是自然语言，人类可以直接理解模型的决策过程
- **技术基础**：思维空间是语言空间，推理过程显式化
- **实验验证**：人工分析显示 ReAct 轨迹更易于理解和诊断
- **对比分析**：相比黑盒决策，ReAct 提供完整的决策链路

#### 优势 3：通用性和灵活性
- **描述**：适用于不同类型的任务（QA、验证、游戏、导航）
- **技术基础**：灵活的思维-行动格式，可根据任务特点调整
- **实验验证**：在 4 个不同领域都取得显著效果
- **对比分析**：相比任务特定方法，ReAct 具有更好的泛化能力

#### 优势 4：高效的少样本学习
- **描述**：仅需 1-6 个上下文示例就能学习任务
- **技术基础**：强大的 LLM 先验知识 + 人类示例的高效利用
- **实验验证**：在 ALFWorld 上，1-shot ReAct 超越 10^5 训练的模仿学习方法
- **对比分析**：相比需要大量训练数据的方法，ReAct 更加高效

### 局限性分析

#### 局限 1：推理灵活性受限
- **描述**：结构化的推理-行动约束降低了推理的灵活性
- **表现**：47% 的失败案例源于推理错误，包括重复生成相同的思维和动作
- **原因**：交替的思维-行动格式限制了自由推理
- **影响**：在需要复杂推理的任务上可能不如纯 CoT
- **可能的解决方案**：使用更好的解码策略（如 beam search）或更灵活的思维-行动调度

#### 局限 2：搜索质量依赖
- **描述**：ReAct 的成功高度依赖于外部搜索的质量
- **表现**：23% 的失败案例源于搜索结果不佳
- **原因**：使用简单的 Wikipedia API，检索能力有限
- **影响**：在知识库覆盖不足的领域效果受限
- **可能的解决方案**：使用更强大的检索系统或神经检索器

#### 局限 3：上下文长度限制
- **描述**：复杂任务需要更多示例，但容易超出上下文长度限制
- **表现**：在 ALFWorld 等长序列任务中，轨迹可能很长
- **原因**：当前 LLM 的上下文窗口有限
- **影响**：难以处理需要大量步骤的任务
- **可能的解决方案**：使用更长上下文的 LLM 或压缩轨迹的技术

#### 局限 4：缺乏学习和适应能力
- **描述**：基于提示的 ReAct 无法从新经验中学习
- **表现**：相同类型的错误会重复出现
- **原因**：不更新模型参数或记忆
- **影响**：难以持续改进性能
- **可能的解决方案**：结合微调或在线学习机制

### 适用性与场景分析

#### 适用场景
- **场景 1：多跳知识问答**
  - 适用原因：需要推理多个知识片段并检索外部信息
  - 预期效果：显著减少幻觉，提高答案准确性
  - 注意事项：需要高质量的知识库和检索系统

- **场景 2：交互式任务完成**
  - 适用原因：需要长期规划、子目标分解和异常处理
  - 预期效果：提高任务成功率，增强可解释性
  - 注意事项：任务动作空间不能太大

- **场景 3：可解释决策系统**
  - 适用原因：需要决策透明度和可审计性
  - 预期效果：提供完整的决策推理链
  - 注意事项：推理质量取决于 LLM 能力

#### 不适用场景
- **场景 1：纯数学或逻辑推理**
  - 不适用原因：不需要外部信息，纯推理更高效
  - 替代方案：使用 CoT 或专门的推理模型

- **场景 2：实时系统**
  - 不适用原因：推理-行动循环增加延迟
  - 替代方案：使用端到端模型或缓存机制

- **场景 3：高风险决策**
  - 不适用原因：LLM 的推理可能不可靠
  - 替代方案：结合规则系统或人类监督

## 与相关论文对比

### 对比论文选择依据
选择与 ReAct 最相关的 3 篇论文：
1. **Chain-of-Thought (CoT)**：纯推理的代表方法
2. **WebGPT**：纯行动的代表方法
3. **Inner Monologue**：另一种推理-行动结合方式

### [[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models]] - CoT

#### 基本信息
- **作者**：Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, Denny Zhou
- **发表时间**：2022 年
- **会议/期刊**：NeurIPS 2022
- **核心方法**：通过提示生成推理链，逐步解决复杂问题

#### 方法对比
| 对比维度 | CoT | ReAct |
|----------|-----|-------|
| 核心思想 | 纯推理，生成思维链 | 推理-行动协同 |
| 技术路线 | 提示工程 | 扩展动作空间 |
| 关键组件 | 思维链生成 | 思维 + 动作 + 观察 |
| 创新程度 | 开创性工作 | 范式扩展 |

#### 性能对比
| 数据集 | 指标 | CoT | ReAct | 提升幅度 |
|--------|------|-----|-------|----------|
| HotpotQA | EM | 29.4 | 27.4 | -6.8% |
| Fever | Acc | 56.3 | 60.9 | +8.2% |

#### 关系分析
- **关系类型**：扩展/改进
- **本文改进**：将推理与外部行动结合，减少幻觉
- **优势**：更事实准确，可解释性更强
- **劣势**：纯推理灵活性更高
- **互补性**：ReAct + CoT-SC 组合效果最佳

### [[WebGPT: Browser-assisted question answering with human feedback]] - WebGPT

#### 基本信息
- **作者**：Reiichiro Nakano, Jacob Hilton, Saurav Kadavath, et al.
- **发表时间**：2021 年
- **会议/期刊**：--
- **核心方法**：使用 LLM 与浏览器交互，搜索和综合信息回答问题

#### 方法对比
| 对比维度 | WebGPT | ReAct |
|----------|--------|-------|
| 核心思想 | 纯行动，交互式信息检索 | 推理-行动协同 |
| 技术路线 | 强化学习 | 提示学习 |
| 关键组件 | 浏览器交互 | 思维 + 搜索 API |
| 创新程度 | 早期 Web Agent | 推理-行动范式 |

#### 性能对比
| 数据集 | 指标 | WebGPT | ReAct | 说明 |
|--------|------|--------|-------|------|
| HotpotQA | EM | ~30 | 27.4 | WebGPT 使用 RL 训练 |
| Fever | Acc | -- | 60.9 | ReAct 无需 RL 训练 |

#### 关系分析
- **关系类型**：对比/改进
- **本文改进**：显式建模推理过程，无需昂贵的 RL 训练
- **优势**：更高效的学习方式，更强的可解释性
- **劣势**：WebGPT 的浏览器交互能力更强
- **互补性**：可结合 ReAct 的推理和 WebGPT 的强检索能力

### [[Inner Monologue: Embodied Reasoning through Planning with Language Models]] - Inner Monologue

#### 基本信息
- **作者**：Wenlong Huang, Fei Xia, Ted Xiao, et al.
- **发表时间**：2022 年
- **会议/期刊**：--
- **核心方法**：通过"内部独白"进行具身推理和规划

#### 方法对比
| 对比维度 | Inner Monologue | ReAct |
|----------|-----------------|-------|
| 核心思想 | 密集外部反馈驱动推理 | 稀疏、灵活的内部推理 |
| 技术路线 | 环境反馈主导 | 推理-行动协同 |
| 关键组件 | 环境状态反馈 | 多样化思维类型 |
| 创新程度 | 早期闭环系统 | 推理-行动范式 |

#### 性能对比
| 数据集 | 指标 | Inner Monologue | ReAct | 提升幅度 |
|--------|------|-----------------|-------|----------|
| ALFWorld | 成功率 | 53% | 71% | +34% |

#### 关系分析
- **关系类型**：改进
- **本文改进**：更灵活的推理空间，多样化的思维类型
- **优势**：更好的目标分解和常识推理能力
- **劣势**：需要更强的 LLM 能力
- **互补性**：可结合两者的优势

### 对比总结

| 方法 | 推理能力 | 行动能力 | 可解释性 | 训练成本 | 适用场景 |
|------|----------|----------|----------|----------|----------|
| CoT | ✅ | ❌ | ✅ | 低 | 纯推理任务 |
| WebGPT | ❌ | ✅ | ❌ | 高 | 信息检索 |
| Inner Monologue | 部分 | ✅ | 部分 | 中 | 具身任务 |
| **ReAct** | **✅** | **✅** | **✅** | **低** | **通用** |

## 技术路线定位

### 所属技术路线
本文属于 **LLM Agent 推理-行动协同**技术路线，该技术路线的核心特点是：
- 将 LLM 作为 Agent 的核心
- 推理和行动在同一个框架中协同工作
- 利用 LLM 的语言能力和世界知识

### 技术路线发展历程
```
CoT (2022) → WebGPT (2021) → Inner Monologue (2022) → ReAct (2022) → Toolformer (2023) → AutoGPT (2023)
   ↑              ↑                 ↑                    ↑                ↑                ↑
 纯推理         纯行动           早期闭环系统         推理-行动范式      工具使用         自主 Agent
```

### 本文在技术路线中的位置
- **承上**：继承了 CoT 的推理能力和 WebGPT 的行动能力
- **启下**：为后续 Agent 系统（如 LangChain、AutoGPT）奠定基础
- **关键节点**：开创了推理-行动协同范式，成为 Agent 研究的里程碑

### 具体子方向
本文主要关注 **few-shot LLM Agent**，该子方向的研究重点是：
- 如何用少量示例让 LLM 学习复杂任务
- 如何平衡推理和行动
- 如何提高 Agent 的可解释性

## 未来工作建议

### 作者建议的未来工作
1. **多任务训练**
   - 建议：在更多任务上训练 ReAct
   - 可行性：高，已有初步实验验证
   - 价值：提升泛化能力
   - 难度：需要大量高质量标注数据

2. **结合强化学习**
   - 建议：将 ReAct 与 RL 结合
   - 可行性：中，需要设计合适的奖励函数
   - 价值：提升探索和利用能力
   - 难度：RL 训练不稳定

3. **更好的解码策略**
   - 建议：使用 beam search 等解码策略
   - 可行性：高，技术成熟
   - 价值：减少重复生成等推理错误
   - 难度：低

### 基于分析的未来方向
1. **动态思维-行动调度**
   - 动机：当前的固定模式可能不适合所有任务
   - 可能的方法：学习何时需要推理，何时直接行动
   - 预期成果：更灵活的推理-行动协同
   - 挑战：需要设计合适的训练信号

2. **长期记忆机制**
   - 动机：当前 ReAct 无法从经验中学习
   - 可能的方法：添加外部记忆或经验回放
   - 预期成果：持续改进的 Agent
   - 挑战：记忆管理和检索效率

3. **多模态 ReAct**
   - 动机：现实世界是多模态的
   - 可能的方法：将视觉、听觉等模态纳入动作空间
   - 预期成果：更通用的具身 Agent
   - 挑战：多模态表征和对齐

4. **协作式 ReAct**
   - 动机：复杂任务需要多个 Agent 协作
   - 可能的方法：多个 ReAct Agent 通信和协调
   - 预期成果：解决更复杂的任务
   - 挑战：通信协议和任务分配

### 改进建议
1. **增强检索能力**
   - 当前问题：简单的 Wikipedia API 检索能力有限
   - 改进方案：使用神经检索器或更强的搜索引擎
   - 预期效果：减少搜索失败，提高答案质量

2. **自适应步数控制**
   - 当前问题：固定的步数限制可能不适合所有任务
   - 改进方案：学习何时停止或调整步数
   - 预期效果：更高效的任务解决

3. **思维质量评估**
   - 当前问题：无法评估思维的质量
   - 改进方案：添加思维质量评估机制
   - 预期效果：提高推理质量

## 我的综合评价

### 价值评分

#### 总体评分
**9.0/10** - 开创性工作，奠定了 LLM Agent 推理-行动协同范式，影响深远。

#### 分项评分

| 评分维度 | 分数 | 评分理由 |
|----------|------|----------|
| 创新性 | 9/10 | 首次将推理和行动整合到 LLM 同一轨迹，开创了新范式 |
| 技术质量 | 9/10 | 方法设计简洁优雅，理论清晰，实验充分 |
| 实验充分性 | 9/10 | 在 4 个不同领域验证，包括消融实验和人工分析 |
| 写作质量 | 9/10 | 清晰易懂，图表丰富，示例详细 |
| 实用性 | 9/10 | 方法简单易用，广泛适用，后续影响巨大 |

### 重点关注

#### 值得关注的技术点
1. **扩展动作空间**：$\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$ 的设计思想
2. **推理-行动协同机制**：思维如何帮助行动，行动如何帮助推理
3. **人类可解释性**：自然语言推理轨迹的价值

#### 需要深入理解的部分
1. **两种推理-行动模式**：密集思维 vs 稀疏思维的适用场景
2. **ReAct + CoT-SC 组合**：如何平衡内部知识和外部知识
3. **失败模式分析**：推理错误、搜索失败、幻觉的分布

## 我的笔记

%% 用户可以在这里添加个人阅读笔记 %%

## 相关论文

### 直接相关
- [[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models|CoT]] - 纯推理方法，ReAct 的重要基线和互补方法
- [[WebGPT: Browser-assisted question answering with human feedback|WebGPT]] - 纯行动方法，ReAct 的另一个重要对比
- [[Inner Monologue: Embodied Reasoning through Planning with Language Models|Inner Monologue]] - 另一种推理-行动结合方式

### 背景相关
- [[PaLM: Scaling Language Modeling with Pathways|PaLM]] - ReAct 使用的基础模型
- [[Self-Consistency Improves Chain of Thought Reasoning in Language Models|CoT-SC]] - ReAct 实验中的重要基线

### 后续工作
- [[Toolformer: Language Models Can Teach Themselves to Use Tools|Toolformer]] - 受 ReAct 启发的工具使用方法
- [[LLaMA-Tool: Augmenting LLMs with Tools|LLaMA-Tool]] - ReAct 范式的后续发展
- [[Generative Agents: Interactive Simulacra of Human Behavior|Generative Agents]] - 将 ReAct 思想应用于多 Agent 系统

## 外部资源
- **项目主页**：https://react-lm.github.io/
- **代码实现**：https://github.com/ysymyth/ReAct
- **GPT-3 实现**：https://github.com/openai/tree/main/examples/react
- **LangChain ReAct**：https://python.langchain.com/docs/modules/agents/agent_types/react

> [!tip] 关键启示
> 推理和行动不是割裂的——让 LLM 在同一个轨迹中交替思考和行动，可以实现双向增强：推理帮助规划，行动提供真实信息。

> [!warning] 注意事项
> - ReAct 的成功高度依赖于外部搜索的质量
> - 推理-行动格式可能限制推理的灵活性
> - 需要高质量的上下文示例来引导 LLM

> [!success] 推荐指数
> ⭐⭐⭐⭐⭐ 强烈推荐阅读！这是 LLM Agent 领域的里程碑论文，奠定了推理-行动协同范式，对后续 Agent 研究影响深远。

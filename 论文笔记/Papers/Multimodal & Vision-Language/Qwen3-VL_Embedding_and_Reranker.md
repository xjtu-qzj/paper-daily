---
date: "2026-05-06"
paper_id: "arXiv:2601.04720"
title: "Qwen3-VL-Embedding and Qwen3-VL-Reranker: A Unified Framework for Multimodal Retrieval and Ranking"
authors: "Mingxin Li et al. (Qwen Team)"
domain: "Multimodal & Vision-Language"
tags:
  - 论文笔记
  - Multimodal-&-Vision-Language
  - Qwen3-VL-Embedding
  - Qwen3-VL-Reranker
  - Multimodal-Retrieval
  - Matryoshka-Representation
  - Cross-Encoder
  - Embedding
quality_score: "8.0/10"
created: "2026-05-06"
updated: "2026-05-06"
status: analyzed
---

# Qwen3-VL-Embedding & Qwen3-VL-Reranker: Multimodal Retrieval and Ranking

## 核心信息
- **论文ID**：arXiv:2601.04720
- **作者**：Mingxin Li et al.（Qwen Team, 12 authors）
- **机构**：Alibaba Group
- **发布时间**：2026-01
- **链接**：[arXiv](https://arxiv.org/abs/2601.04720)
- **模型**：Embedding 2B/8B + Reranker 2B/8B

## 摘要翻译

基于 Qwen3-VL 基础模型构建的端到端多模态检索与排序框架。Qwen3-VL-Embedding 通过多阶段训练（对比预训练→Reranker 蒸馏）实现高质量多模态嵌入，支持 Matryoshka 表示学习实现灵活嵌入维度，输入最多 32K token。Qwen3-VL-Reranker 是交叉编码器，用交叉注意力实现细粒度查询-文档相关性估计。两个模型均支持 30+ 语言。8B Embedding 在 MMEB-V2 上以 77.8 总分排名第 1。

## 方法概述

### 整体架构

![[pipeline_page1.png|800]]

> 图1：Embedding + Reranker 两阶段检索管道

### Qwen3-VL-Embedding（双编码器）

**基础架构**：基于 Qwen3-VL，去掉生成头，将最后一层隐藏状态池化为固定维度向量。

**训练三阶段**：

1. **对比预训练**：用 InfoNCE 损失在大规模图文对 $(q, d^+, d^-_1, ..., d^-_n)$ 上训练。使得正样本的嵌入靠近查询，负样本远离：

$$\mathcal{L}_{NCE} = -\log \frac{\exp(s(q, d^+) / \tau)}{\sum_i \exp(s(q, d_i) / \tau)}$$

2. **Reranker 蒸馏**：用更强的 Qwen3-VL-Reranker 作为教师，其相关性分数作为软标签。嵌入模型学习模仿 Reranker 的精细判断：

$$\mathcal{L}_{distill} = \text{MSE}(s_{emb}(q, d), s_{rerank}(q, d))$$

3. **Matryoshka 表示学习**：训练时同时优化多个维度的嵌入（256, 512, 1024, 2048），每个维度独立计算对比损失。推理时可根据延迟/精度需求选择维度。

![[qwen3vl_embed_arch_page1.png|600]]

> 图2：Embedding 模型架构

### Qwen3-VL-Reranker（交叉编码器）

![[demonstration_page1.png|600]]

**与 Embedding 的关键差异**：Reranker 是**交叉编码器**——查询和文档拼接后一起输入模型，通过交叉注意力实现细粒度交互。

- 输入：$[query, document]$ 拼接序列
- 输出：单一相关性分数（通过 [CLS] token 的线性投影）
- 精度远超双编码器（Embedding），但速度慢 $10\times$ 以上（无法预计算文档向量）

**两阶段检索的典型使用**：
1. Embedding 粗筛：从百万文档中召回 top-100
2. Reranker 精排：对 100 个候选逐一打分，返回 top-10

### 多模态检索支持

输入可包含图文混合 query 和 document：
- Text-to-Image 检索
- Image-to-Text 检索
- Text+Image → Document Image 检索
- Video-to-Text 检索

![[data_distribution_page1.png|600]]

> 图3：训练数据分布

## 实验结果

### MMEB-V2 基准（多模态嵌入综合评估）

![[performance_comparison_page1.png|600]]

> 图4：MMEB-V2 性能对比

| 模型 | MMEB-V2 总分 | 排名 |
|------|:---:|:---:|
| **Qwen3-VL-Embed-8B** | **77.8** | #1 |
| Qwen3-VL-Embed-2B | 73.2 | #3 |
| 商业 API-A | 76.5 | #2 |
| 开源 SOTA（前） | 71.0 | -- |

### 关键发现
- Matryoshka 降维到 256 维时仅损失 ~1.5 分，存储节省 8×
- Reranker 8B 比 Embedding 8B 在 top-10 精度上提升 8-12 个百分点
- 30+ 语言的检索质量一致，中英之外的罕见语言无明显退化

## 深度分析

### 设计亮点
- **Reranker 蒸馏嵌入模型**：将交叉编码器的精细判断能力"压缩"到高效的双编码器
- **Matryoshka**：一个模型覆盖所有嵌入维度需求，无需为不同维度训练多个版本
- **基于 VL 基础模型**：文本、图像、视频的检索共享统一表示空间，无需域适配

### 局限性
- Reranker 推理速度慢（适合 top-100 而非全量），大规模实时检索场景需仔细权衡
- 32K token 上限对于超长文档仍有限制
- 视频检索目前仅支持短片段（< 10 分钟）

## 我的综合评价

### 总体评分：8.0/10

| 维度 | 分数 |
|------|:---:|
| 创新性 | 7/10 |
| 技术质量 | 9/10 |
| 实验充分性 | 8/10 |
| 写作质量 | 8/10 |
| 实用性 | 9/10 |

> [!tip] 关键启示
> Embedding + Reranker 的两阶段范式比单独的 Embedding 效果好一个量级——Reranker 蒸馏更是让 Embedding 模型"偷师"了交叉编码器的精细判断力。

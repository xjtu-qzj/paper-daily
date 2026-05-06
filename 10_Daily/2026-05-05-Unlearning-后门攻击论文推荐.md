---
keywords: ["machine unlearning", "backdoor attack", "backdoor defense", "knowledge erasure", "federated learning", "robust unlearning", "poisoning", "LLM backdoor", "adversarial attack", "data privacy"]
tags: ["llm-generated", "daily-paper-recommend"]
---

## 今日概览

今日推荐的10篇论文聚焦于 **machine unlearning** 与 **backdoor attack** 的交叉领域，主要覆盖**鲁棒知识遗忘**、**后门攻击新范式**、**后门防御与检测**三大方向。

- **总体趋势**：Unlearning 与 backdoor 的交叉研究正快速升温。一方面，研究者发现 unlearning 过程本身可能引入新的安全漏洞（如知识污染被利用于推理攻击和间接攻击）；另一方面，unlearning 也被用作后门防御的工具。联邦学习、LLM、VLM 等场景下的后门与遗忘问题正在成为热点。
- **质量分布**：由于搜索到的论文均来自过去一年（Semantic Scholar 热门论文），引用数整体偏低（0-31），评分范围 0.99-1.96，但这些论文覆盖了该方向的前沿问题和新兴范式。
- **研究热点**：
  - **鲁棒遗忘与对抗**：ROKA 提出 Neural Healing 框架，首个为遗忘过程提供知识保持理论保证
  - **后门攻击新范式**：可撤销后门、联邦遗忘后门、间接遗忘攻击等新型攻击面
  - **LLM 后门检测**：BAIT（SP'25, TrojAI 竞赛榜首）通过逆向攻击目标检测 LLM 后门
- **阅读建议**：建议优先阅读 ROKA（鲁棒遗忘理论框架）、BAIT（LLM 后门扫描 SOTA）和 BadFU（联邦遗忘安全新视角）。

---

### [[20_Research/Papers/unlearning backdoor attack/ROKA_Robust_Knowledge_Unlearning_against_Adversaries|ROKA: Robust Knowledge Unlearning against Adversaries]]
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2603.00436) | [Semantic Scholar](https://www.semanticscholar.org/paper/e323001f48903d3163a77dc73a1eb69227c97e69)
- **来源**：Semantic Scholar (Hot)
- **详细报告**：[[20_Research/Papers/unlearning backdoor attack/ROKA_Robust_Knowledge_Unlearning_against_Adversaries|ROKA 深度分析]]
- **笔记**：已生成详细分析

**一句话总结**：首个为遗忘过程提供知识保持理论保证的框架，提出 Neural Healing 策略——在消除遗忘数据影响的同时强化其概念邻居，有效防御间接遗忘攻击。

![[threat_model_A.png|600]]

**核心贡献/观点**：
- 揭示间接遗忘攻击（无需数据操纵，仅利用知识污染扰动模型安全关键预测）
- 提出 Neural Knowledge Systems 理论框架建模神经网络
- ROKA 策略在遗忘数据的同时恢复性地重新平衡模型，保护相关概念知识
- 首个提供遗忘过程中知识保持理论保证的工作

**关键结果**：在 Vision Transformer、多模态模型和 LLM 上验证，有效遗忘目标同时保持甚至增强保留数据的准确性。

---

### Trap: Mitigating Poisoning-Based Backdoor Attacks by Treating Poison With Poison
- **作者**：--
- **机构**：--
- **链接**：[DOI](https://doi.org/10.1109/TDSC.2025.3644371)
- **来源**：IEEE TDSC (Hot)
- **笔记**：--

**一句话总结**：以毒攻毒——利用中毒样本在训练早期形成的聚类特征和分类器中的后门路径，通过重标记中毒样本重新训练分类器来移除后门。

**核心贡献/观点**：
- 发现中毒样本在训练早期即形成远离良性样本的聚类
- 发现分类器中存在连接后门特征到目标标签的独特通路
- 提出训练早期检测中毒样本 + 重标记重训练分类器的方法
- 无需 unlearning，避免其导致的次优性能

**关键结果**：12 种攻击 × 4 个数据集上 ASR 降至 0.07%，准确率仅下降 0.33%。

---

### BadFU: Backdoor Federated Learning through Adversarial Machine Unlearning
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2508.15541) | [Semantic Scholar](https://www.semanticscholar.org/paper/943e37f86c7c9d3f1dbb3e962ae8a4c2ecea6472)
- **来源**：RAID 2025 (Hot)
- **笔记**：--

**一句话总结**：联邦遗忘领域的首个后门攻击——恶意客户端通过正常的遗忘请求（请求遗忘伪装样本）将全局模型隐蔽地转为后门状态。

**核心贡献/观点**：
- 联邦学习中遗忘过程的首次安全分析
- 恶意客户端使用后门样本 + 伪装样本正常训练，之后请求遗忘伪装样本即触发后门
- 揭示了当前联邦遗忘机制的关键漏洞

**关键结果**：在多种 FL 框架和遗忘策略下验证有效，暴露联邦遗忘的严重安全风险。

---

### [[20_Research/Papers/unlearning backdoor attack/Injection_Attack_and_Erasure_Revocable_Backdoor_Attacks_via_Machine_Unlearning|Injection, Attack and Erasure: Revocable Backdoor Attacks via Machine Unlearning]]
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2510.13322)
- **来源**：arXiv (Hot)
- **详细报告**：[[20_Research/Papers/unlearning backdoor attack/Injection_Attack_and_Erasure_Revocable_Backdoor_Attacks_via_Machine_Unlearning|可撤销后门 深度分析]]
- **笔记**：已生成详细分析

**一句话总结**：首个可撤销后门攻击范式——攻击达成后可主动、彻底地通过遗忘机制清除后门，不留静态分析痕迹。

![[framework_page1.png|600]]

**核心贡献/观点**：
- 提出可撤销后门攻击新范式
- 将触发器优化建模为双层优化问题：同时优化攻击成功率和可遗忘性
- 使用确定性中毒/遗忘样本分配 + PCGrad 解决注入和移除目标的优化冲突

**关键结果**：CIFAR-10 和 ImageNet 上 ASR 与 SOTA 后门攻击相当，遗忘后后门行为可被有效移除。

---

### Illuminating the Black Box: Real-Time Monitoring of Backdoor Unlearning in CNNs via Explainable AI
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2511.21291)
- **来源**：arXiv (Hot)
- **笔记**：--

**一句话总结**：将 Grad-CAM 可解释性集成到后门遗忘过程中，提出 TAR 度量实时监控模型注意力从触发器到合法特征的转移。

**核心贡献/观点**：
- 首次将 XAI 集成到后门遗忘过程实现实时可解释监控
- 提出 Trigger Attention Ratio (TAR) 度量注意力转移
- 平衡遗忘策略：梯度上升 + EWC 防灾难遗忘 + 恢复阶段

**关键结果**：CIFAR-10 BadNets 上 ASR 从 96.51% 降至 5.52%，清洁准确率保持 99.48%。

---

### DarkHash: A Data-Free Backdoor Attack Against Deep Hashing
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2510.08094)
- **来源**：IEEE TIFS (Hot)
- **笔记**：--

**一句话总结**：首个无需访问训练数据的深度哈希后门攻击，利用代理数据集和拓扑对齐损失实现双重语义引导的影子后门嵌入。

**核心贡献/观点**：
- 首个 data-free 深度哈希后门攻击
- 双重语义引导的影子后门框架
- 拓扑对齐损失：优化个体和邻居中毒样本朝向目标样本

**关键结果**：4 个数据集 × 5 个架构 × 2 种哈希方法验证，抗主流防御。

---

### BAIT: Large Language Model Backdoor Scanning by Inverting Attack Target
- **作者**：--
- **机构**：--
- **链接**：[DOI](https://doi.org/10.1109/SP61157.2025.00103)
- **来源**：IEEE S&P 2025 (Hot, 31 citations)
- **笔记**：--

**一句话总结**：IEEE S&P'25 + TrojAI 竞赛榜首——通过逆向攻击目标（而非触发器）来扫描 LLM 后门，利用自回归训练中目标 token 间的强因果性大幅缩小搜索空间。

**核心贡献/观点**：
- LLM 后门学习的理论分析：自回归训练在目标 token 间引入强因果关系
- 逆向目标而非逆向触发器的扫描策略，将指数级搜索空间转为可处理
- 仅需黑盒访问即可扫描后门

**关键结果**：153 个 LLM × 8 架构 × 6 种攻击类型验证，超越 5 个基线，TrojAI LLM 轮榜首。

---

### IAG: Input-aware Backdoor Attack on VLM-based Visual Grounding
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2508.09456)
- **来源**：arXiv (Hot, 8 citations)
- **笔记**：--

**一句话总结**：首个 VLM 视觉定位多目标后门攻击——文本条件化 UNet 动态生成输入感知触发器，可针对任意目标描述执行攻击。

**核心贡献/观点**：
- VLM 定位系统的首次安全审计
- 文本条件化 UNet 生成输入感知、文本引导的动态触发器
- 联合训练目标平衡语言能力和感知重建

**关键结果**：LLaVA、InternVL、Ferret 等 VLM 上最佳 ASR，保持清洁准确率，跨数据集/模型可迁移。

---

### Step-by-Step Reasoning Attack: Revealing 'Erased' Knowledge in Large Language Models
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2506.17279)
- **来源**：arXiv (Hot, 4 citations)
- **笔记**：--

**一句话总结**：揭示逐步推理可作为恢复 LLM 中被遗忘知识的后门——62.5% 对抗提示成功恢复被遗忘内容，50% 暴露不公平的知识压制。

**核心贡献/观点**：
- 发现 unlearning 往往只是"压制"而非"删除"知识，逐步推理可解锁
- Sleek 黑盒攻击框架：逐步推理对抗提示生成 + 结构化攻击机制
- 提示分类（直接/间接/隐含）识别最有效的遗忘利用方式

**关键结果**：4 种 SOTA unlearning 方法 × 2 LLM，62.5% 恢复 Harry Potter 知识，50% 暴露保留知识的不公平压制。

---

### Unlearning-Enhanced Website Fingerprinting Attack: Against Backdoor Poisoning in Anonymous Networks
- **作者**：--
- **机构**：--
- **链接**：[arXiv](https://arxiv.org/abs/2506.13563)
- **来源**：arXiv (Hot)
- **笔记**：--

**一句话总结**：利用 unlearning 技术实现网站指纹攻击中的后门中毒自动检测与移除——计算训练样本影响力来识别中毒点并动态调整模型参数。

**核心贡献/观点**：
- 将 unlearning 与网站指纹（WF）攻击结合
- 利用已知中毒测试点的影响力分数识别训练数据中的中毒样本
- 量化模型参数对训练数据和清洁数据的贡献差异，动态调整目标参数

**关键结果**：闭集/开集场景下准确率稳定在 80%，运行效率比基线快 2-3 倍。

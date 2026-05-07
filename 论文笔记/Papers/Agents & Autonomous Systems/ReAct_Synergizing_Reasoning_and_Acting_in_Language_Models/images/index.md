# ReAct 论文图片索引

## 图片列表

### 核心架构图
- `Figure1.png` - ReAct 框架概览：对比四种提示方法（Standard、CoT、Act-only、ReAct）

### 实验结果图
- `Figure2.png` - PaLM-540B 在 HotpotQA 和 Fever 上的提示结果
- `Figure3.png` - HotpotQA 上的微调和提示缩放结果

### 任务示例图
- `Figure4.png` - HotpotQA 示例：ReAct vs CoT 的成功和失败模式

### 环境截图
- `ALFWorld_screenshot.png` - ALFWorld 文本游戏环境
- `WebShop_screenshot.png` - WebShop 在线购物环境

## 图片说明

### Figure 1: ReAct 框架概览
展示了四种提示方法的对比：
1. **Standard Prompting**：直接回答问题
2. **CoT (Reason Only)**：纯推理，生成思维链
3. **Act-only**：纯行动，直接搜索和回答
4. **ReAct (Reason+Act)**：推理-行动协同，交替生成思维和动作

关键观察：
- ReAct 能够通过推理指导搜索（"I need to search Apple Remote"）
- ReAct 能够从观察中提取信息并更新推理
- ReAct 的轨迹更易于人类理解和诊断

### Figure 2: 提示结果对比
展示了不同提示方法在 HotpotQA 和 Fever 上的性能：
- ReAct + CoT-SC 组合效果最佳
- ReAct 在 Fever 上超越 CoT
- 图表显示了 CoT-SC 样本数量对性能的影响

### Figure 3: 微调效果
展示了不同模型规模下提示和微调的性能：
- 微调 3,000 个 ReAct 轨迹就能显著提升性能
- PaLM-8B 微调的 ReAct 超越所有 62B 提示方法
- PaLM-62B 微调的 ReAct 超越所有 540B 提示方法

### Figure 4: 成功和失败模式
人工分析了 200 个轨迹的成败原因：
- ReAct 的假阳性率仅 6%，远低于 CoT 的 14%
- 47% 的 ReAct 失败源于推理错误
- 23% 的失败源于搜索结果不佳

## 使用说明

在笔记中引用图片时，使用 Obsidian 的 wikilink 语法：

```markdown
![[Figure1.png|800]]
```

或者带标题：

```markdown
![[Figure1.png|800]]
> 图1：ReAct 框架概览，展示了四种提示方法的对比
```

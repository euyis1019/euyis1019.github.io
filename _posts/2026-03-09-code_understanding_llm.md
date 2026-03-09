---
layout: post
title: "Your LLM Understood the Code — Then Forgot the Answer"
date: 2026-03-09
permalink: /blog/code_understanding_llm/
categories: [Research, Interpretability, Code]
tags: [LLM, Code, Interpretability, Patchscopes, Early-Exit]
excerpt: "We use Patchscopes to dissect how Code LLMs process five types of code understanding tasks layer-by-layer, revealing fundamentally different internal dynamics across data-flow and control-flow reasoning — and a striking phenomenon where models compute the right answer mid-forward-pass, then forget it."
---

*by Yifu Guo — March 2026*  
*Language: [中文](#zh) | [English](#en)*

---

<a name="zh"></a>

## 中文版

---

代码大模型在 SWE-bench 风格的评测上每隔几个月就会刷新记录，但我们对这些模型 *内部是如何处理代码* 的了解，依然非常有限。最终准确率告诉我们模型会不会，但并不告诉我们模型 *怎么* 会——或者更重要的，*为什么不会*。

本文记录了一系列基于 Patchscopes 的逐层可解释性实验。我们发现：对于代码 LLM 来说，不同类型的代码任务不只是"难度不同"，而是激活了截然不同的内部处理模式——拥有完全不同的信息成熟时序、稳定性签名和编码格式。

最惊人的发现之一：在某些任务上，模型曾经在中间层"算对了"，却在后续层将正确答案主动覆盖，最终输出错误。

---

### 一、一个让人不安的对比

把 **Qwen2.5-Coder 7B** 放在两类等难度任务上：

| 任务 | 最终准确率 |
|---|---|
| Conditional（控制流：if/else 条件判断） | **81.7%** |
| Computing（数据流：变量追踪 + 算术运算） | **18.6%** |

差距高达 63 个百分点。这不是因为这两组题目难度不同——我们后面会通过 Difficulty-Matched 实验证明，即使在赋值链难度相同的情况下，Computing 格式依然更差。

这个差距在 Code Agent 的实际场景里不是抽象的。当一个 agent 在 resolve issue 时，它反复在做两件事：**追踪数据流**（这个变量链式赋值之后是什么值？）和**推理控制流**（这个条件成立，代码走哪条分支？）。如果模型处理这两类操作的内部机制完全不同，agent 的失败模式就会有很强的 task-specificity，而不是随机分布。

---

### 二、任务设计

我们设计了五类可控的代码理解题目，作为 interpretability 实验的 controlled stimuli：

**数据流组：**
- `Copying`：纯链式变量赋值追踪（例：`u = 7; t = u; d = t`，问 `d` 的值）
- `Computing`：链式赋值 + 基本算术（例：`a = 2; b = a + 1; c = b + 2`，问 `c` 的值）

**控制流组：**
- `Conditional`：if/else 条件判断（执行哪条分支后变量的值？）
- `Loop`：带循环的变量追踪（迭代 n 次后的结果）
- `Short-circuit`：短路求值（`and`/`or` 的惰性求值结果）

这些题目是程序生成的，答案由 Python 解释器计算保证 ground truth 确定性，且均约束为单数字答案（0-9），以获得干净的 Patchscopes 信号。

实验覆盖 **Qwen2.5-Coder 0.5B / 1.5B / 3B / 7B / 14B** 五个规模。

---

### 三、Patchscopes：给模型做逐层 MRI

本文的核心方法是 **Patchscopes**（Ghandeharioun et al., 2024），一种通过将模型中间层的 hidden state 注入到另一个 target prompt 来探测该层信息的工具。

具体配置：
- **Source**：代码题目的 prompt，提取最后 token 位置的 $h_\ell$（第 $\ell$ 层 hidden state）
- **Target**：`"# The value of x is \""` — 一个标准化的解码 prompt
- **注入位置**：Target prompt 的 **last position**（关键适配点，见下）
- **评估**：在 target prompt forward 完成后，用 LM head 看第一个 token 是否等于正确答案

**三个代码场景的方法论适配：**

**M1：必须用 last-position 注入。** 我们比较了以下注入策略：

| Prompt | 注入策略 | Final Acc | Ever Correct |
|---|---|---|---|
| `# The value of x is "` | last-pos | **79%** | **100%** |
| `0→0; 1→1; x→` | x-pos | 0% | 13% |
| `# x=` | x-pos | 13% | 20% |

x-position 注入几乎完全失效。原因是：注入 $h_\ell$ 后，后续 token（如 `→`）的 KV cache 是基于原始 source context 计算的，与注入内容不一致，造成信息污染。last-position 注入后直接进入生成阶段，没有后续 token 干扰。

**M2：Patchscopes 的解码能力是 task-dependent 的。** 对于 Computing 任务（高运算密度），PS 的 final accuracy 和 Probing 检测到的信息存在量之间存在巨大 gap（例如 7B 的 Probing peak 和 PS final 相差数十个百分点），说明信息存在不等于信息可被输出。

**M3：大模型需要 first-token 校正。** 14B 模型出现了 Copying instability > Computing 的违反直觉的反转，排查发现 79.7% 的样本被模型生成惯性续写成了幻觉文本（如连续生成字符串，而非在引号后输出单数字）。使用 first-token evaluation 截断后，pattern 恢复正常。

---

### 四、主要发现：两种截然不同的处理动力学

以 7B 模型为例，五个任务的完整数字画像如下：

| 维度 | Copying | Computing | Conditional | Loop | Short-circuit |
|---|---|---|---|---|---|
| Final Acc | 79% | **18.6%** | 81.7% | 51.6% | 85% |
| Ever Correct | 100% | 55.8% | 90% | 59.4% | 95.6% |
| Overthinking Gap | 21pp | **37.2pp** | 8.3pp | 7.8pp | 10.6pp |
| Instability | ≈0 | **0.356** | 0.22 | 0.03 | 0.04 |
| FCL 位置 | 晚（70-80%） | 早（50-60%） | 中 | 中 | 中 |
| 曲线形态 | 阶跃式 | 闪烁式 | 渐进式 | 阶跃式 | 渐进式 |

*Overthinking Gap = Ever Correct − Final Acc；Instability = 在 FCL 之后、最终稳定预测之前，预测值波动的层数比例。*

**数据流组内部的分裂最令人惊讶。** 两个都是"数据流"的任务，却有着完全相反的动力学特征：
- `Copying` 在浅层没有信号（FCL 在 70-80% 处），但一旦出现就极度稳定（instability ≈ 0）；
- `Computing` 极早出现信号（FCL 在 50-60% 处），但极度不稳定（instability = 0.356）。

**控制流组则保持一致的模式：** instability 均在低位（0.03-0.22），overthinking gap 均在 10pp 以内。

<!-- FIGURE 1 PLACEHOLDER -->
*[Figure 1: Layer-wise accuracy curves for all five tasks on the 7B model. X-axis: normalized layer depth (0-1). Y-axis: accuracy. Key features to highlight: the step-wise jump of Copying in the late layers; the flickering/oscillating pattern of Computing in mid-layers; the smooth progressive rise of Conditional.]*

**一个关键的排除实验：这不是难度问题。**

Difficulty-Matched 实验中，我们将 Computing 的运算密度设为 `d=0`（纯赋值链，无算术运算），使其与 Copying 的难度尽量接近。结果：

- `Copying`（7B）：Final Acc = 79%，Ever Correct = 100%
- `Computing (d=0)`（7B）：Final Acc = 29.2%，Ever Correct = 84.2%

即使题目结构几乎等价，Computing 格式下的处理仍然更不稳定。这说明导致 gap 的不是"计算难度"，而是代码格式本身激活了不同的处理链路。

---

### 五、Overthinking：算对了，又忘了

在 Computing 任务上，**7B 模型有 55.8% 的样本在某个中间层曾经输出了正确答案**，但最终层只有 18.6% 正确——中间有 **37.2pp** 的正确答案在推理过程中被覆盖、"想没了"。

这不是一个孤立的异常值。我们在全部 25 个 (Model, Task) 组合上验证了：**25/25 均存在比最终层准确率更高的退出层**（Binomial test $p < 10^{-7}$）。

<!-- FIGURE 2 PLACEHOLDER -->
*[Figure 2: Overthinking Gap heatmap, 5 models (0.5B–14B) × 5 tasks. X-axis: task; Y-axis: model size. Cell value: Ever Correct − Final Acc (pp). Notable: 7B Computing cell should be the deepest red at 37.2pp; 14B consistently shows lighter values across tasks.]*

Overthinking gap 的分布极度不均匀——它是 **task-specific 和 scale-specific** 的，而不是一个统一的现象：

- **控制流组**（Conditional / Loop / Short-circuit）：gap 均不超过 10pp
- **Computing**：7B 的 gap 高达 37.2pp，是同模型 Conditional 的约 4.5 倍
- **14B 模型**：所有任务的 gap 全面收窄（3.3-11.1pp），大模型 overthinking 整体更轻

---

### 六、Information Brewing：知道不等于能说出来

为什么 Computing 的 overthinking 如此严重？线性 Probing 实验给出了一个结构性的解释。

我们用 LogisticRegression 探针对每一层的 last-position hidden state 训练一个分类器，看该层的信息是否足以预测正确答案。对比 Probing 的 peak 准确率和 Patchscopes 的 final 准确率：

**Copying 任务（7B）：**
- Probing Peak：100% @ L22
- PS Final Acc：79% @ L27（最终层）
- Peak Gap：1pp

**Computing (d=0) 任务（7B）：**
- Probing Peak：100% @ 某层
- PS Final Acc：29.2%（最终层）
- Peak Gap：**70.8pp**

更极端的是 14B：
- Probing onset：Layer 2（极浅层就线性可分）
- PS 解码起飞：Layer 33
- **Gap：31 层**

这个 31 层的差距说明：信息以线性可识别的方式存在于 hidden state 中（Probing 能读出），但并不是以可以直接输出的格式编码的。模型需要额外的 30+ 层将其"翻译"为 output-ready 的格式——我们把这个过程称为 **Information Brewing（信息酝酿）**。

<!-- FIGURE 3 PLACEHOLDER -->
*[Figure 3: Dual curves for Copying (top) and Computing (bottom) on the 14B model. Each panel shows two lines: Probing accuracy (gradually rising from left) and Patchscopes accuracy (near-flat then steep jump right). The large horizontal gap between the Probing onset and PS onset in the Computing panel visually represents the brewing gap of ~31 layers.]*

Brewing gap 解释了 Computing 出现极端 overthinking 的根本原因：算术中间结果以"非 output-ready"格式编码，后续层需要强行转换。这个转换过程极其脆弱——稍有扰动，已经接近正确的表征就可能被覆盖，让模型最终错过了正确答案。

**Copying 则几乎没有 brewing gap：** 一旦变量追踪完成，信息就以 output-ready 格式直接存在，无需额外的格式转换层。

---

### 七、Loop 的伪循环理解

Loop 任务有一个独立的有趣发现。

| 迭代次数 | 0.5B | 1.5B | 3B | 7B | 14B |
|---|---|---|---|---|---|
| iter = 1 | 高 | **98.8%** | **95%** | **100%** | **100%** |
| iter = 2 | 低 | 15% | 16.2% | 41.2% | 73.8% |
| iter = 3 | 低 | 27.5% | 25% | 30% | 76.2% |
| iter = 4 | 低 | 13.8% | 31.2% | 35% | **100%** |

`iter=1` 时准确率极高，但 `iter>1` 时大幅崩溃，且这个断崖是突然的（Fisher 精确检验 $p < 0.03$）。

验证实验（Unrolled Loop）中，将循环展开成顺序代码后，7B 模型在 `iter=3` 上提升了 +70pp（统计显著）。这说明：**模型对 `iter=1` 的高准确率并不是来自"理解循环"，而是 pattern matching 走了 Copying 的数据流回路** — 单次迭代的循环格式和直接赋值在语义上几乎等价。一旦循环次数增加，Copying 回路失效，模型就暴露了自己没有真正建立迭代推理能力。

---

### 八、Task-Aware Early Exit：一个直接的应用

上述发现有一个清晰的实际含义：如果模型在某一中间层"已经知道答案了"（且尚未因为 overthinking 而忘记），我们是否可以在那里停止推理？

**Best Fixed Layer Exit（最有实用价值的策略）：** 基于 validation set 找到每个任务的最佳固定退出层，不依赖 oracle 知识。

7B 模型结果：

| 任务 | Final 准确率 | 最佳退出层 | 退出后准确率 | 净收益 |
|---|---|---|---|---|
| Copying | 79% | L23 | **99%** | +20pp |
| Computing | 18.6% | L22 | **45%** | +26.4pp |
| Conditional | 81.7% | L26 | 82.7% | +1pp |
| Loop | 51.6% | L22 | 54.1% | +2.5pp |
| Short-circuit | 85% | L23 | 88.9% | +3.9pp |

收益最大的两个任务恰好是 overthinking 最严重的 Copying 和 Computing。控制流组已经接近最优，early exit 几乎没有额外收益。这种 **task-specificity** 正是"task-aware"策略的核心价值——如果不区分任务，统一使用某个固定层退出，实际上会损害部分任务。

**25/25 个 (Model, Task) 组合均存在不低于 final layer 的退出层**（Binomial test $p < 10^{-7}$），overthinking 不是测量噪声，而是普遍存在的现象。

<!-- FIGURE 4 PLACEHOLDER -->
*[Figure 4: Grouped bar chart for the 7B model, four conditions per task: Final (baseline), Best Fixed Layer, Correct-k (k=3), Oracle. X-axis: the five tasks. Y-axis: accuracy (%). The Computing bar should show the most dramatic jump from Final (18.6%) to Best Fixed (45%) and Oracle (55.8%).]*

---

### 九、对 Code Agent 的含义

这组实验揭示的，不只是 LLM 在隔离任务上的行为，而是 code agent 的一个结构性弱点。

**数据流追踪的脆弱性：** Agent 在 resolve issue 时反复进行变量追踪和算术推断。Computing 任务的极端 overthinking（37.2pp gap）和 information brewing（信息以无法直接输出的格式存在）意味着，即使模型在某个中间层"算出了"正确的变量值，它也有很高概率在最终输出时将其遗忘。这不是 context 不够的问题，而是计算链路本身的脆弱。

**任务类型感知的重要性：** 同一个 7B 模型处理 Conditional（81.7%）和 Computing（18.6%）时表现出天壤之别，而普通 benchmark 的 final accuracy 会将这两种完全不同的内部状态混在一起汇报成一个平均分，掩盖了实际能力的 task-specificity。

**Loop 的假象：** 很多 agent benchmark 里的循环相关代码，模型可能只是在处理单次迭代的 Copying 版本，而非真正理解迭代推理。多次迭代的大幅崩溃，在复杂的真实代码库中可能更为严重。

---

### 十、局限性与展望

**当前局限：**
- 只测试了 Qwen2.5-Coder 一个模型家族；跨架构是否有相同 pattern 是开放问题
- 粒度停留在 layer-level；circuit-level（哪些 head、MLP 负责哪类计算）尚未探索
- Probing 只覆盖了 Copying 和 Computing；其他三个任务缺少 probing 的交叉验证
- 实验任务是 controlled stimuli，不是真实代码库

**开放问题：**
- 跨模型家族（Llama、Gemma、CodeLlama 等）是否有相同的 data-flow vs control-flow 分裂？
- 在多文件、长上下文的真实代码库中，两类能力的分离是否会加剧？
- 能否基于 layer-wise processing profile 设计自适应的推理路由策略？

---

*实验基于 Patchscopes，参考 Ghandeharioun et al. (2024)。感谢 Computing、Conditional、Loop 等任务设计参考了 interpretability 领域的受控刺激范式。*

---

<a name="en"></a>

## English Version

---

Code LLM benchmarks refresh every few months, yet our understanding of *how* these models process code internally remains thin. Final accuracy tells us whether a model can or cannot — but not *how* it succeeds or, more importantly, *why* it fails.

This post documents a series of Patchscopes-based layer-wise interpretability experiments. The central finding: different types of code tasks do not just differ in *difficulty* — they activate fundamentally different internal processing dynamics, with distinct timelines for information maturity, stability signatures, and encoding formats.

Most strikingly: on certain tasks, the model computes the correct answer mid-forward-pass, then actively overwrites it in subsequent layers.

---

### I. A Disquieting Comparison

Running **Qwen2.5-Coder 7B** on two task types of comparable difficulty:

| Task | Final Accuracy |
|---|---|
| Conditional (control-flow: if/else logic) | **81.7%** |
| Computing (data-flow: variable tracking + arithmetic) | **18.6%** |

A 63-percentage-point gap. This is not explained by raw difficulty — a controlled Difficulty-Matched experiment (described below) demonstrates that even when the assignment chain structure is equivalent, Computing format produces inferior results.

The gap is not abstract from a code agent perspective. When an agent resolves an issue, it repeatedly performs two fundamental operations: **data-flow tracking** (what is the value of this variable after a chain of assignments?) and **control-flow reasoning** (given this condition is true, which branch executes?). If the model's internal mechanisms for these two operations are fundamentally different, agent failure modes will be highly task-specific rather than randomly distributed.

---

### II. Task Design

We designed five categories of controlled code understanding problems as interpretability stimuli:

**Data-flow group:**
- `Copying`: pure chain-assignment variable tracking (e.g., `u = 7; t = u; d = t`, query `d`)
- `Computing`: chain-assignment with basic arithmetic (e.g., `a = 2; b = a + 1; c = b + 2`, query `c`)

**Control-flow group:**
- `Conditional`: if/else branching (what is the variable value after executing the correct branch?)
- `Loop`: loop-body variable tracking (value after n iterations)
- `Short-circuit`: lazy evaluation in `and`/`or` expressions

All problems are programmatically generated with answers verified by Python interpreter. Answers are constrained to single digits (0–9) for clean Patchscopes signal.

Experiments cover **Qwen2.5-Coder 0.5B / 1.5B / 3B / 7B / 14B**.

---

### III. Patchscopes: Layer-by-Layer MRI

The core methodology is **Patchscopes** (Ghandeharioun et al., 2024): extracting the hidden state $h_\ell$ at layer $\ell$ of the source model and injecting it into a target prompt to probe the information available at that layer.

Our configuration:
- **Source**: code problem prompt; extract last-position $h_\ell$
- **Target**: `"# The value of x is \""` — a standardized decoding prompt
- **Injection site**: last position of target prompt (key adaptation, see below)
- **Evaluation**: whether the first generated token matches the correct answer

**Three methodological adaptations for code tasks:**

**M1: Last-position injection is mandatory.** Comparison:

| Prompt | Injection | Final Acc | Ever Correct |
|---|---|---|---|
| `# The value of x is "` | last-pos | **79%** | **100%** |
| `0→0; 1→1; x→` | x-pos | 0% | 13% |
| `# x=` | x-pos | 13% | 20% |

x-position injection fails because subsequent tokens' KV caches are computed from the original source context, not from the injected $h_\ell$, creating information contamination. Last-position injection feeds directly into generation, with no subsequent tokens to interfere.

**M2: Patchscopes decoding capability is task-dependent.** For high-density Computing tasks, there is a large gap between the peak information detectable by linear Probing and what Patchscopes can decode (sometimes 50+ pp), indicating that information *existing* in the hidden state does not imply it is in *output-ready* format.

**M3: Large models require first-token correction.** On 14B, we observed a counterintuitive reversal (Copying instability > Computing). Investigation revealed 79.7% of samples were suffering from generation momentum — the model continued generating text past the quote character rather than outputting a single digit. First-token evaluation restores expected behavior.

---

### IV. Core Finding: Two Fundamentally Different Processing Dynamics

Complete profile for the 7B model across all five tasks:

| Dimension | Copying | Computing | Conditional | Loop | Short-circuit |
|---|---|---|---|---|---|
| Final Acc | 79% | **18.6%** | 81.7% | 51.6% | 85% |
| Ever Correct | 100% | 55.8% | 90% | 59.4% | 95.6% |
| Overthinking Gap | 21pp | **37.2pp** | 8.3pp | 7.8pp | 10.6pp |
| Instability | ≈0 | **0.356** | 0.22 | 0.03 | 0.04 |
| FCL Position | Late (70-80%) | Early (50-60%) | Mid | Mid | Mid |
| Curve Shape | Step-wise | Flickering | Progressive | Step-wise | Progressive |

*Overthinking Gap = Ever Correct − Final Acc; Instability = fraction of layers between FCL onset and final stable prediction where the output deviates from the final answer.*

**The split within the data-flow group is most striking.** Two tasks both categorized as "data-flow" exhibit opposite dynamics:
- `Copying` has no signal in early layers (FCL at 70-80%), but is rock-stable once it appears (instability ≈ 0);
- `Computing` emerges very early (FCL at 50-60%), but is highly volatile (instability = 0.356).

**The control-flow group is consistent:** low instability (0.03-0.22), overthinking gap within 10pp for all three tasks.

<!-- FIGURE 1 PLACEHOLDER -->
*[Figure 1: Layer-wise accuracy curves for all five tasks on the 7B model. X-axis: normalized layer depth (0–1). Y-axis: accuracy. Highlight the step-wise jump of Copying in late layers; the flickering/oscillating behavior of Computing in mid-layers; the smooth progressive rise of Conditional.]*

**Key ablation — this is not a difficulty artifact:**

Difficulty-Matched experiment: Computing with `d=0` (pure assignment chains, no arithmetic) vs. Copying (structurally near-equivalent):

- `Copying` (7B): Final Acc = 79%, Ever Correct = 100%
- `Computing (d=0)` (7B): Final Acc = 29.2%, Ever Correct = 84.2%

Even with equivalent structural difficulty, Computing format produces worse and more unstable processing. The gap is not driven by computation complexity — it is driven by the code *format* activating a different processing pathway.

---

### V. Overthinking: Computed Correctly, Then Forgot

On Computing tasks, **55.8% of 7B model samples produced the correct answer at some intermediate layer** — yet only 18.6% maintained correctness to the final layer. **37.2 percentage points** of correct answers were overwritten during later processing.

This is not an isolated outlier. Across all 25 (Model, Task) combinations, 25/25 have an exit layer that outperforms the final layer (Binomial test, $p < 10^{-7}$). Overthinking is ubiquitous.

<!-- FIGURE 2 PLACEHOLDER -->
*[Figure 2: Overthinking Gap heatmap. Rows: 5 model sizes (0.5B–14B). Columns: 5 tasks. Cell value: Ever Correct − Final Acc (pp). Expected: 7B Computing is the deepest red (~37pp); 14B row is consistently lighter across all tasks.]*

The distribution of overthinking gap is highly non-uniform:

- **Control-flow group** (Conditional / Loop / Short-circuit): gap uniformly ≤ 10pp
- **Computing**: 7B gap = 37.2pp, approximately 4.5× the same model's Conditional gap
- **14B**: gap narrows to 3.3–11.1pp across all tasks — larger models overthink less

---

### VI. Information Brewing: Knowing vs. Being Able to Say

Why is Computing's overthinking so catastrophic? Linear Probing experiments provide a structural explanation.

We train a LogisticRegression probe on last-position hidden states at each layer to predict the correct answer. Comparing Probing peak accuracy against Patchscopes final accuracy:

**Copying (7B):**
- Probing Peak: 100% @ L22
- PS Final Acc: 79%
- Gap: ~1pp

**Computing d=0 (7B):**
- Probing Peak: 100%
- PS Final Acc: 29.2%
- Gap: **~70.8pp**

**Computing (14B), most extreme:**
- Probing onset: Layer 2 (linearly decodable in the first few layers)
- PS decoding onset: Layer 33
- **Gap: 31 layers**

This 31-layer gap means: the answer information exists in the hidden state in a linearly separable form (Probing can read it), but is *not* encoded in a format that can be directly output — the model requires dozens of additional layers to "translate" it into output-ready format. We call this process **Information Brewing**.

<!-- FIGURE 3 PLACEHOLDER -->
*[Figure 3: Two panels for 14B model — Copying (top) and Computing (bottom). Each shows Probing accuracy curve vs Patchscopes accuracy curve overlaid. Computing panel should show a wide horizontal gap (~31 layers) between where Probing rises steeply and where Patchscopes finally takes off, illustrating the brewing gap.]*

This explains Computing's extreme overthinking: arithmetic intermediate results are encoded in a non-output-friendly format. The subsequent "translation" layers are highly fragile — the still-forming output representation can easily be overwritten by attention updates, causing the model to "forget" the correct answer it briefly held.

**Copying has near-zero brewing gap:** once variable tracking completes, information is already in output-ready format with no translation required.

---

### VII. Loop's Pseudo-iteration Understanding

Loop tasks reveal a separate interesting finding:

| Iterations | 0.5B | 1.5B | 3B | 7B | 14B |
|---|---|---|---|---|---|
| iter = 1 | High | **98.8%** | **95%** | **100%** | **100%** |
| iter = 2 | Low | 15% | 16.2% | 41.2% | 73.8% |
| iter = 3 | Low | 27.5% | 25% | 30% | 76.2% |
| iter = 4 | Low | 13.8% | 31.2% | 35% | **100%** |

The cliff-edge collapse at `iter > 1` is statistically significant (Fisher exact test, $p < 0.03$).

Unrolled Loop experiment: rewriting the loop as sequential plain assignments. The 7B model improves by **+70pp** on `iter=3` after unrolling. This indicates: **the high accuracy at `iter=1` does not reflect genuine loop comprehension** — it reflects pattern matching that routes through the Copying data-flow circuit. A single-iteration loop is semantically near-equivalent to a direct assignment. Once iteration count increases, the Copying circuit fails and the model's lack of true iterative reasoning is exposed.

---

### VIII. Task-Aware Early Exit

The above findings have a direct practical implication: if the model "already has the answer" at some intermediate layer but hasn't yet suffered overthinking, can we stop there?

**Best Fixed Layer Exit** (most practically valuable strategy): use a validation set to identify the optimal fixed exit layer per task. No oracle knowledge required.

7B model results:

| Task | Final Acc | Best Exit Layer | Exit Acc | Gain |
|---|---|---|---|---|
| Copying | 79% | L23 | **99%** | +20pp |
| Computing | 18.6% | L22 | **45%** | +26.4pp |
| Conditional | 81.7% | L26 | 82.7% | +1pp |
| Loop | 51.6% | L22 | 54.1% | +2.5pp |
| Short-circuit | 85% | L23 | 88.9% | +3.9pp |

The two tasks with the largest gains are the same two with the worst overthinking. Control-flow tasks are already near-optimal at the final layer — early exit offers minimal benefit.

This task-specificity is precisely the value of a *task-aware* strategy. A naively applied single fixed exit layer would help some tasks and hurt others. **25/25 (Model, Task) combinations have a better-than-final exit layer** ($p < 10^{-7}$) — overthinking is not noise.

<!-- FIGURE 4 PLACEHOLDER -->
*[Figure 4: Grouped bar chart for the 7B model. Four conditions per task: Final (baseline), Best Fixed Layer, Correct-k (k=3), Oracle. X-axis: five tasks. Y-axis: accuracy (%). Computing should show the most dramatic improvement from Final (18.6%) to Best Fixed (45%) and Oracle (55.8%).]*

---

### IX. Implications for Code Agents

These experiments reveal not just isolated LLM behavior, but a structural vulnerability in code agents.

**The fragility of data-flow tracking:** Code agents continuously perform variable tracking and arithmetic inference while resolving issues. Computing's extreme overthinking (37.2pp gap) and information brewing (information exists but not in output-ready format) means that even when the model internally "computed" the correct variable value at some intermediate layer, it has a high probability of losing it before final output. This is not a context window problem — it is intrinsic to the computational pathway.

**The importance of task-type awareness:** The same 7B model processes Conditional (81.7%) and Computing (18.6%) with dramatically different outcomes. Standard benchmarks that report averaged final accuracy conflate these two completely different internal states, masking the true task-specificity of capabilities.

**The loop illusion:** Loop-related code in many agent benchmarks may be processed via the Copying circuit (pattern matching for single-iteration equivalents), rather than genuine iterative reasoning. The performance cliff at `iter > 1` may be even steeper in complex real-world codebases.

---

### X. Limitations and Open Questions

**Current limitations:**
- Experiments limited to the Qwen2.5-Coder family; whether other architectures exhibit the same data-flow/control-flow split is unknown
- Analysis at layer-level granularity; circuit-level decomposition (which attention heads and MLP neurons mediate each task type) is future work
- Probing experiments cover only Copying and Computing; control-flow tasks lack cross-validation with probing
- Controlled stimuli; not real-world codebases

**Open questions:**
- Do other model families (Llama, Gemma, CodeLlama, etc.) show the same dynamics?
- In multi-file, long-context real codebases, does the ability split worsen?
- Can we design an adaptive inference router based on layer-wise processing profiles?

---

*Experiments based on Patchscopes (Ghandeharioun et al., 2024). Tasks designed as controlled stimuli analogous to the neuroscience paradigm of using minimal stimuli to isolate specific cognitive primitives.*

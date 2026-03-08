---
layout: default
title: "Your LLM Understood the Code — Then Forgot the Answer"
date: 2026-03-09
permalink: /blog/code_understanding_llm/
categories: [Research, Interpretability, Code]
excerpt: "Patchscopes 揭示代码 LLM 内部处理动态差异，以及这对 Code Agent 意味着什么 | Patchscopes reveals dynamic differences in internal processing of Code LLMs."
---

<style>
/* Custom Styles for Technical Blog Integration */
.tech-blog-container {
    max-width: 850px;
    margin: 0 auto;
    font-family: 'Inter', -apple-system, system-ui, sans-serif;
    line-height: 1.7;
}
.tech-header {
    text-align: center;
    margin-bottom: 40px;
}
.tech-title {
    font-size: 2.5em;
    font-weight: 800;
    margin-bottom: 10px;
    background: linear-gradient(135deg, #6e8efb, #a777e3);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}
.tech-subtitle {
    font-size: 1.25em;
    font-weight: 300;
    color: var(--text-muted, #768390);
}
.drop-cap:first-letter {
    font-size: 3em;
    float: left;
    margin-top: 0.15em;
    margin-right: 0.1em;
    line-height: 0.65;
    font-weight: bold;
    color: #a777e3;
}
.insight-box {
    background: var(--bg-surface-2, rgba(45, 51, 59, 0.4));
    border-left: 4px solid #6e8efb;
    padding: 1.5rem;
    margin: 2rem 0;
    border-radius: 0 8px 8px 0;
}
.insight-title {
    font-weight: 700;
    margin-bottom: 10px;
}
.data-highlight {
    font-size: 1.8em;
    font-weight: 700;
    color: #e06c75;
}
.tech-image {
    width: 100%;
    border-radius: 12px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.2);
    margin: 30px 0 10px 0;
}
.img-placeholder {
    width: 100%;
    height: 350px;
    background: repeating-linear-gradient(
        45deg,
        rgba(255, 255, 255, 0.05),
        rgba(255, 255, 255, 0.05) 10px,
        rgba(255, 255, 255, 0.02) 10px,
        rgba(255, 255, 255, 0.02) 20px
    );
    border: 2px dashed rgba(255, 255, 255, 0.2);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: var(--text-muted, #adbac7);
    border-radius: 12px;
    margin: 30px 0 10px 0;
    font-family: monospace;
}
.img-placeholder i { font-size: 3em; margin-bottom: 10px; opacity: 0.5; }
figcaption {
    text-align: center;
    font-size: 0.9em;
    color: var(--text-muted, #768390);
    margin-bottom: 30px;
    font-style: italic;
    display: block;
}
.styled-table {
    width: 100%;
    border-collapse: collapse;
    margin: 25px 0;
    font-size: 0.95em;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}
.styled-table thead tr {
    background-color: #2d333b;
    color: #c9d1d9;
    text-align: left;
}
.styled-table th, .styled-table td { padding: 12px 15px; }
.styled-table tbody tr { border-bottom: 1px solid #3e4451; }
.styled-table tbody tr:nth-of-type(even) { background-color: rgba(255, 255, 255, 0.02); }
.styled-table tbody tr:last-of-type { border-bottom: 2px solid #6e8efb; }
.code-inline { background: rgba(110, 142, 251, 0.1); color: #6e8efb; padding: 2px 6px; border-radius: 4px; font-family: monospace; }
/* Language Toggle CSS */
.lang-toggle-container {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 20px;
}
.lang-btn {
    background: transparent;
    border: 1px solid #6e8efb;
    color: var(--text-color, #c9d1d9);
    padding: 6px 14px;
    margin-left: 10px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9em;
    font-weight: 600;
    transition: all 0.2s ease;
}
.lang-btn.active {
    background: #6e8efb;
    color: #fff;
}
</style>

<div class="tech-blog-container">

<div class="lang-toggle-container">
    <button class="lang-btn active" onclick="switchLang('zh')">中文</button>
    <button class="lang-btn" onclick="switchLang('en')">English</button>
</div>

<!-- ================= 中文版 ================= -->
<div class="lang-zh lang-content" markdown="1">

<div class="tech-header">
    <h1 class="tech-title">您的模型看懂了代码，但却在最后忘了答案</h1>
    <p class="tech-subtitle">Patchscopes 揭示代码 LLM 内部处理动态差异，以及这对 Code Agent 意味着什么</p>
</div>

## 一、Hook：同一个模型，判若两人

<p class="drop-cap">在最新的 Qwen2.5-Coder 7B 模型上，我们观测到了一个极度反直觉的现象：对于同样难度的代码切片，模型的预测准确率在不同的任务类型上展现出了天壤之别。</p>

当模型处理 **Computing（数据流算术追踪）** 任务时，最终准确率仅有惨淡的 **<span style="color:#e06c75">18.6%</span>**；但当面对 **Conditional（控制流逻辑判断）** 任务时，准确率却高达 **<span style="color:#98c379">81.7%</span>**。

**这不是简单的“题目难度差异” —— 这暗示着模型内部能力的完全异构。**

当我们把视线放到真实的开发场景（如 SWE-bench 或是日常使用 Claude Code、Cursor 时），Code Agent 在 resolve 一个 issue 时本质上也在反复进行这两项基础操作：
1. **追踪数据流**（例如：*这个变量经过多层赋值和计算后，到这里值是什么？*）
2. **推理控制流**（例如：*如果条件为真，代码会走哪个分支？循环具体执行几次？*）

这引出了我们的核心疑问：**这两种能力在 LLM 内部究竟是如何运作的？它们的失败模式是否也截然不同？**

---

## 二、任务设计：从 Agent 场景到可控实验

为了回答这些问题，我们需要拆解 Code Agent 的真实操作。我们将它们映射到五类可控的实验任务：

- **数据流追踪**：<span class="code-inline">Copying</span>（纯变量赋值追踪）与 <span class="code-inline">Computing</span>（变量追踪+基本算术）。
- **控制流推理**：<span class="code-inline">Conditional</span>（条件判断）、<span class="code-inline">Loop</span>（循环）、<span class="code-inline">Short-circuit</span>（短路求值）。

为什么必须拆分成这五个极其简单的形式？因为在 Interpretability（机制可解释性）的研究中，我们需要 **隔离单一变量、提供确定性的 ground truth，并保证单 token 的答案**，从而获得最干净的信号分析模型内部的动态变化，而不是受到大模型自回归解码时带来的幻觉干扰。

<img src="assets/progress.png" class="tech-image" alt="Agent scenario mapping">
<figcaption>图1：左侧为实际 agent 的代码逻辑，右侧为对应提纯出的单步 Copying / Conditional 测试用例（答案均设计为 0-9 的单一数字）。</figcaption>

对于本次评估，我们覆盖了 Qwen2.5-Coder 家族从 **0.5B 到 14B** 的全部参数量级。

---

## 三、工具：用 Patchscopes 给模型做逐层 MRI

要弄清楚黑盒里的猫发生了什么，仅看长出的爪子（Final Output）是不够的。我们使用了 **Patchscopes** 工具对模型进行逐层扫描。

**一句话核心原理：** 我们逐层提取源模型的 hidden state $\mathbf{h}_\ell$，将其注入到另一个 Target Prompt（例如 <span class="code-inline">"# The value of x is \""</span>）中去，然后直接逼促模型给出这一层的生成结果：“这一层，你算出答案了吗？”

在代码能力研究场景下，我们对工具进行了特化适配：
1. **关键约束：必须使用 Last-position 注入。** 如果在中间位置注入，会导致后续 token 的 KV cache 基于原始状态计算，造成严峻的信息污染（实验证实 `x-pos` 注入准确率近乎为 0）。
2. **解码能力的 Task-Dependent。** Patchscopes 不是万能工具，它能否顺利提取出信息，高度取决于代码任务的运算密度配置。
3. **大模型 First-token 校正。** 在 14B 模型上发现，生成惯性太大会导致幻觉续写，必须截断使用 First-token evaluation。

<img src="assets/patchscopes.png" class="tech-image" alt="Patchscopes extraction process">
<figcaption>图2：源模型在计算代码切块时，逐层隐状态被提取并 Patch 到探测 prompt 的对应位置，直观展示各层知识成熟状态。</figcaption>

---

## 四、发现1：两大类任务，两种截然不同的“脑电图”

探针深入模型后，我们通过收集各层准确率（Layer-wise Accuracy），得到了两组截然不同的处理时序特征。

### 控制流组：稳步递进
对于 Conditional、Loop、Short-circuit 等任务：
- 它们的内部表征高度连贯，**预测不稳定性（Instability）极低**（仅在 0.03-0.22 之间）。一旦在某一层找到了答案，在后续层中很少翻车。
- 7B 模型在这组上表现卓越：Conditional (81.7%)、Short-circuit (85%)。
- **有趣的是 Loop 行为：** $iter=1$ 时几乎全对（100%），但只要 $iter>1$ 就大崩溃。模型并不是真正理解了循环的控制概念，对于单次迭代它只是通过 Pattern Matching 在走最基本的**数据流 Copy** 回路。

### 数据流组：极度分裂
这组内部展现出两种极其不同的人格（纯拷贝 vs 计算）：
- <span class="code-inline">Copying</span> 纯拷贝任务：呈现 **阶跃式成熟**。Instability 近乎为 0。它的 First Correct Layer (FCL) 极晚（约处理进程的 70-80% 处），但它**一旦到达，就稳如磐石**。
- <span class="code-inline">Computing</span> 算术追踪任务：呈现 **闪烁式跳跃**。它的 FCL 反而出现得很早（约 50-60%处），但**极度不稳定**（Instability 达 0.356）。

<div class="insight-box">
    <div class="insight-title">💡 关键证据: 格式引发的处理质变 (Difficulty-Matched)</div>
    即使退化到 <span class="code-inline">d=0</span>（纯赋值，无算术运算）的 Computing 格式，7B 模型的表现依然比 Copying 任务差 <b>15.8 个百分点</b>。这表明导致崩溃的原因<b>不是绝对难度上升，而是"计算"的代码格式本身就激活了更脆弱、更不稳定的处理链路。</b>
</div>

<img src="assets/findings.png" class="tech-image" alt="Layer-wise accuracy evolution">
<figcaption>图3：五个代码任务的逐层准确率演化折线。注意 Copying 后期的阶跃陡升与 Computing 在中期的剧烈震荡（闪烁）。</figcaption>

---

## 五、发现2：Overthinking——算对了又忘了

在分析 Computing 的失败链时，一个最令人痛心的现象是 **Overthinking**。

<div class="insight-box">
    <span class="data-highlight">37.2 pp</span>
    <p style="margin-top: 10px;">在 7B 模型中，有高达 55.8% 的测试例在模型<b>中间某一层</b>已经被完美计算出了正确答案。但最终能一路保持正确走到最后输出层的，仅剩下了 18.6%。</p>
</div>

接近 **37 个百分点** 的正确结果，在模型加深思考的最后阶段被“想没了”。这构成了本次测试体系中最大的 Overthinking Gap。
- **控制流组**：Gap 都温和保持在 10pp 级别内。
- **数据流组（计算）**：灾难性的 37.2pp Gap，完全是另一种量级的失败。

我们通过统计验证了 25/25 组 $\text{(Model, Task)}$ 组合，一致确认存在着比最终层结果显著更好的“完美退出层”（Binomial test $p<10^{-7}$）。

<div class="img-placeholder">
    <i>🔥</i>
    <span>[占位图] 图4：Overthinking Gap 热力图 (5 Model × 5 Task)</span>
</div>
<figcaption>7B 模型的 Computing 呈现深红（极端遗忘），而大参数量 14B 在各任务上的 Overthinking 现象相对缓解。</figcaption>

---

## 六、发现3：Information Brewing——知道与能说出来是两回事

为什么 Computing 任务会出现如此离谱的波动与遗忘？我们引入纯线性空间分类探针（Probing）进行对比剖析。

- **对于 Copying**：Probing 的 Peak 准确率层与 Patchscopes 能够解码出来的层，差距近乎为零（Gap $\approx$ 0pp）。**信息一旦存在，就是可输出（Output-Ready）格式的。**
- **对于 Computing**：Gap 触目惊心。在 14B 模型中，Probing 探针在极浅层面（Layer 2）就已经能以近 100% 准确率读出最终答案；但对于 Patchscopes 等生成范式，必须直到极其深邃的层（Layer 33）才能把最终结果真正 Decode 出来。

我们将这两者间的 **31 层差距** 称为 **"Information Brewing"（信息酝酿期）**。
这很好地解释了 Computing 的极端不稳定：模型在极浅层计算出了算术特征并编码进 Hidden state 中，但该表征是一种**对自回归解码及其不友好的格式**。模型强行花费数十层对其进行翻转与“翻译”，此过程中由于特征非常脆弱，很容易被后续注意力的噪声覆盖（即发生 Overthinking）。

<img src="assets/method.png" class="tech-image" alt="Probing vs Patchscopes comparison">
<figcaption>图5：Probing渐进上升 vs Patchscopes阶跃跳变的重叠对比。两者的宽阔带揭示了 “信息酝酿” 的过程。</figcaption>

---

## 七、So What：Task-Aware Early Exit

既然模型在深层陷入了过度计算的自毁倾向，我们能否进行外科干预？
因为 Overthinking 展现出的高度 Task-specific，这意味着存在着极具价值的 **Task-Aware Early Exit（任务感知提前退出）** 策略：基于任务种类动态熔断推理。

我们无需做 Oracle 级别的作弊预测，也不需要修改架构，只需基于 Validation set 找到每种任务的**最佳静态熔断层（Best Fixed Layer）**：

<table class="styled-table">
    <thead>
        <tr>
            <th>Task (7B Coder)</th>
            <th>Final 层原生准确率</th>
            <th>Best Fixed 熔断层</th>
            <th>熔断后表现</th>
            <th>净收益 Gain</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Copying</td>
            <td>79.0%</td>
            <td>L23</td>
            <td><strong>99.0%</strong></td>
            <td><span style="color:#98c379"><b>+20.0 pp</b></span></td>
        </tr>
        <tr>
            <td>Computing</td>
            <td>18.6%</td>
            <td>L22</td>
            <td><strong>45.0%</strong></td>
            <td><span style="color:#98c379"><b>+26.4 pp</b></span></td>
        </tr>
        <tr>
            <td>Conditional</td>
            <td>81.7%</td>
            <td>L26</td>
            <td><strong>82.7%</strong></td>
            <td>+1.0 pp</td>
        </tr>
    </tbody>
</table>

<div class="img-placeholder">
    <i>📊</i>
    <span>[占位图] 图6：Early-Exit 策略对比柱状图 (Final vs Best Fixed vs Oracle)</span>
</div>
<figcaption>7B 模型各项任务在最终输出、最佳静态层融断与巅峰神谕状态下的预测表现。</figcaption>

---

## 八、Insights：这对 Code Agent 意味着什么

通过揭开模型的“颅骨”，我们发现 Code LLM 并不是一条完美均质的流水线：

1. **“快思考”与“慢思考”的冲突：** 处理简单的条件判定（控制流）与追踪变量复制，模型有非常稳定、成熟的路由链路特征；但一旦要求它去实时追踪并运算结果（Computing），模型必须使用特殊的、极不稳定的“翻译转换”通道，这直接导致了灾难性的遗忘。
2. **Context 不是万能药：** 很多时候你的 SWE-bench agent 做错决定，不是因为它没有在 Context 中包含该数值，**而是数值计算链路本身的极端脆弱导致了该特征在深层被注意力噪声覆盖（Overthinking）**。
3. **Adaptive 推理路由潜力：** 了解了 Information Processing Profile 后，我们或许可以为 Code Agent 植入类似 `correct-k` 的监控拦截器。如果在追踪超长控制流时，模型某几层内部呈现强烈一致性反馈，Agent 框架甚至可以强制早期提取避免“想岔路”。

---

## 九、讨论与展望

当然，我们的探针也有其**局限性**。我们在实验中刻意剥离并净化出的 Controlled stimuli （单行逻辑小样本片段），尚不可完全类比一个 10k token 规模的深层代码库。当前的颗粒度停留在 Layer-level，并未精确到单个 Head 或 Neuron circuit 的解剖，而这些往往是未来更加深入探究计算流（Attention vs MLP 通道）差异的原点。

但它带给我们不可忽视的底层启发是：当我们凝视一份完美的 Code Benchmark 放榜测试集时，我们看到的是单薄的 “Final Accuracy”。但在这个数字背后，是阶跃、闪烁、在深层不断生灭又重启的信息链路。

**你的大模型确实理解过你的代码，它只是不小心在最后一层忘了告诉你。**

</div>


<!-- ================= 英文版 ================= -->
<div class="lang-en lang-content" style="display:none;" markdown="1">

<div class="tech-header">
    <h1 class="tech-title">Your LLM Understood the Code — Then Forgot the Answer</h1>
    <p class="tech-subtitle">Patchscopes reveals dynamic differences in the internal processing of Code LLMs, and what it implies for Code Agents</p>
</div>

## I. Hook: Same Model, Two Different Personas

<p class="drop-cap">On the latest Qwen2.5-Coder 7B model, we observed a highly counter-intuitive phenomenon: for the exact same level of code snippet difficulty, the model's prediction accuracy exhibits a stark contrast across different task types.</p>

When processing **Computing (data-flow arithmetic tracking)** tasks, the final accuracy is a dismal **<span style="color:#e06c75">18.6%</span>**; but when facing **Conditional (control-flow logic reasoning)** tasks, the accuracy soars to **<span style="color:#98c379">81.7%</span>**.

**This is not simply a "difficulty difference" — it implies completely heterogeneous internal capabilities within the model.**

When we look at real-world development scenarios (like SWE-bench, or using Claude Code and Cursor daily), a Code Agent conceptually repeats two fundamental operations when resolving an issue:
1. **Tracking data flow** (e.g., *What is the value of this variable after multiple layers of assignment and computation?*)
2. **Reasoning control flow** (e.g., *If the condition is true, which branch will the code take? How many times will this loop execute?*)

This leads to our core question: **How exactly do these two capabilities operate inside an LLM? Are their failure modes completely different as well?**

---

## II. Task Design: From Agent Scenarios to Controlled Experiments

To answer these questions, we need to map the real-world operations of Code Agents to five categories of controlled experimental tasks:

- **Data-flow tracking**: <span class="code-inline">Copying</span> (pure variable tracking) and <span class="code-inline">Computing</span> (variable tracking + basic arithmetic).
- **Control-flow reasoning**: <span class="code-inline">Conditional</span> (if-else logic), <span class="code-inline">Loop</span> (loops), <span class="code-inline">Short-circuit</span> (short-circuit evaluation).

Why do we need to break them down into such extremely simple forms? Because in Interpretability research, we must **isolate single variables, provide deterministic ground truths, and guarantee single-token answers**. This ensures we obtain the cleanest signals to analyze the model's internal dynamics, avoiding the hallucination interference brought by autoregressive decoding in large models.

<img src="assets/progress.png" class="tech-image" alt="Agent scenario mapping">
<figcaption>Figure 1: Left - Long code logic in real agent contexts. Right - Extracted single-step test cases (answers constrained to 0-9 digits).</figcaption>

For this evaluation, we covered the entire parameter spectrum of the Qwen2.5-Coder family from **0.5B to 14B**.

---

## III. Tool: Using Patchscopes for a Layer-by-Layer MRI

To figure out what happens inside the black box, merely looking at the Final Output is not enough. We used **Patchscopes** to scan the model layer by layer.

**Core Principle:** We extract the hidden state $\mathbf{h}_\ell$ layer by layer from the source model, inject it into a Target Prompt (e.g., <span class="code-inline">"# The value of x is \""</span>), and directly prompt the model to generate the result for that layer: "Did you calculate the answer at this layer?"

In the context of studying code capabilities, we specially adapted this tool:
1. **Critical Constraint: Last-position injection.** Injecting at an intermediate position corrupts subsequent tokens' KV caches (experiments proved `x-pos` injection yields near 0% accuracy).
2. **Task-Dependent Decoding Capabilities.** Patchscopes is not a silver bullet; its ability to smoothly extract information heavily depends on the code's computational density.
3. **First-token Correction for Large Models.** We discovered that the 14B model exhibits hallucinated generation due to its immense momentum, necessitating a First-token evaluation cutoff.

<img src="assets/patchscopes.png" class="tech-image" alt="Patchscopes extraction process">
<figcaption>Figure 2: Hidden states are extracted layer by layer and "patched" into a probing prompt, intuitively displaying knowledge maturity.</figcaption>

---

## IV. Finding 1: Two Distinct "EEGs" for Two Task Groups

By probing layer-wise accuracy, we unveiled two entirely different processing temporal characteristics.

### Control-flow tasks: Steady progression
For Conditional, Loop, and Short-circuit tasks:
- Their internal representations are highly coherent, with **extremely low predictive Instability** (0.03-0.22). Once an answer is found at a certain layer, it rarely overturns later.
- The 7B model excels here: Conditional (81.7%), Short-circuit (85%).
- **Interesting Loop Behavior:** It is nearly 100% accurate at $iter=1$, but collapses horribly at $iter>1$. The model doesn't truly grasp the control concept of loops; for a single iteration, it relies on simple Pattern Matching via a **data-flow Copy** circuit.

### Data-flow tasks: Extreme polarity
This group reveals two vastly different personas internally:
- <span class="code-inline">Copying</span>: Exhibits **step-wise maturity**. Instability is practically 0. Its First Correct Layer (FCL) arrives late (around 70-80% depth), but **once it arrives, it's rock solid**.
- <span class="code-inline">Computing</span>: Exhibits **flickering leaps**. Its FCL appears much earlier (around 50-60% depth), but it is **highly unstable** (Instability reaches 0.356).

<div class="insight-box">
    <div class="insight-title">💡 Crucial Evidence: Formatting Changes Processing Nature</div>
    Even when degraded to <span class="code-inline">d=0</span> (pure assignment syntax, no math), the 7B model still performs <b>15.8 percentage points worse</b> than the Copying task. This indicates that the failure is <b>not due to absolute difficulty, but rather the "computing" code format itself activating a more fragile and unstable processing pathway.</b>
</div>

<img src="assets/findings.png" class="tech-image" alt="Layer-wise accuracy evolution">
<figcaption>Figure 3: Layer-wise accuracy evolution. Notice the steep late rise of Copying compared to the violent mid-stage oscillation ("flicker") of Computing.</figcaption>

---

## V. Finding 2: Overthinking — Got It Right, But Forgot

When analyzing Computing's failure chain, the most heartbreaking phenomenon is **Overthinking**.

<div class="insight-box">
    <span class="data-highlight">37.2 pp</span>
    <p style="margin-top: 10px;">In the 7B model, an astonishing 55.8% of test cases had the perfect correct answer already calculated at <b>some intermediate layer</b>! Yet, only 18.6% could maintain that correctness to the final output layer.</p>
</div>

Nearly **37 percentage points** of correct results were "overthought" away during the final stages of the model's processing depth. This creates the largest Overthinking Gap in our testing suite.
- **Control-flow group**: Gaps remain mildly within ~10pp.
- **Data-flow group (Computing)**: A catastrophic 37.2pp Gap.

Through statistical verification across 25/25 $\text{(Model, Task)}$ combinations, we confirmed the consistent existence of an "ideal exit layer" that significantly outperforms the final layer (Binomial test $p<10^{-7}$).

<div class="img-placeholder">
    <i>🔥</i>
    <span>[Placeholder] Figure 4: Overthinking Gap Heat Map</span>
</div>
<figcaption>7B's Computing shows extreme forgetting (deep red), whereas the larger 14B mitigates this.</figcaption>

---

## VI. Finding 3: Information Brewing — Knowing vs Telling

Why does Computing exhibit such absurd volatility and forgetting? We introduced purely linear classification Probing for comparison.

- **For Copying**: The peak accuracy layer of Probing and the layer where Patchscopes can decode identical information have almost zero gap (Gap $\approx$ 0pp). **Once the information exists, it is in an Output-Ready format.**
- **For Computing**: The gap is shocking. In the 14B model, Probing can read the final answer with near 100% accuracy at a very shallow layer (Layer 2); but for generation paradigms like Patchscopes, it must wait until an extremely deep layer (Layer 33) to actually Decode the result.

We refer to this **31-layer gap** as **"Information Brewing"**. 
This perfectly explains Computing's extreme instability: the model calculates arithmetic features in shallow layers and encodes them into the Hidden state, but in a **highly unfriendly format for autoregressive decoding**. The model forces dozens of layers to flip and "translate" it. During this process, because the feature is fragile, it gets easily overwritten by subsequent attention noise (hence, Overthinking).

<img src="assets/method.png" class="tech-image" alt="Probing vs Patchscopes comparison">
<figcaption>Figure 5: Probing shows when information exists; Patchscopes shows when it becomes conveyable. The broad gap between them reveals the "Brewing" phase.</figcaption>

---

## VII. So What: Task-Aware Early Exit

Since the model has a self-destructive tendency towards over-computation in deeper layers, can we perform surgical intervention? 
Because Overthinking is highly Task-specific, this points to a highly valuable **Task-Aware Early Exit** strategy: dynamically short-circuiting inference based on the task type.

We don't need Oracle-level cheating, nor do we need architecture changes. We simply use a validation set to find the **Best Fixed Layer** to halt for each task:

<table class="styled-table">
    <thead>
        <tr>
            <th>Task (7B Coder)</th>
            <th>Final Layer Native Acc</th>
            <th>Best Fixed Exit Layer</th>
            <th>Accuracy at Exit</th>
            <th>Net Gain</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Copying</td>
            <td>79.0%</td>
            <td>L23</td>
            <td><strong>99.0%</strong></td>
            <td><span style="color:#98c379"><b>+20.0 pp</b></span></td>
        </tr>
        <tr>
            <td>Computing</td>
            <td>18.6%</td>
            <td>L22</td>
            <td><strong>45.0%</strong></td>
            <td><span style="color:#98c379"><b>+26.4 pp</b></span></td>
        </tr>
        <tr>
            <td>Conditional</td>
            <td>81.7%</td>
            <td>L26</td>
            <td><strong>82.7%</strong></td>
            <td>+1.0 pp</td>
        </tr>
    </tbody>
</table>

<div class="img-placeholder">
    <i>📊</i>
    <span>[Placeholder] Figure 6: Early-Exit Strategy Comparison</span>
</div>
<figcaption>Performance of various tasks on 7B across Final output, Best Fixed Layer, and Oracle conditions.</figcaption>

---

## VIII. Insights: What this means for Code Agents

By lifting the "skull" of the model, we discover that a Code LLM is not a perfectly homogenous pipeline:

1. **The Conflict between "Fast" and "Slow" Thinking:** When processing simple control flow conditional checks and variable copying, the model has highly stable routing pathways; but once asked to track and compute results in real-time (Computing), it must utilize an extremely unstable "translation" channel, causing catastrophic forgetting.
2. **Context is Not a Silver Bullet:** Often when your SWE-bench agent makes a wrong decision, it's not because the value wasn't present in the context. **It is because the value calculation pathway itself is so fragile that the feature is overwritten by attention noise in deeper layers (Overthinking).**
3. **Adaptive Inference Routing Potential:** By understanding this Information Processing Profile, we could inject `correct-k` monitoring interceptors into Code Agents. If the model exhibits strong internal consensus at intermediate layers when tracking long control flows, the framework might even enforce an early extraction to prevent it from "going astray."

---

## IX. Discussion & Future Work

Naturally, our probes have **limitations**. The controlled stimuli (single-line snippets) we meticulously purified for experiments cannot completely substitute a deep 10k-token codebase. The current granularity rests at the Layer-level, without dissecting down to single Heads or Neuron circuits—which is often the starting point for uncovering the core differences in computational flows (Attention vs MLP channels).

However, the profound insight it brings is this: When we gaze at a pristine Final Accuracy on a Code Benchmark leaderboard, we are looking at a flat number. Behind that number are steps, flickers, and information pathways repeatedly emerging, vanishing, and restarting in the deep layers.

**Your Large Language Model truly understood your code once — it just accidentally forgot to tell you in the final layer.**

</div>
</div>

<script>
function switchLang(lang) {
    document.querySelectorAll('.lang-btn').forEach(btn => btn.classList.remove('active'));
    document.querySelectorAll('.lang-content').forEach(el => el.style.display = 'none');
    
    if (lang === 'zh') {
        document.querySelector('.lang-btn[onclick="switchLang(\\\'zh\\\')"]').classList.add('active');
        document.querySelectorAll('.lang-zh').forEach(el => el.style.display = 'block');
    } else {
        document.querySelector('.lang-btn[onclick="switchLang(\\\'en\\\')"]').classList.add('active');
        document.querySelectorAll('.lang-en').forEach(el => el.style.display = 'block');
    }
}
</script>

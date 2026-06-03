---
layout: default
title: "From Brewing to Resolution: Tracing the Internal Lifecycle of Code Reasoning in LLMs"
date: 2026-06-04
permalink: /blog/code_understanding_llm/
categories: [Research, Interpretability, Code]
excerpt: "LLMs first brew the answer — it becomes linearly recoverable many layers before the model can decode it itself — then diverge into four resolution outcomes: Resolved, Overprocessed, Misresolved, or Unresolved. We trace this lifecycle across six code-reasoning task families and 16 models."
multilang: true
---

<style>
/* Premium Academic / Tech Blog Styling */
:root {
    --post-bg: #ffffff;
    --post-text-primary: #1d1d1f;
    --post-text-secondary: #515154;
    --post-accent-color: #0071e3;
    --post-accent-gradient: linear-gradient(135deg, #0071e3, #4b90ff);
    --post-border-color: rgba(0, 0, 0, 0.08);
    --post-code-bg: #f5f5f7;
    --post-code-color: #ff3b30;
    --post-card-bg: rgba(255, 255, 255, 0.7);
    --post-glass-border: rgba(255, 255, 255, 0.5);
    --post-glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.03);
    --post-table-th-bg: #fafafa;
    --post-em-bg: rgba(0, 113, 227, 0.05);
    --post-caption-bg: #fafafa;
}

[data-theme="dark"] {
    --post-bg: #1e293b;
    --post-text-primary: #ffffff;
    --post-text-secondary: #94a3b8;
    --post-accent-color: #3b82f6;
    --post-accent-gradient: linear-gradient(135deg, #3b82f6, #60a5fa);
    --post-border-color: rgba(255, 255, 255, 0.1);
    --post-code-bg: #0f172a;
    --post-code-color: #fb7185;
    --post-card-bg: rgba(30, 41, 59, 0.7);
    --post-glass-border: rgba(255, 255, 255, 0.1);
    --post-glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.15);
    --post-table-th-bg: #0f172a;
    --post-em-bg: rgba(59, 130, 246, 0.15);
    --post-caption-bg: #0f172a;
}

body {
    background-color: var(--bg-primary, #fafafa);
    transition: background-color 0.3s ease;
}

.post-container {
    max-width: 1200px;
    margin: 3rem auto;
    font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    line-height: 1.8;
    color: var(--post-text-primary);
    padding: 24px;
    background: var(--post-bg);
    border-radius: 20px;
    box-shadow: 0 10px 60px rgba(0, 0, 0, 0.01);
    padding: 4rem;
    transition: all 0.3s ease;
}

[data-theme="dark"] .post-container {
    box-shadow: 0 10px 60px rgba(0, 0, 0, 0.08);
}

.post-header {
    text-align: center;
    margin-bottom: 3.5rem;
    padding-bottom: 2rem;
    border-bottom: 1px solid var(--post-border-color);
    position: relative;
}

.post-header::after {
    content: '';
    position: absolute;
    bottom: -1px;
    left: 50%;
    transform: translateX(-50%);
    width: 60px;
    height: 3px;
    background: var(--post-accent-gradient);
    border-radius: 3px;
}

.post-title {
    font-size: 2.4em;
    font-weight: 800;
    letter-spacing: -0.02em;
    color: var(--post-text-primary);
    margin-bottom: 1rem;
    line-height: 1.25;
    background: var(--post-text-primary);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}

[data-theme="light"] .post-title {
    background: linear-gradient(90deg, #1d1d1f, #434345);
    -webkit-background-clip: text;
}

.post-subtitle {
    font-size: 1.15em;
    color: var(--post-text-secondary);
    font-weight: 500;
    margin-bottom: 1.2rem;
}

.post-meta {
    font-size: 1.0em;
    color: var(--post-text-secondary);
    font-weight: 500;
    margin-top: 1.2rem;
    letter-spacing: 0.02em;
}

.post-meta strong {
    color: var(--post-text-primary);
    font-weight: 700;
}

.author-list {
    font-size: 0.88em;
    color: var(--post-text-secondary);
    margin-top: 0.8rem;
    line-height: 1.6;
}

.post-container h2 {
    font-size: 1.8em;
    font-weight: 700;
    margin-top: 3.5rem;
    margin-bottom: 1.5rem;
    color: var(--post-text-primary);
    position: relative;
    padding-left: 1rem;
}

.post-container h2::before {
    content: '';
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    width: 4px;
    height: 80%;
    background: var(--post-accent-gradient);
    border-radius: 4px;
}

.post-container h3 {
    font-size: 1.4em;
    font-weight: 600;
    margin-top: 2.5rem;
    margin-bottom: 1rem;
    color: var(--post-text-primary);
}

.post-container p {
    margin-bottom: 1.5rem;
    font-size: 1.1em;
    color: var(--post-text-secondary);
}

.post-container em {
    color: var(--post-text-primary);
    font-weight: 600;
    font-style: normal;
    background: var(--post-em-bg);
    padding: 0 4px;
    border-radius: 4px;
}

.post-container strong {
    color: var(--post-text-primary);
    font-weight: 700;
}

.post-container code {
    background-color: var(--post-code-bg);
    border-radius: 6px;
    font-size: 0.85em;
    margin: 0;
    padding: 0.3em 0.5em;
    font-family: "JetBrains Mono", SFMono-Regular, Consolas, Monaco, monospace;
    color: var(--post-code-color);
    border: 1px solid var(--post-border-color);
}

/* Premium Tables */
.academic-table {
    width: 100%;
    margin: 3rem 0;
    border-collapse: separate;
    border-spacing: 0;
    font-size: 1em;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.04);
    border: 1px solid var(--post-border-color);
}
.academic-table th, .academic-table td {
    padding: 16px 20px;
    text-align: left;
    border-bottom: 1px solid var(--post-border-color);
}
.academic-table th {
    font-weight: 700;
    background-color: var(--post-table-th-bg);
    color: var(--post-text-primary);
    text-transform: uppercase;
    font-size: 0.85em;
    letter-spacing: 0.05em;
}
.academic-table tr:last-child td {
    border-bottom: none;
}
.academic-table tbody tr {
    transition: background-color 0.2s ease;
}
.academic-table tbody tr:hover {
    background-color: var(--post-em-bg);
}

/* Figures */
.figure-container {
    margin: 3.5rem 0;
    text-align: center;
}
.figure-img {
    width: 100%;
    max-width: 1000px;
    background: #ffffff;
    border: 1px solid var(--post-border-color);
    box-shadow: var(--post-glass-shadow);
    border-radius: 16px;
    padding: 1.2rem;
    box-sizing: border-box;
    transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), box-shadow 0.3s ease;
}
.figure-img:hover {
    transform: translateY(-5px);
    box-shadow: 0 15px 40px rgba(31, 38, 135, 0.08);
}
[data-theme="dark"] .figure-img:hover {
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.25);
}
.figure-img.narrow {
    max-width: 620px;
}
.figure-caption {
    margin: 1.5rem auto 0;
    max-width: 1000px;
    font-size: 0.95em;
    color: var(--post-text-secondary);
    line-height: 1.6;
    text-align: left;
    border-left: 4px solid var(--post-accent-color);
    padding-left: 1.2rem;
    background: var(--post-caption-bg);
    padding-top: 0.8rem;
    padding-bottom: 0.8rem;
    padding-right: 1rem;
    border-radius: 0 8px 8px 0;
}

/* TL;DR card */
.tldr-box {
    background: var(--post-em-bg);
    border: 1px solid var(--post-border-color);
    border-left: 4px solid var(--post-accent-color);
    border-radius: 0 12px 12px 0;
    padding: 1.5rem 2rem;
    margin: 2.5rem 0;
}
.tldr-box .tldr-title {
    font-weight: 800;
    font-size: 1.05em;
    color: var(--post-accent-color);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 0.8rem;
}
.tldr-box ul {
    margin-bottom: 0 !important;
}

.post-container blockquote {
    padding: 1rem 1.5rem;
    color: var(--post-text-secondary);
    background: linear-gradient(to right, var(--post-em-bg), transparent);
    border-left: 4px solid var(--post-accent-color);
    margin: 2rem 0;
    font-style: italic;
    border-radius: 0 8px 8px 0;
}

.post-container ul, .post-container ol {
    margin-bottom: 1.5rem;
    padding-left: 1.5rem;
    color: var(--post-text-secondary);
    font-size: 1.1em;
}
.post-container li {
    margin-bottom: 0.5rem;
    padding-left: 0.5rem;
}
.post-container li::marker {
    color: var(--post-accent-color);
}

.highlight-num {
    color: var(--post-code-color);
    font-weight: 700;
}
.highlight-success {
    color: #34c759;
    font-weight: 700;
    background: rgba(52, 199, 89, 0.1);
    padding: 2px 8px;
    border-radius: 12px;
}
.highlight-danger {
    color: #ff3b30;
    font-weight: 700;
    background: rgba(255, 59, 48, 0.1);
    padding: 2px 8px;
    border-radius: 12px;
}

/* Outcome badges */
.badge {
    display: inline-block;
    font-weight: 700;
    font-size: 0.85em;
    padding: 2px 10px;
    border-radius: 12px;
    white-space: nowrap;
}
.badge-resolved      { color: #2e7d52; background: rgba(52, 199, 89, 0.12); }
.badge-overprocessed { color: #b25e09; background: rgba(255, 149, 0, 0.12); }
.badge-misresolved   { color: #c22e2e; background: rgba(255, 59, 48, 0.10); }
.badge-unresolved    { color: #5b6470; background: rgba(120, 130, 145, 0.14); }
[data-theme="dark"] .badge-resolved      { color: #6ee7a0; }
[data-theme="dark"] .badge-overprocessed { color: #fbbf6e; }
[data-theme="dark"] .badge-misresolved   { color: #fb8585; }
[data-theme="dark"] .badge-unresolved    { color: #aab4c0; }

/* Smooth Transitions for Tab Switching */
.lang-content {
    animation: postFadeIn 0.4s ease-out;
}
@keyframes postFadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
}
</style>

<div class="post-container">

<!-- ================= 中文版 ================= -->
<div class="lang-zh lang-content" style="display:none;">

<div class="post-header">
    <h1 class="post-title">从酝酿到决议：追踪 LLM 代码推理的内部生命周期</h1>
    <div class="post-subtitle">From Brewing to Resolution: Tracing the Internal Lifecycle of Code Reasoning in LLMs</div>
    <div class="post-meta">
        <strong>Yifu Guo</strong>* &middot; <strong>Siyue Chen</strong>* &middot; et al. &middot; June 2026
    </div>
    <div class="author-list">
        Yifu Guo*, Siyue Chen*, Yuquan Lu, Jiaye Lin, Zishan Xu, Jianbo Lin, Siyu Zhang,<br>
        Cheng Yang, Junxin Li, Yujia Li, Yu Huo, Ruixuan Wang&dagger;<br>
        <em style="background:none;font-weight:500;">* 共同一作 &nbsp;&dagger; 通讯作者</em>
    </div>
</div>

<p>这个博客最早是论文动工时留下的占位符。现在论文终于完成了，这里更新为论文的精简版导读——只讲最关键的发现，完整细节请以论文为准。</p>

<div class="tldr-box">
    <div class="tldr-title">TL;DR</div>
    <ul>
        <li>LLM 做代码推理时存在一个内部生命周期：答案先被<strong>酝酿（Brewing）</strong>出来——在约 14% 深度处就能被线性探测器读出，但模型自己要到约 50% 深度才能解码它。</li>
        <li>之后轨迹分化为四种<strong>决议结果（Resolution Outcomes）</strong>：Resolved、Overprocessed、Misresolved、Unresolved——四类都占据可观比例，整体 Resolved 仅 <span class="highlight-num">41.5%</span>。</li>
        <li>三组因果干预实验（激活补丁、跳层、再注入）证实这四类对应真实的内部计算状态，而不是事后贴的标签。</li>
        <li>跨 Qwen、Llama、DeepSeek 三个家族共 16 个模型（0.5B–14B），酝酿脚手架始终稳定（归一化时长 24–42%），但决议成功率随能力、规模与训练数据变化。</li>
    </ul>
</div>

<h2>一、起点：准确率解释不了的割裂</h2>

<p>同一个模型处理代码时表现出令人费解的不均匀：它能轻松追踪变量赋值，一旦引入算术就变得脆弱；能处理显式的 <code>for</code> 循环，却在<strong>语义完全等价</strong>的展开序列上崩溃。在我们的实验中，Value Tracking（变量追踪）的 Resolved 率为 <span class="highlight-num">70.8%</span>，而只需要顺序算术的 Loop-unrolled（循环展开）只有 <span class="highlight-num">28.0%</span>。</p>

<p>这种差距是标准准确率指标完全看不见的：两个准确率相近的模型，内部可能以完全不同的方式失败。要回答“为什么”，我们需要把问题从“某一层<em>编码</em>了什么信息”推进到：<strong>模型在每一层是否已经把问题解出来了？</strong></p>

<p>答案信息可能早已存在于隐藏状态中，却还没有被组织成模型自己能稳定使用的形式。我们由此区分两个事件：<strong>信息可得（availability）</strong>——答案何时能被外部读出；<strong>信息就绪（readiness）</strong>——答案何时能被模型自己的解码管线使用。</p>

<h2>二、双诊断框架：Probing × CSD</h2>

<p>我们构建了一个覆盖数据流、控制流及其组合的六任务族合成基准（Value Tracking、Computing、Conditional、Function Call、Loop、Loop-unrolled；每模型 24,300 个样本，答案恒为单个数字，保证跨 tokenizer 的单 token 目标），并引入两个逐层诊断函数：</p>

<ul>
    <li><strong>线性探测（Linear Probing）</strong>：在每层隐藏状态上训练逻辑回归分类器，检测答案是否已经<em>线性可恢复</em>——回答 availability。</li>
    <li><strong>上下文剥离解码（Context-Stripped Decoding, CSD）</strong>：受 Patchscopes (Ghandeharioun et al., 2024) 启发，把第 &ell; 层的最后位置隐藏状态注入一个只保留问题后缀、剥掉全部代码上下文的目标 prompt，让模型从 &ell;+1 层继续前向传播。关键改造是<strong>基线 logit 减法</strong>：先在目标 prompt 上跑一遍干净前向得到语言先验 logit，再从补丁后的 logit 中减去，从而分离出隐藏状态本身携带的信号——回答 readiness。</li>
</ul>

<p>如果 CSD 能在剥离代码上下文的情况下解出正确答案，说明这个隐藏状态是<em>自包含</em>的：模型不需要回看代码就能完成解码。两个诊断在深度上一致或分歧的位置，就刻画了代码推理在模型内部展开的过程。</p>

<h2>三、酝酿：信息先于自解码出现</h2>

<p>逐层扫描后，一个稳定的顺序浮现出来。我们定义两个关键层：<strong>FPCL</strong>（First Probe-Correct Layer，探测首次正确的层）和 <strong>FJC</strong>（First Joint-Correct Layer，探测与 CSD 首次同时正确的层）。在锚定设定（Qwen2.5-Coder-7B）下：</p>

<ul>
    <li>FPCL 出现在平均归一化深度 <span class="highlight-num">14%</span> 处——答案很早就线性可读；</li>
    <li>FJC 要到 <span class="highlight-num">50%</span> 深度才出现——模型自己要晚得多才能解码它；</li>
    <li>两者之间的区间就是<strong>酝酿期（Brewing）</strong>，平均跨越 10.7 层（占总深度 <span class="highlight-num">38%</span>），且 FPCL &lt; FJC 的顺序在<strong>所有</strong>实验配置中无一例外。</li>
</ul>

<div class="figure-container">
    <img class="figure-img" src="/blog/code_understanding_llm/assets/resolution_taxonomy.png" alt="Brewing-to-resolution lifecycle">
    <div class="figure-caption">
        <strong>图 1：</strong>酝酿—决议生命周期（Qwen2.5-Coder-7B，Computing 任务）。每个面板是一条样本轨迹：蓝色为 Probing、橙色为 CSD 的逐层数字分布。Probing 先变对，CSD 晚若干层才跟上；FPCL 与 FJC 界定酝酿区间，之后轨迹分化为 (a) Resolved、(b) Overprocessed、(c) Misresolved、(d) Unresolved 四种结局。
    </div>
</div>

<p>这意味着代码推理不是“最后一层的事件”。答案在浅层就已经算出来了，但它存在的形式对自回归生成是“不友好”的；网络要再花超过三分之一的深度把它转换成可输出的格式。<strong>计算答案和重整答案，是 Transformer 深度承担的两件可分离的工作。</strong></p>

<h2>四、四种决议结果：殊途不同归</h2>

<p>酝酿之后，轨迹并不都以同一种方式收敛。依据 FJC 是否存在、最终输出是否正确、以及尾部窗口（最后 1/4 层）的 CSD 置信度，每条轨迹恰好落入四类之一：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>结果</th>
            <th>定义</th>
            <th>占比（锚定设定）</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><span class="badge badge-resolved">Resolved</span></td>
            <td>到达 FJC，且最终输出正确——内部确认并保持到底</td>
            <td><strong>41.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-overprocessed">Overprocessed</span></td>
            <td>到达 FJC，但最终输出错误——正确计算曾经形成，又被后续层<strong>摧毁</strong></td>
            <td><strong>26.4%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-misresolved">Misresolved</span></td>
            <td>从未到达 FJC，但尾部稳定收敛到一个<strong>错误</strong>答案——自信地答错</td>
            <td><strong>8.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span></td>
            <td>从未到达 FJC，尾部也没有稳定决议——在可用深度内<strong>没算完</strong></td>
            <td><strong>23.7%</strong></td>
        </tr>
    </tbody>
</table>

<blockquote>四类全部占据可观比例，没有任何一类可以当作边角料忽略。这套分类法刻画的是内部计算的<strong>常规多样性</strong>，而不是边缘失败案例。</blockquote>

<h2>五、因果干预：这不是事后叙事</h2>

<p>如果 FJC 和四分类对应真实的内部结构，那么针对它们的干预应当产生<strong>可预测的、结果特异的</strong>效应。我们设计了三组实验：</p>

<ul>
    <li><strong>FJC 处的激活补丁</strong>：把源 prompt 的隐藏状态补丁到中性模板里，FJC 处的答案翻转率达 20–44%，而酝酿前层只有 3–18%——FJC 是信息首次可读的因果特权位置。</li>
    <li><strong>对 Overprocessed 跳层</strong>：直接用 FJC 的隐藏状态替换末层只能挽救约 10%，但诊断发现这是表征<strong>范数失配</strong>所致；改用 alpha 混合注入（0.7·原状态 + 0.3·FJC 状态）后，平均 <span class="highlight-success">47.8%</span> 的 Overprocessed 样本恢复正确。两种注入方式的巨大差距本身就证明：FJC 携带正确信息，只是几何形态与深层表征不同。</li>
    <li><strong>对 Unresolved 再注入</strong>：把 FPCL（信息最早可读处）的隐藏状态注入倒数 2–4 层，可挽救 22–38% 的样本，而 Resolved 对照组保持 84–100% 不受损——相当一部分 Unresolved 包含的是“没算完”而非“根本不会”的计算。</li>
</ul>

<p>三组干预各自指向分类法的不同分支，且每组效应都与对应类别一致、与其他类别不一致。进一步的组件归因显示，改写发生在网络末段（28 层中的 22–27 层）：Qwen 家族里由<strong>注意力</strong>主导，Llama/DeepSeek 里由 <strong>MLP</strong> 主导——末段改写跨模型稳定，但执行者是家族特异的。</p>

<h2>六、每种代码原语都有独特的失败指纹</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/fingerprint.png" alt="Task to outcome Sankey and difficulty response">
    <div class="figure-caption">
        <strong>图 2：</strong>(a) 任务→结果的 Sankey 流图（Qwen2.5-Coder-7B），条带宽度与样本比例成正比；(b) 各任务 Resolved 率随难度递增的响应曲线。六个任务中有三个低于 30% Resolved。
    </div>
</div>

<p>总体 41.5% 的 Resolved 率掩盖了任务间的巨大差异。受控难度扫描进一步暴露出每个任务特有的瓶颈：</p>

<ul>
    <li><strong>Computing</strong>：Overprocessed 占比最高（35.6%）——模型提取到了信息，却在内部计算中失稳。算术步数从 2 增到 4，Overprocessed 从 25.4% 爬到 <span class="highlight-num">47.5%</span>。更微妙的是：纯加法的 Misresolved 率（17.5%）远高于加乘混合（6.4%）——越简单的算子，越容易产生自信的错误。</li>
    <li><strong>Function Call</strong>：所有任务中最严重的退化。调用深度从 1 增到 3，Resolved 从 61.1% 崩塌到 <span class="highlight-danger">2.5%</span>，Unresolved 飙到 57.4%。函数调用的间接性是我们观察到的最大单一瓶颈，FPCL 也被推迟 2.4 倍。</li>
    <li><strong>Conditional</strong>：整体尚可（59.2% Resolved），但 <code>boolean_flag</code> 条件的 Misresolved 高达 18.5%（数值比较仅 6.5%）——布尔求值是一个与嵌套深度无关的潜在瓶颈，模型会自信地选错分支。</li>
    <li><strong>Loop vs Loop-unrolled</strong>：最干净的因果对照——算术完全相同，只有语法不同。Loop 的 Resolved（35.5%）反而高于展开版（28.0%），信息可读的时刻也更早。双变量追踪场景下，Unresolved 从 Loop 的 23.5% 翻倍到展开版的 <span class="highlight-num">53.6%</span>：<strong>循环语法本身是一种结构脚手架</strong>，展开后脚手架消失，同样的计算反而更难完成。轨迹配对分析显示，是末段 MLP（25–26 层）在循环版本中更用力地把正确数字写入残差流；把这次 MLP 写入移植给展开版孪生样本，能弥合 82.6% 的就绪差距。</li>
</ul>

<h2>七、跨 16 个模型：脚手架稳定，能力分化</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/brewing_stability.png" alt="Brewing stability across 16 models">
    <div class="figure-caption">
        <strong>图 3：</strong>(a) 16 个模型的归一化 FJC 分布（按家族着色）；(b) FJC 存在率；(c) 归一化酝酿时长。尽管家族间存在差异，全部 16 个模型级均值都落在 24–42% 的窄带内。
    </div>
</div>

<p>把分析从单一模型扩展到 Qwen2.5-Coder、Qwen2.5、Qwen3、DeepSeek-Coder、CodeLlama、Llama-2 共 16 个模型（0.5B–14B），出现了一个清晰的解耦：</p>

<ul>
    <li><strong>机制稳定</strong>：酝酿在 0.5B 的最小模型上就已存在，归一化酝酿时长在所有 16 个模型上都落在 <span class="highlight-num">24–42%</span> 区间。Coder 与同架构 Base 模型的任务级 FJC 位置强相关（Pearson r = 0.901），平均偏移仅 0.67 层。</li>
    <li><strong>能力分化</strong>：规模增大把概率质量从 Unresolved/Misresolved 推向 Resolved（0.5B 的 Resolved 为 18%，14B 为 50.3%）；代码特化训练改善 Computing 上的结果分布；CodeLlama/Llama-2 在 Computing 上 Resolved 仅 7–8%，瓶颈在 CSD 能力而非探测——同架构的 DeepSeek-Coder 达到 21.3%，说明关键差异在<strong>代码预训练数据质量</strong>而非架构。</li>
</ul>

<blockquote>规模扩展不创造新的计算机制，它只是<strong>锐化</strong>已有的机制。训练改变的是模型“解得多好”（能力），远多于改变“决议转变发生在哪里”（机制）。</blockquote>

<h2>八、不看 Ground Truth，也能观察酝酿</h2>

<p>上面的全部分类都依赖真值标签——实际部署中没有这种奢侈。好在生命周期会在模型自己的输出分布上留下可观测的签名：从 CSD 的数字 softmax 中提取<strong>熵</strong>和<strong>最大置信度</strong>两个无真值信号后，尾层熵的持续上升是 Overprocessed 最强的单一探测器（AUC 0.71–0.86）——过度处理的签名是“不确定性上升”，而不只是“置信度下降”。一个仅由端点统计量构成的闭式 Resolution Functional，在 Resolved-vs-Rest 二分类上达到 <span class="highlight-num">AUC 0.850</span>。</p>

<p>这种判别力是真正<strong>双透镜</strong>的：去掉任何一个透镜重新拟合，四分类平衡准确率就从 0.62 跌到约 0.40——最强的特征（探测与 CSD 的 argmax 一致性/分歧度）单靠任何一边都算不出来。</p>

<h2>九、为什么重要：不存在万能的 Early-Exit 策略</h2>

<p>两大主导失败模式提出的是<strong>截然相反</strong>的需求：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>失败模式</th>
            <th>本质</th>
            <th>需要的干预</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><span class="badge badge-overprocessed">Overprocessed</span>（26.4%，Computing 高发）</td>
            <td>末层损毁了已经形成的正确答案</td>
            <td><strong>提早退出</strong>，在黄金期截取答案</td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span>（23.7%，Function Call 高发）</td>
            <td>可用深度内计算没有完成</td>
            <td><strong>更多深度</strong>或再注入早期信息</td>
        </tr>
    </tbody>
</table>

<p>任何单一的 early-exit 策略必然顾此失彼：帮了 Overprocessed 就伤了 Unresolved，反之亦然。这指向<strong>结果感知（outcome-aware）的推理策略</strong>——而 AUC 0.850 的无真值信号说明，运行时监测模型当前处于哪种内部状态，是一条切实可行的路径。</p>

<p>当然也要诚实地交代边界：我们的诊断停留在层粒度，还无法定位是哪些注意力头或 MLP 子层驱动了酝酿—决议转变；基准使用短小的单原语程序，这些动态是否在组合式多语句代码中延续仍是开放问题；以及两个更深的谜：末层<em>为什么</em>会损毁正确答案？“先可得、后就绪”的顺序是残差流几何的必然，还是当前训练范式的偶然？</p>

<h2>引用 (Citation)</h2>

<pre><code>@article{guo2026brewing,
  title  = "From Brewing to Resolution: Tracing the Internal
            Lifecycle of Code Reasoning in LLMs",
  author = "Guo, Yifu and Chen, Siyue and Lu, Yuquan and Lin, Jiaye
            and Xu, Zishan and Lin, Jianbo and Zhang, Siyu and
            Yang, Cheng and Li, Junxin and Li, Yujia and Huo, Yu
            and Wang, Ruixuan",
  year   = "2026"
}</code></pre>

<h2>关键参考文献 (Key References)</h2>
<ol>
    <li>Ghandeharioun, Asma, et al. <a href="https://arxiv.org/abs/2401.06102">"Patchscopes: A unifying framework for inspecting hidden representations of language models."</a> ICML (2024).</li>
    <li>Belinkov, Yonatan. <a href="https://arxiv.org/abs/2102.12452">"Probing classifiers: Promises, shortcomings, and advances."</a> Computational Linguistics (2022).</li>
    <li>Belrose, Nora, et al. <a href="https://arxiv.org/abs/2303.08112">"Eliciting latent predictions from transformers with the tuned lens."</a> arXiv:2303.08112 (2023).</li>
    <li>Halawi, Danny, et al. <a href="https://arxiv.org/abs/2307.09476">"Overthinking the truth: Understanding how language models process false demonstrations."</a> ICLR (2024).</li>
    <li>Meng, Kevin, et al. <a href="https://arxiv.org/abs/2202.05262">"Locating and editing factual associations in GPT."</a> NeurIPS (2022).</li>
    <li>Schuster, Tal, et al. <a href="https://arxiv.org/abs/2207.07061">"Confident adaptive language modeling."</a> NeurIPS (2022).</li>
    <li>Gu, Alex, et al. <a href="https://arxiv.org/abs/2401.03065">"CRUXEval: A benchmark for code reasoning, understanding and execution."</a> ICML (2024).</li>
</ol>

</div>


<!-- ================= 英文版 ================= -->
<div class="lang-en lang-content">

<div class="post-header">
    <h1 class="post-title">From Brewing to Resolution: Tracing the Internal Lifecycle of Code Reasoning in LLMs</h1>
    <div class="post-meta">
        <strong>Yifu Guo</strong>* &middot; <strong>Siyue Chen</strong>* &middot; et al. &middot; June 2026
    </div>
    <div class="author-list">
        Yifu Guo*, Siyue Chen*, Yuquan Lu, Jiaye Lin, Zishan Xu, Jianbo Lin, Siyu Zhang,<br>
        Cheng Yang, Junxin Li, Yujia Li, Yu Huo, Ruixuan Wang&dagger;<br>
        <em style="background:none;font-weight:500;">* Equal contribution &nbsp;&dagger; Corresponding author</em>
    </div>
</div>

<p>This post started life as a placeholder while the paper behind it was still in the making. The paper is now done, so this is its condensed companion — only the key findings, with full details deferred to the paper.</p>

<div class="tldr-box">
    <div class="tldr-title">TL;DR</div>
    <ul>
        <li>Code reasoning in LLMs follows an internal lifecycle: the answer is first <strong>brewed</strong> — linearly recoverable at ~14% of network depth, yet not self-decodable by the model until ~50%.</li>
        <li>Trajectories then diverge into four <strong>resolution outcomes</strong> — Resolved, Overprocessed, Misresolved, Unresolved — and all four carry substantial mass: overall Resolved is only <span class="highlight-num">41.5%</span>.</li>
        <li>Three causal interventions (activation patching, layer skipping, re-injection) confirm the outcomes are intervention-sensitive computational states, not post-hoc labels.</li>
        <li>Across 16 models from the Qwen, Llama, and DeepSeek families (0.5B&ndash;14B), the brewing scaffold stays stable (normalized duration 24&ndash;42%) while resolution success covaries with capability, scale, and training.</li>
    </ul>
</div>

<h2>I. The Puzzle Accuracy Cannot Explain</h2>

<p>A single model exhibits puzzling heterogeneity on code: it effortlessly traces variable assignments yet becomes brittle once arithmetic is introduced; it handles explicit <code>for</code> loops yet struggles with <strong>semantically equivalent</strong> unrolled sequences. In our experiments, Value Tracking achieves <span class="highlight-num">70.8%</span> Resolved while Loop-unrolled — which requires nothing more than sequential arithmetic — reaches only <span class="highlight-num">28.0%</span>.</p>

<p>This gap is invisible to standard accuracy metrics: two models with similar accuracy may fail internally in entirely different ways. To explain it, we must move past asking what information is <em>encoded</em> at a given layer, and ask instead: <strong>has the model already <em>solved</em> the problem at each layer?</strong></p>

<p>Answer information may reside in the hidden state long before it is organized into a form the model can stably use. We therefore distinguish two events: <strong>information availability</strong> — when the answer becomes externally readable — and <strong>information readiness</strong> — when it becomes usable by the model's own decoding pipeline.</p>

<h2>II. The Dual Diagnostic Framework: Probing &times; CSD</h2>

<p>We build a synthetic benchmark of six task families spanning data flow, control flow, and their combination (Value Tracking, Computing, Conditional, Function Call, Loop, Loop-unrolled; 24,300 samples per model, every answer a single digit to guarantee a one-token target across tokenizers), and pair two layer-wise diagnostics:</p>

<ul>
    <li><strong>Linear Probing</strong>: a logistic classifier trained per layer tests whether the answer is already <em>linearly recoverable</em> from the hidden state — this measures availability.</li>
    <li><strong>Context-Stripped Decoding (CSD)</strong>: inspired by Patchscopes (Ghandeharioun et al., 2024), we extract the last-token hidden state at layer &ell; and inject it into a target prompt that keeps only the question suffix — the entire code context is stripped away — letting the model continue its forward pass from layer &ell;+1. The key adaptation is <strong>baseline logit subtraction</strong>: a clean forward pass on the bare target prompt yields the language prior, which we subtract from the patched logits to isolate the signal carried by the hidden state itself. This measures readiness.</li>
</ul>

<p>If CSD produces the correct answer with the code context gone, the hidden state is <em>self-contained</em>: the model's own decoding pipeline can finish the job without ever looking back at the code. Scanning both diagnostics across depth — and tracking where they agree or diverge — reveals how code reasoning unfolds inside the network.</p>

<h2>III. Brewing: Information Precedes Self-Decoding</h2>

<p>A stable ordering emerges from the layer-wise scan. We define two landmark layers: the <strong>FPCL</strong> (First Probe-Correct Layer) and the <strong>FJC</strong> (First Joint-Correct Layer, where probing and CSD are simultaneously correct). In the anchor setting (Qwen2.5-Coder-7B):</p>

<ul>
    <li>FPCL occurs at mean normalized depth <span class="highlight-num">14%</span> — the answer becomes linearly readable remarkably early;</li>
    <li>FJC arrives only at <span class="highlight-num">50%</span> depth — the model itself catches up far later;</li>
    <li>The interval between them is <strong>brewing</strong>: it averages 10.7 layers (<span class="highlight-num">38%</span> of total depth), and the ordering FPCL &lt; FJC holds across <strong>every</strong> experimental configuration.</li>
</ul>

<div class="figure-container">
    <img class="figure-img" src="/blog/code_understanding_llm/assets/resolution_taxonomy.png" alt="Brewing-to-resolution lifecycle">
    <div class="figure-caption">
        <strong>Figure 1:</strong> The brewing-to-resolution lifecycle (Qwen2.5-Coder-7B, Computing). Each panel shows one sample trajectory: blue ridges are layer-wise Probing distributions, orange are CSD. Probing becomes correct first; CSD catches up several layers later. FPCL and FJC bound the brewing interval, after which trajectories diverge into (a) Resolved, (b) Overprocessed, (c) Misresolved, or (d) Unresolved.
    </div>
</div>

<p>Code reasoning is therefore not a last-layer event. The answer is computed in shallow layers, but in a form hostile to autoregressive generation; the network spends more than a third of its depth converting it into an output-ready format. <strong>Computing the answer and reformatting it are two separable functions of transformer depth.</strong></p>

<h2>IV. Four Resolution Outcomes Cover All Trajectories</h2>

<p>After brewing, trajectories do not all converge the same way. Based on whether FJC exists, whether the final output is correct, and the CSD confidence over the tail window (last quarter of layers), every trajectory falls into exactly one of four categories:</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Outcome</th>
            <th>Definition</th>
            <th>Share (anchor)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><span class="badge badge-resolved">Resolved</span></td>
            <td>FJC reached, final output correct — internally attested and preserved</td>
            <td><strong>41.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-overprocessed">Overprocessed</span></td>
            <td>FJC reached, final output wrong — a correct computation formed, then was <strong>destroyed</strong> by later layers</td>
            <td><strong>26.4%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-misresolved">Misresolved</span></td>
            <td>FJC never reached, but the tail converges to a stable <strong>wrong</strong> answer — confidently incorrect</td>
            <td><strong>8.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span></td>
            <td>FJC never reached, no stable resolution in the tail — the computation <strong>never finished</strong> within the available depth</td>
            <td><strong>23.7%</strong></td>
        </tr>
    </tbody>
</table>

<blockquote>All four outcomes carry substantial mass; none can be dismissed as marginal. The taxonomy characterizes the <strong>routine diversity</strong> of internal computation, not edge-case failures.</blockquote>

<h2>V. The Taxonomy Is Causally Real</h2>

<p>If FJC and the four outcomes correspond to genuine internal structure, interventions centered on them should produce <strong>predictable, outcome-specific</strong> effects. Three experiments test this:</p>

<ul>
    <li><strong>Activation patching at FJC</strong>: patching the source hidden state into a neutral template flips the answer at FJC in 20&ndash;44% of cases, versus only 3&ndash;18% at pre-brewing layers — FJC is a causally privileged point where information first becomes readable.</li>
    <li><strong>Layer skipping for Overprocessed</strong>: naively replacing the final hidden state with the FJC state rescues only ~10% — but diagnosis reveals a representation <strong>norm mismatch</strong>. An alpha-blend injection (0.7&middot;original + 0.3&middot;FJC state) restores correctness in <span class="highlight-success">47.8%</span> of Overprocessed samples on average. The gap between injection modes is itself the proof: the FJC state carries the correct information but differs geometrically from late-layer representations.</li>
    <li><strong>Re-injection for Unresolved</strong>: injecting the FPCL hidden state — where information first became readable — into the penultimate layers rescues 22&ndash;38% of samples, while Resolved controls remain at 84&ndash;100%. A substantial fraction of Unresolved samples are "not finished," not "fundamentally incapable."</li>
</ul>

<p>Each intervention targets a different branch of the taxonomy, and each produces effects consistent with its branch and inconsistent with the others. Component attribution further localizes the rewrite to the network's late segment (layers 22&ndash;27 of 28): driven by <strong>attention</strong> in the Qwen family, but by the <strong>MLP</strong> in Llama/DeepSeek — the late-segment rewrite is stable across models, while its responsible component is family-specific.</p>

<h2>VI. Every Code Primitive Has a Unique Failure Fingerprint</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/fingerprint.png" alt="Task to outcome Sankey and difficulty response">
    <div class="figure-caption">
        <strong>Figure 2:</strong> (a) Task&rarr;Outcome Sankey diagram (Qwen2.5-Coder-7B); ribbon width proportional to sample fraction. (b) Resolved% under increasing difficulty for each task. Three of six tasks fall below 30% Resolved.
    </div>
</div>

<p>The aggregate 41.5% Resolved rate masks dramatic variation across tasks, and controlled difficulty sweeps expose bottlenecks specific to each:</p>

<ul>
    <li><strong>Computing</strong>: the largest Overprocessed share (35.6%) — the model extracts the information but destabilizes mid-computation. Increasing arithmetic steps from 2 to 4 drives Overprocessed from 25.4% to <span class="highlight-num">47.5%</span>. Subtler still: pure addition yields a far higher Misresolved rate (17.5%) than mixed add-multiply (6.4%) — simpler operators produce more <em>confident</em> wrong answers.</li>
    <li><strong>Function Call</strong>: the most severe degradation of any task. Call depth 1&rarr;3 collapses Resolved from 61.1% to <span class="highlight-danger">2.5%</span> and drives Unresolved to 57.4%. Function-call indirection is the single largest bottleneck we observe, delaying FPCL by a factor of 2.4&times;.</li>
    <li><strong>Conditional</strong>: solid overall (59.2% Resolved), but <code>boolean_flag</code> conditions yield 18.5% Misresolved (vs. 6.5% for numeric comparison) — Boolean evaluation is a latent bottleneck independent of nesting depth: the model confidently selects the wrong branch.</li>
    <li><strong>Loop vs. Loop-unrolled</strong>: the cleanest causal contrast — identical arithmetic, differing only in syntax. Loop achieves <em>higher</em> Resolved (35.5% vs. 28.0%) and earlier information readability. Under dual-variable tracking, Unresolved doubles from 23.5% (Loop) to <span class="highlight-num">53.6%</span> (unrolled): <strong>loop syntax acts as a structural scaffold</strong>; once unrolled, the cues vanish and the same computation becomes harder. Trajectory-matched analysis shows the late MLP (layers 25&ndash;26) writes the correct digit into the residual stream more forcefully for loops — and transplanting that MLP write into the unrolled twin closes 82.6% of the readiness gap.</li>
</ul>

<h2>VII. Across 16 Models: Stable Scaffold, Divergent Capability</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/brewing_stability.png" alt="Brewing stability across 16 models">
    <div class="figure-caption">
        <strong>Figure 3:</strong> (a) Normalized FJC distributions (KDE) for 16 models, colored by family; (b) FJC existence rates; (c) normalized brewing duration. Despite variation across families, all 16 model-level means fall within a 24&ndash;42% band.
    </div>
</div>

<p>Extending the analysis to 16 models across Qwen2.5-Coder, Qwen2.5, Qwen3, DeepSeek-Coder, CodeLlama, and Llama-2 (0.5B&ndash;14B) reveals a clean dissociation:</p>

<ul>
    <li><strong>The mechanism is stable</strong>: brewing is already present at 0.5B, and normalized brewing duration falls within <span class="highlight-num">24&ndash;42%</span> for all 16 models. Coder and Base variants of the same architecture place their task-level FJC positions almost identically (Pearson r = 0.901; mean shift of only 0.67 layers).</li>
    <li><strong>Capability diverges</strong>: scale shifts mass from Unresolved/Misresolved toward Resolved (0.5B: 18% Resolved &rarr; 14B: 50.3%); code-specialized training improves the Computing outcome mix; CodeLlama/Llama-2 collapse on Computing (7&ndash;8% Resolved) with the bottleneck in CSD capability rather than probing — while DeepSeek-Coder, on the <em>same architecture</em>, reaches 21.3%, pointing to <strong>code pretraining data quality</strong>, not architecture, as the differentiator.</li>
</ul>

<blockquote>Scaling does not create a new computational mechanism; it <strong>sharpens</strong> an existing one. Training changes how <em>well</em> the model resolves tasks (capability) far more than it changes <em>where</em> the resolution transition occurs (mechanism).</blockquote>

<h2>VIII. Watching Brewing Without Ground Truth</h2>

<p>Everything above relies on ground-truth labels — a luxury deployment doesn't have. Fortunately, the lifecycle leaves observable signatures in the model's own output distributions. From the CSD digit softmax we derive two GT-free signals — <strong>entropy</strong> and <strong>max-confidence</strong> — and find that a sustained tail-layer entropy rise is the strongest single detector of Overprocessed samples (AUC 0.71&ndash;0.86): the signature of overprocessing is <em>rising uncertainty</em>, not merely falling confidence. A closed-form Resolution Functional built from endpoint statistics alone achieves a Resolved-vs-Rest <span class="highlight-num">AUC of 0.850</span>.</p>

<p>This discriminative power is genuinely <strong>dual</strong>: remove either lens and refit, and four-class balanced accuracy collapses from 0.62 to about 0.40 — the strongest features (probe&ndash;CSD argmax agreement and divergence) simply cannot be computed from one lens alone.</p>

<h2>IX. Why It Matters: No Single Early-Exit Policy Can Work</h2>

<p>The two dominant failure modes make <strong>opposite</strong> demands:</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Failure mode</th>
            <th>Nature</th>
            <th>Required intervention</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><span class="badge badge-overprocessed">Overprocessed</span> (26.4%, peaks on Computing)</td>
            <td>Late layers corrupt an already-correct answer</td>
            <td><strong>Exit early</strong>, harvest the answer in its golden window</td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span> (23.7%, peaks on Function Call)</td>
            <td>Computation never finished within available depth</td>
            <td><strong>More depth</strong>, or re-injection of early information</td>
        </tr>
    </tbody>
</table>

<p>Any single early-exit policy is therefore self-defeating: whatever helps Overprocessed hurts Unresolved, and vice versa. This motivates <strong>outcome-aware inference</strong> — and the 0.850 AUC of the GT-free signals suggests that monitoring which internal state the model currently occupies is a practical path, not a fantasy.</p>

<p>Honest boundaries, too: our diagnostics operate at layer granularity and cannot yet pinpoint which attention heads or MLP sublayers drive the brewing-to-resolution transition; the benchmark uses short single-primitive programs, so whether these dynamics persist in compositional multi-statement code remains open; and two deeper mysteries stand — <em>why</em> do late layers corrupt correct answers, and is availability-before-readiness a necessary consequence of residual-stream geometry or a contingent property of current training regimes?</p>

<h2>Citation</h2>

<pre><code>@article{guo2026brewing,
  title  = "From Brewing to Resolution: Tracing the Internal
            Lifecycle of Code Reasoning in LLMs",
  author = "Guo, Yifu and Chen, Siyue and Lu, Yuquan and Lin, Jiaye
            and Xu, Zishan and Lin, Jianbo and Zhang, Siyu and
            Yang, Cheng and Li, Junxin and Li, Yujia and Huo, Yu
            and Wang, Ruixuan",
  year   = "2026"
}</code></pre>

<h2>Key References</h2>
<ol>
    <li>Ghandeharioun, Asma, et al. <a href="https://arxiv.org/abs/2401.06102">"Patchscopes: A unifying framework for inspecting hidden representations of language models."</a> ICML (2024).</li>
    <li>Belinkov, Yonatan. <a href="https://arxiv.org/abs/2102.12452">"Probing classifiers: Promises, shortcomings, and advances."</a> Computational Linguistics (2022).</li>
    <li>Belrose, Nora, et al. <a href="https://arxiv.org/abs/2303.08112">"Eliciting latent predictions from transformers with the tuned lens."</a> arXiv:2303.08112 (2023).</li>
    <li>Halawi, Danny, et al. <a href="https://arxiv.org/abs/2307.09476">"Overthinking the truth: Understanding how language models process false demonstrations."</a> ICLR (2024).</li>
    <li>Meng, Kevin, et al. <a href="https://arxiv.org/abs/2202.05262">"Locating and editing factual associations in GPT."</a> NeurIPS (2022).</li>
    <li>Schuster, Tal, et al. <a href="https://arxiv.org/abs/2207.07061">"Confident adaptive language modeling."</a> NeurIPS (2022).</li>
    <li>Gu, Alex, et al. <a href="https://arxiv.org/abs/2401.03065">"CRUXEval: A benchmark for code reasoning, understanding and execution."</a> ICML (2024).</li>
</ol>

</div>
</div>

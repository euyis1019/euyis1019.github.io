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
    <h1 class="post-title">从酝酿到收束：LLM 代码推理在内部经历了什么</h1>
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

<p>这篇博客最早只是论文还没写完时放在这里的一个占位。现在论文版本已经整理出来了，我也把这里改成一篇更短的导读：不重复所有实验细节，只讲我觉得最值得带走的几件事。</p>

<div class="tldr-box">
    <div class="tldr-title">TL;DR</div>
    <ul>
        <li>LLM 做代码推理时，答案通常不是在最后几层突然出现的。它会先进入一个<strong>酝酿（Brewing）</strong>阶段：大约在 14% 深度处，线性探测器已经能读出答案；但模型自己的解码路径平均要到 50% 深度左右才跟上。</li>
        <li>酝酿之后，样本会走向四种结果：Resolved、Overprocessed、Misresolved、Unresolved。它们都不是可以忽略的小类；在锚定模型上，真正 Resolved 的比例只有 <span class="highlight-num">41.5%</span>。</li>
        <li>我们用激活补丁、跳层、再注入三组干预检查这些类别。结果显示，它们更像真实的内部计算状态，而不只是事后给曲线贴标签。</li>
        <li>在 Qwen、Llama、DeepSeek 三个家族的 16 个模型上，酝酿阶段的位置和长度相当稳定（归一化时长 24–42%）；变化更大的，是模型最后能不能把答案稳稳收束出来。</li>
        <li><strong>核心 takeaway</strong>：固定层数并不总是合适。它对一部分样本太长，会把已经形成的正确答案改坏；对另一部分样本又太短，计算还没来得及完成。这也是自适应深度模型需要认真解决的停机问题：该看的不是表面置信度，而是答案是否已经“就绪”。</li>
    </ul>
</div>

<h2>一、起点：准确率看不出的差异</h2>

<p>做这项工作时，最先让我在意的是一个很普通、但很难用准确率解释的现象：同一个模型可以很轻松地追踪变量赋值，一加入算术就明显不稳；它能处理显式的 <code>for</code> 循环，却会在<strong>语义等价</strong>的展开序列上掉下来。在我们的实验里，Value Tracking（变量追踪）的 Resolved 率是 <span class="highlight-num">70.8%</span>，而只需要顺序算术的 Loop-unrolled（循环展开）只有 <span class="highlight-num">28.0%</span>。</p>

<p>如果只看最终准确率，这类差异很容易被抹平。两个模型答对率接近，并不代表它们是在同一个地方失败、或者用同一种方式成功。所以我们把问题换了一种问法：不只问某一层<em>编码</em>了什么，而是问<strong>到这一层为止，模型是不是已经把问题解出来了？</strong></p>

<p>这里有一个关键区别：答案可能已经藏在隐藏状态里，外部探测器能读出来，但它还没有变成模型自己可以稳定拿来生成的形式。我们把前者叫作<strong>信息可得（availability）</strong>，把后者叫作<strong>信息就绪（readiness）</strong>。这两个时刻不重合，后面的故事基本都从这个裂缝展开。</p>

<h2>二、双诊断框架：Probing × CSD</h2>

<p>为了把这件事拆清楚，我们做了一个合成基准，覆盖数据流、控制流以及两者的组合：Value Tracking、Computing、Conditional、Function Call、Loop、Loop-unrolled。每个模型 24,300 个样本，答案都控制成单个数字，这样跨 tokenizer 时目标仍然是单 token。诊断上，我们并排使用两把尺子：</p>

<ul>
    <li><strong>线性探测（Linear Probing）</strong>：在每层隐藏状态上训练逻辑回归分类器，看答案是否已经<em>线性可恢复</em>。它回答的是 availability。</li>
    <li><strong>上下文剥离解码（Context-Stripped Decoding, CSD）</strong>：受 Patchscopes (Ghandeharioun et al., 2024) 启发，我们把第 &ell; 层最后位置的隐藏状态注入到一个只保留问题后缀的目标 prompt 中，原始代码上下文全部拿掉，然后让模型从 &ell;+1 层继续前向传播。为了不把目标 prompt 自带的语言先验算进去，我们先跑一遍干净前向，拿到 baseline logit，再从补丁后的 logit 里减掉它。这样剩下的更接近隐藏状态本身携带的信号。CSD 回答的是 readiness。</li>
</ul>

<p>如果 CSD 在没有代码上下文的情况下仍然能解出正确答案，说明这个隐藏状态已经足够<em>自包含</em>：模型不需要再回看原始代码，就可以沿着自己的解码路径走到答案。于是，Probing 和 CSD 在不同层上的一致与分歧，就给了我们一张内部推理过程的剖面图。</p>

<h2>三、酝酿：信息先于自解码出现</h2>

<p>逐层扫过去以后，一个非常稳定的顺序出现了。我们定义两个关键层：<strong>FPCL</strong>（First Probe-Correct Layer，探测首次正确的层）和 <strong>FJC</strong>（First Joint-Correct Layer，探测与 CSD 首次同时正确的层）。在锚定设定（Qwen2.5-Coder-7B）下：</p>

<ul>
    <li>FPCL 平均出现在归一化深度 <span class="highlight-num">14%</span> 左右，说明答案很早就已经线性可读；</li>
    <li>FJC 平均要到 <span class="highlight-num">50%</span> 深度才出现，说明模型自己的解码路径晚了很多；</li>
    <li>中间这段就是<strong>酝酿期（Brewing）</strong>，平均跨越 10.7 层，占总深度的 <span class="highlight-num">38%</span>。在我们的全部实验配置里，FPCL 都早于 FJC。</li>
</ul>

<div class="figure-container">
    <img class="figure-img" src="/blog/code_understanding_llm/assets/resolution_taxonomy.png" alt="Brewing-to-resolution lifecycle">
    <div class="figure-caption">
        <strong>图 1：</strong>酝酿到收束的生命周期（Qwen2.5-Coder-7B，Computing 任务）。每个面板是一条样本轨迹：蓝色是 Probing、橙色是 CSD 的逐层数字分布。Probing 先变对，CSD 晚几层才跟上；FPCL 与 FJC 夹出的就是酝酿区间。之后，轨迹会分成 (a) Resolved、(b) Overprocessed、(c) Misresolved、(d) Unresolved 四种结果。
    </div>
</div>

<p>这说明代码推理并不是一个“最后一层才发生”的事件。很多时候，答案在浅层就已经有了雏形，只是它的表示形式还不适合自回归生成。模型还要继续花相当一段深度，把这个答案整理成自己能输出的状态。换句话说，<strong>算出答案</strong>和<strong>把答案整理到可输出</strong>，在 Transformer 里更像两件可以分开的工作。</p>

<h2>四、四种结果：不是所有样本都会顺利收束</h2>

<p>酝酿之后，轨迹并不会整齐地走向同一个终点。根据 FJC 是否存在、最终输出是否正确，以及尾部窗口（最后 1/4 层）的 CSD 置信度，每条轨迹可以归入下面四类：</p>

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
            <td>到达 FJC，且最终输出正确。答案被模型自己确认，并保持到最后</td>
            <td><strong>41.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-overprocessed">Overprocessed</span></td>
            <td>到达 FJC，但最终输出错误。正确答案曾经形成，后来被后续层改坏</td>
            <td><strong>26.4%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-misresolved">Misresolved</span></td>
            <td>从未到达 FJC，但尾部稳定偏向一个错误答案。模型不是犹豫，而是自信地错</td>
            <td><strong>8.5%</strong></td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span></td>
            <td>从未到达 FJC，尾部也没有稳定下来。在可用深度内，计算还没完成</td>
            <td><strong>23.7%</strong></td>
        </tr>
    </tbody>
</table>

<blockquote>这四类都占了相当比例，所以它们不是一些偶发的失败样本。更准确地说，它们是在描述模型内部计算的常见分岔方式。</blockquote>

<h2>五、因果干预：这些类别真的能被干预区分</h2>

<p>只画出四类轨迹还不够。它们可能只是事后看起来合理的分类。更关键的问题是：如果我们针对不同类别做干预，效果会不会按预期发生变化？为此我们做了三组实验：</p>

<ul>
    <li><strong>FJC 处的激活补丁</strong>：把源 prompt 的隐藏状态补丁到中性模板里，FJC 处的答案翻转率达到 20–44%；相比之下，酝酿前层只有 3–18%。这说明 FJC 不只是一个统计上好看的位置，它确实处在信息变得可用的关键点附近。</li>
    <li><strong>对 Overprocessed 跳层</strong>：如果直接用 FJC 的隐藏状态替换末层，只能救回约 10% 的样本。进一步看发现，主要问题是表征范数不匹配。换成 alpha 混合注入（0.7·原状态 + 0.3·FJC 状态）后，平均 <span class="highlight-success">47.8%</span> 的 Overprocessed 样本恢复正确。这个差异本身很有意思：FJC 携带了正确答案，但它的几何形态和末层状态并不完全兼容。</li>
    <li><strong>对 Unresolved 再注入</strong>：把 FPCL（信息最早可读处）的隐藏状态注入倒数 2–4 层，可以救回 22–38% 的样本；Resolved 对照组则基本保持不受损（84–100%）。这说明不少 Unresolved 并不是“模型完全不会”，而是“在给定层数里还没算完”。</li>
</ul>

<p>三组干预分别落在分类法的不同分支上，效果也和对应类别对得上。进一步的组件归因显示，改写主要发生在网络末段（28 层中的 22–27 层）：Qwen 家族里更多由<strong>注意力</strong>主导，Llama/DeepSeek 里更多由 <strong>MLP</strong> 主导。也就是说，末段改写这个现象比较稳定，但具体由哪个组件承担，会随模型家族变化。</p>

<h2>六、不同代码原语，失败方式也不同</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/fingerprint.png" alt="Task to outcome Sankey and difficulty response">
    <div class="figure-caption">
        <strong>图 2：</strong>(a) 任务到结果的 Sankey 流图（Qwen2.5-Coder-7B），条带宽度对应样本比例；(b) 各任务在难度增加时的 Resolved 率。六个任务里，有三个任务的 Resolved 低于 30%。
    </div>
</div>

<p>如果只看总体 41.5% 的 Resolved，很容易低估任务之间的差别。把每类任务单独拆开，再做受控难度扫描后，瓶颈会变得很清楚：</p>

<ul>
    <li><strong>Computing</strong>：Overprocessed 占比最高（35.6%）。模型不是完全没抓到信息，而是后面容易把已经形成的答案弄乱。算术步数从 2 增到 4 时，Overprocessed 从 25.4% 升到 <span class="highlight-num">47.5%</span>。还有一个反直觉点：纯加法的 Misresolved 率（17.5%）高于加乘混合（6.4%），说明“看起来更简单”的算子不一定更安全，模型有时会更自信地错。</li>
    <li><strong>Function Call</strong>：这是退化最明显的一类。调用深度从 1 增到 3，Resolved 从 61.1% 掉到 <span class="highlight-danger">2.5%</span>，Unresolved 升到 57.4%。也就是说，函数调用带来的间接性本身就是很强的瓶颈，FPCL 也会被明显推迟。</li>
    <li><strong>Conditional</strong>：整体看起来还可以（59.2% Resolved），但 <code>boolean_flag</code> 条件的 Misresolved 达到 18.5%，数值比较只有 6.5%。这说明布尔条件不是一个简单的附属细节，它会单独制造一类“自信选错分支”的错误。</li>
    <li><strong>Loop vs Loop-unrolled</strong>：这是最干净的一组对照，算术完全相同，只是语法形式不同。Loop 的 Resolved（35.5%）反而高于展开版（28.0%），信息变得可读的时间也更早。在双变量追踪场景下，Unresolved 从 Loop 的 23.5% 翻到展开版的 <span class="highlight-num">53.6%</span>。这说明循环语法本身给了模型一种结构提示；展开之后，同样的计算反而更难。轨迹配对分析进一步显示，末段 MLP（25–26 层）在循环版本中更强地把正确数字写进残差流；把这次 MLP 写入移植给展开版的孪生样本，可以弥合 82.6% 的 readiness 差距。</li>
</ul>

<h2>七、跨 16 个模型：过程稳定，能力不同</h2>

<div class="figure-container">
    <img class="figure-img narrow" src="/blog/code_understanding_llm/assets/brewing_stability.png" alt="Brewing stability across 16 models">
    <div class="figure-caption">
        <strong>图 3：</strong>(a) 16 个模型的归一化 FJC 分布（按家族着色）；(b) FJC 存在率；(c) 归一化酝酿时长。不同家族之间确实有差异，但 16 个模型的模型级均值都落在 24–42% 这个区间内。
    </div>
</div>

<p>把分析从单一模型扩展到 Qwen2.5-Coder、Qwen2.5、Qwen3、DeepSeek-Coder、CodeLlama、Llama-2 共 16 个模型（0.5B–14B）之后，一个比较清楚的分离出现了：</p>

<ul>
    <li><strong>过程稳定</strong>：酝酿在 0.5B 的小模型上就已经出现，归一化酝酿时长在所有 16 个模型上都落在 <span class="highlight-num">24–42%</span> 区间。Coder 与同架构 Base 模型的任务级 FJC 位置强相关（Pearson r = 0.901），平均偏移只有 0.67 层。</li>
    <li><strong>能力分化</strong>：模型变大以后，更多样本会从 Unresolved/Misresolved 转向 Resolved（0.5B 的 Resolved 为 18%，14B 为 50.3%）。代码特化训练会改善 Computing 上的结果分布。CodeLlama/Llama-2 在 Computing 上 Resolved 只有 7–8%，问题主要出在 CSD 能力而不是探测；同架构的 DeepSeek-Coder 能到 21.3%，这更像是代码预训练数据质量带来的差异，而不是架构本身的差异。</li>
</ul>

<blockquote>至少在这些模型和任务上，规模扩展更像是在提高已有过程的成功率，而不是创造一套全新的内部流程。训练改变的主要是“最后能不能解好”，而不是“酝酿到收束大概发生在什么位置”。</blockquote>

<h2>八、没有 Ground Truth 时，能不能看见这些状态</h2>

<p>前面的分类都用了真值标签，但真实部署时当然拿不到 ground truth。我们关心的是：这些内部状态会不会在模型自己的输出分布里留下痕迹？答案是会。从 CSD 的数字 softmax 中提取<strong>熵</strong>和<strong>最大置信度</strong>两个无真值信号后，尾层熵的持续上升是 Overprocessed 最强的单一探测器（AUC 0.71–0.86）。也就是说，过度处理的信号不是简单的“置信度降低”，而更像是“不确定性重新升起来”。一个只用端点统计量构成的闭式 Resolution Functional，在 Resolved-vs-Rest 二分类上可以达到 <span class="highlight-num">AUC 0.850</span>。</p>

<p>这里也能看出双诊断的必要性：去掉 Probing 或 CSD 任意一边重新拟合，四分类平衡准确率都会从 0.62 掉到约 0.40。最有用的特征之一，正是 Probing 和 CSD 的 argmax 是否一致，以及它们如何分歧；单靠其中一边是算不出来的。</p>

<h2>九、为什么重要：Early Exit 不是一个统一答案</h2>

<p>这套分类最直接的含义，是两种主导失败模式需要的处理方式正好相反：</p>

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
            <td>后续层改坏了已经形成的正确答案</td>
            <td><strong>提早退出</strong>，在答案还稳定时取出来</td>
        </tr>
        <tr>
            <td><span class="badge badge-unresolved">Unresolved</span>（23.7%，Function Call 高发）</td>
            <td>在可用深度内，计算还没有完成</td>
            <td><strong>更多深度</strong>或再注入早期信息</td>
        </tr>
    </tbody>
</table>

<p>所以，单一的 early-exit 策略很难同时解决这两类问题：太早退出会伤到 Unresolved，太晚退出又会伤到 Overprocessed。更合理的方向是<strong>结果感知（outcome-aware）的推理策略</strong>。AUC 0.850 的无真值信号说明，运行时监测模型当前处于哪种内部状态，并不是完全不可做。</p>

<h3>这对 Recursive / 自适应深度 Transformer 意味着什么</h3>

<p>这也和最近一类架构思路有关：<strong>不要把 Decoder 固定死在同一个层数预算里</strong>。比如循环复用权重、按需迭代的 Recursive / Looped Transformer（如 LoopFormer），按 token 动态分配深度的 Mixture-of-Depths，以及做动态深度缩放的 Inner Thinking Transformer。它们背后的共同假设是：<em>不同输入需要的计算深度本来就不一样</em>。我们的分类法给了这个假设一个样本级的观察：</p>

<ul>
    <li><span class="badge badge-unresolved">Unresolved</span>（23.7%）说明固定层数预算对一部分输入确实太短。再注入实验能救回其中 22–38%，说明追加计算不是空想，这些样本很多只是还没算完。</li>
    <li><span class="badge badge-overprocessed">Overprocessed</span>（26.4%）说明多余深度也不只是浪费。继续前向传播有时会把已经成形的正确答案改写掉。</li>
</ul>

<blockquote>固定深度 Transformer 让所有样本走完同一个层数预算。但在我们的数据里，这个预算对约四分之一的样本太长，对另外四分之一又太短。真正需要建模的不是“要不要早退”，而是每个样本此刻到底处在什么内部状态。</blockquote>

<p>再往前一步，自适应深度架构绕不开一个问题：<strong>到底什么时候停？</strong>我们的结果提示，停机判据不应该只看 token 级置信度，而应该看<strong>就绪性（readiness）</strong>：答案还在酝酿时继续；答案已经能被模型自己解码时退出；如果后续层开始把答案改坏，就要及时停住。recursive / depth-adaptive 架构通常默认存在这样一个可观测信号，但本身并不会自动给出它；我们这里 AUC 0.850 的无真值信号系统，可以看作一个初步原型。</p>

<p>最后也需要说清楚边界。我们的诊断还停留在层粒度，不能精确定位到具体注意力头或 MLP 子层；基准里的程序都比较短，主要是单原语控制，因此这些动态能否延续到更复杂的多语句代码，仍然需要进一步验证。还有两个更深的问题没有解决：后层<em>为什么</em>会改坏正确答案？“先可得、后就绪”到底是残差流几何带来的必然结果，还是当前训练范式下形成的经验现象？</p>

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
    <li>Raposo, David, et al. <a href="https://arxiv.org/abs/2404.02258">"Mixture-of-Depths: Dynamically allocating compute in transformer-based language models."</a> arXiv:2404.02258 (2024).</li>
    <li>Chen, Yilong, et al. "Inner Thinking Transformer: Leveraging dynamic depth scaling to foster adaptive internal thinking." ACL (2025).</li>
    <li>Jeddi, Ahmadreza, et al. "LoopFormer: Elastic-depth looped transformers for latent reasoning via shortcut modulation." ICLR (2026).</li>
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

<p>This post used to be a placeholder while the paper was still in progress. Now that the paper is written up, I have turned it into a shorter companion: not a full reproduction of the experiments, just the main things worth taking away.</p>

<div class="tldr-box">
    <div class="tldr-title">TL;DR</div>
    <ul>
        <li>Code reasoning in LLMs is usually not a last-layer event. The answer is first <strong>brewed</strong>: it becomes linearly recoverable at ~14% of depth, while the model's own decoding path only catches up around ~50%.</li>
        <li>After brewing, trajectories split into four outcomes — Resolved, Overprocessed, Misresolved, Unresolved. None of them is negligible: in the anchor setting, only <span class="highlight-num">41.5%</span> of samples are Resolved.</li>
        <li>Activation patching, layer skipping, and re-injection suggest that these outcomes correspond to intervention-sensitive internal states, not just labels assigned after the fact.</li>
        <li>Across 16 models from the Qwen, Llama, and DeepSeek families (0.5B&ndash;14B), the brewing interval remains fairly stable (normalized duration 24&ndash;42%). What varies more is whether the model can carry the answer through to a stable final output.</li>
        <li><strong>Takeaway</strong>: a fixed layer budget is too long for some samples and too short for others. The halting signal for recursive or depth-adaptive models should track readiness, not just token-level confidence.</li>
    </ul>
</div>

<h2>I. What Accuracy Hides</h2>

<p>The starting point was a simple but uncomfortable observation. A model can trace variable assignments quite well, then become brittle once arithmetic is introduced; it can handle explicit <code>for</code> loops, yet struggle with <strong>semantically equivalent</strong> unrolled sequences. In our experiments, Value Tracking reaches <span class="highlight-num">70.8%</span> Resolved, while Loop-unrolled — which only requires sequential arithmetic — reaches <span class="highlight-num">28.0%</span>.</p>

<p>Final accuracy tends to flatten these differences. Two models with similar accuracy can still fail for very different internal reasons. So we shift the question: instead of only asking what information is <em>encoded</em> at a layer, we ask whether the model has already <strong>solved</strong> the problem by that layer.</p>

<p>The answer may already be present in the hidden state before it is organized into a form the model can use for generation. We therefore distinguish <strong>information availability</strong> — when the answer is externally readable — from <strong>information readiness</strong> — when it is usable by the model's own decoding pipeline.</p>

<h2>II. The Dual Diagnostic Framework: Probing &times; CSD</h2>

<p>We build a synthetic benchmark of six task families spanning data flow, control flow, and their combination: Value Tracking, Computing, Conditional, Function Call, Loop, and Loop-unrolled. Each model is evaluated on 24,300 samples, and every answer is a single digit so the target stays one token across tokenizers. We then use two layer-wise diagnostics side by side:</p>

<ul>
    <li><strong>Linear Probing</strong>: a logistic classifier trained per layer tests whether the answer is already <em>linearly recoverable</em> from the hidden state — this measures availability.</li>
    <li><strong>Context-Stripped Decoding (CSD)</strong>: inspired by Patchscopes (Ghandeharioun et al., 2024), we extract the last-token hidden state at layer &ell; and inject it into a target prompt that keeps only the question suffix — the entire code context is stripped away — letting the model continue its forward pass from layer &ell;+1. The key adaptation is <strong>baseline logit subtraction</strong>: a clean forward pass on the bare target prompt yields the language prior, which we subtract from the patched logits to isolate the signal carried by the hidden state itself. This measures readiness.</li>
</ul>

<p>If CSD produces the correct answer after the code context is removed, the hidden state is <em>self-contained</em>: the model's own decoding pipeline can finish the job without looking back at the code. The agreement and disagreement between Probing and CSD across depth then gives us a cross-section of the internal reasoning process.</p>

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

<p>Code reasoning is therefore not a last-layer event. The answer often appears early, but in a form that is not yet friendly to autoregressive generation. The model then spends a substantial part of its depth turning that answer into something it can actually output. <strong>Computing the answer</strong> and <strong>making it output-ready</strong> behave like separable jobs.</p>

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

<blockquote>All four outcomes carry substantial mass. They are better understood as common branches of internal computation, not as rare edge cases.</blockquote>

<h2>V. The Taxonomy Is Causally Real</h2>

<p>The taxonomy is only useful if it survives intervention. If FJC and the four outcomes reflect real internal structure, targeted edits should produce predictable, outcome-specific effects. We test this with three experiments:</p>

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

<h3>Experimental Motivation for Recursive Transformers</h3>

<p>This connects directly to a direction the community is actively pursuing: <strong>architectures that refuse to lock the decoder into a fixed layer count</strong> — recursive/looped transformers that reuse weights and iterate on demand (e.g., LoopFormer), Mixture-of-Depths routing that allocates depth per token, and dynamic depth-scaling designs like the Inner Thinking Transformer. All of these share one premise: <em>different inputs need different amounts of computational depth</em>. Our taxonomy supplies sample-level experimental evidence for exactly that premise:</p>

<ul>
    <li><span class="badge badge-unresolved">Unresolved</span> (23.7%) proves the fixed layer budget is <strong>genuinely too short</strong> for some inputs — re-injection rescues 22&ndash;38% of them, so additional computation demonstrably helps: these samples simply never finished;</li>
    <li><span class="badge badge-overprocessed">Overprocessed</span> (26.4%) proves that surplus depth is not merely wasted but <strong>actively destructive</strong> — continuing the forward pass overwrites an already-formed correct answer.</li>
</ul>

<blockquote>A fixed-depth transformer sends every sample through the same layer budget. In our data, that budget is too long for roughly a quarter of samples and too short for another quarter. The key question is not simply whether to exit early, but which internal state the sample is currently in.</blockquote>

<p>Going one step further, every adaptive-depth architecture faces the same core question: <strong>when should iteration stop?</strong> Our results identify what the halting criterion should track: not token-level confidence, but <strong>readiness</strong> — exit when the answer becomes self-decodable (an FJC-like event), keep iterating while still brewing, and brake before refinement flips into corruption. This is the observability layer that recursive and depth-adaptive architectures <em>assume but do not themselves supply</em> — and our GT-free signal system at AUC 0.850 is a working prototype of that layer.</p>

<p>There are also clear boundaries. Our diagnostics operate at layer granularity, so they do not yet identify the exact attention heads or MLP sublayers that drive the brewing-to-resolution transition. The benchmark uses short single-primitive programs, so the behavior still needs to be tested on more compositional multi-statement code. Two deeper questions also remain: <em>why</em> do late layers corrupt correct answers, and is availability-before-readiness a consequence of residual-stream geometry or a contingent property of current training?</p>

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
    <li>Raposo, David, et al. <a href="https://arxiv.org/abs/2404.02258">"Mixture-of-Depths: Dynamically allocating compute in transformer-based language models."</a> arXiv:2404.02258 (2024).</li>
    <li>Chen, Yilong, et al. "Inner Thinking Transformer: Leveraging dynamic depth scaling to foster adaptive internal thinking." ACL (2025).</li>
    <li>Jeddi, Ahmadreza, et al. "LoopFormer: Elastic-depth looped transformers for latent reasoning via shortcut modulation." ICLR (2026).</li>
</ol>

</div>
</div>

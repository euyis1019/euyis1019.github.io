---
layout: default
title: "Your LLM Understood the Code — Then Forgot the Answer"
date: 2026-03-09
permalink: /blog/code_understanding_llm/
categories: [Research, Interpretability, Code]
excerpt: "We use Patchscopes to dissect how Code LLMs process code layer-by-layer, revealing a counterintuitive phenomenon: models often compute the right answer mid-forward-pass, then actively overwrite it."
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
    --post-glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.05);
    --post-table-th-bg: #fafafa;
    --post-em-bg: rgba(0, 113, 227, 0.05);
    --post-caption-bg: #fafafa;
}

[data-theme="dark"] {
    --post-bg: #1e293b;
    --post-text-primary: #f1f5f9;
    --post-text-secondary: #94a3b8;
    --post-accent-color: #3b82f6;
    --post-accent-gradient: linear-gradient(135deg, #3b82f6, #60a5fa);
    --post-border-color: rgba(255, 255, 255, 0.1);
    --post-code-bg: #0f172a;
    --post-code-color: #fb7185;
    --post-card-bg: rgba(30, 41, 59, 0.7);
    --post-glass-border: rgba(255, 255, 255, 0.1);
    --post-glass-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
    --post-table-th-bg: #0f172a;
    --post-em-bg: rgba(59, 130, 246, 0.15);
    --post-caption-bg: #0f172a;
}

body {
    background-color: var(--bg-primary, #fafafa);
    transition: background-color 0.3s ease;
}

.post-container {
    max-width: 820px;
    margin: 3rem auto;
    font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    line-height: 1.8;
    color: var(--post-text-primary);
    padding: 24px;
    background: var(--post-bg);
    border-radius: 20px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.03);
    padding: 4rem;
    transition: all 0.3s ease;
}

[data-theme="dark"] .post-container {
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
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
    font-size: 2.8em;
    font-weight: 800;
    letter-spacing: -0.02em;
    color: var(--post-text-primary);
    margin-bottom: 1rem;
    line-height: 1.2;
    background: var(--post-text-primary);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}

[data-theme="light"] .post-title {
    background: linear-gradient(90deg, #1d1d1f, #434345);
    -webkit-background-clip: text;
}

.post-meta {
    font-size: 1.05em;
    color: var(--post-text-secondary);
    font-weight: 500;
    margin-top: 1.5rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
}

.post-meta strong {
    color: var(--post-text-primary);
    font-weight: 700;
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

/* Glassmorphic Figures */
.figure-container {
    margin: 3.5rem 0;
    text-align: center;
}
.figure-box {
    width: 100%;
    background: var(--post-card-bg);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border: 1px solid var(--post-glass-border);
    box-shadow: var(--post-glass-shadow);
    border-radius: 16px;
    padding: 4rem 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: var(--post-text-secondary);
    font-family: "Inter", sans-serif;
    transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), box-shadow 0.3s ease;
}
.figure-box:hover {
    transform: translateY(-5px);
    box-shadow: 0 15px 40px rgba(31, 38, 135, 0.1);
}
[data-theme="dark"] .figure-box:hover {
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
}
.figure-icon {
    font-size: 3em;
    margin-bottom: 1.5rem;
    filter: drop-shadow(0 4px 8px rgba(0,0,0,0.1));
}
.figure-caption {
    margin-top: 1.5rem;
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
    <h1 class="post-title">你的模型看懂了代码，但却在最后忘了答案</h1>
    <div class="post-meta">
        By <strong>Yifu Guo</strong> and <strong>Siyue Chen</strong> &middot; March 2026
    </div>
</div>

<p>今天，Code Agent 已经能够作为虚拟的软件工程师，在真实世界的复杂代码库中自主解决 issue、编写测试、重构系统。但剥开这层宏大的表现，它们在底层反复调用的核心基本功，其实就是那么原子的几种代码推演能力。可是，当大模型在处理这些代码时，内部究竟发生了什么？</p>

<p>我们通常只能看到最终输出的“成功”或“失败”，但这掩盖了一个惊人的可能性：<strong>很多时候模型其实会做，它已经在最初的几层网络里算出了正确答案，但却在随后的计算中自己把它覆盖掉了。</strong></p>

<p>在这篇博客中，我们用一组单纯的代码刺激集（Controlled Stimuli），结合可解释性工具 <strong><a href="https://arxiv.org/abs/2401.06102">Patchscopes</a></strong> (Ghandeharioun et al., 2024)，去解剖 Qwen2.5-Coder 内部的处理时序。我们对该方法进行了深度适配与改造，用它照亮了代码推断场景下一个让人意外的现象：</p>

<ul>
    <li>对于某些代码任务，正确的语义会非常早地出现，但极度不稳定，最终极容易被模型自己损毁（我们称之为 <em>Overthinking</em>）。</li>
    <li>“模型内部知道”和“模型能说出来”是两回事。</li>
    <li>这直接暗示了现有 <em>Early Exit</em> 方法在一个真实的 Agent 场景中蕴含着极大的潜能，前提是我们得知道模型正在执行什么类型的任务。</li>
</ul>

<p>代码理解任务其实是观察 LLM 推理不稳定性（Reasoning Instability）的一个绝佳窗口：因为相比起模糊的自然语言，代码运行具有完全确定性的中间状态语义，这使得我们可以获得非常干净的 <em>ground truth</em> 信号。</p>

<h2>一、一个反直觉的对比：数据流 vs 控制流</h2>

<p>让这个研究起步的是我们在真实 Code Agent 场景中观察到的一个差异。当 Agent 在尝试修复一个 issue（比如 SWE-Bench）时，它底层反复在做两件事：</p>
<ol>
    <li>追踪变量的数据流：<code>x = 2; y = x + 1</code></li>
    <li>判断 if/else 的控制流：<code>if x > 1: return A else: return B</code></li>
</ol>

<p>这似乎都是基本的逻辑推理。但当我们把同样规模的 <strong>Qwen2.5-Coder 7B</strong> 放在这两类通过脚本生成的、复杂度极简的纯净任务上时，得到了非常割裂的结果：</p>

<ul>
    <li><strong>Conditional（控制流判断）</strong>：最终准确率 81.7%</li>
    <li><strong>Computing（数据流计算）</strong>：最终准确率 18.6%</li>
</ul>

<p>超过 60 个百分点的差距。而且验证实验证明，即使我们把 Computing 退化成不带加减法、纯粹的连等赋值（<code>a=b; c=a</code>），错误率依然极高。这意味着问题不在于“算术太难”，而是<strong>数据流追溯这种代码格式，在模型内部激活了一条极度脆弱的计算链路。</strong></p>

<h2>二、把手伸进模型：寻找丢失的答案</h2>

<p>既然结果错了，那它是从一开始就不懂，还是在半路走丢的？</p>

<p>为了搞清楚这件事，我们利用 <a href="https://arxiv.org/abs/2401.06102">Patchscopes</a> 作为探索工具。你可以把它理解为一种高级的“读脑器”：我们提取模型在处理代码问题时第 <span style="font-family:serif;">&ell;</span> 层的 hidden state <span style="font-family:serif;">h<sub>&ell;</sub></span>，把它“嫁接”到一个标准化的提问模板（比如 <code>"# The answer is "</code>）里，然后逼迫模型直接生成结果。</p>

<p>这里有两个值得一提的工程细节（caveats），对试图复现的人或许有用（详见 <a href="#appendix-a">附录 A</a>）：</p>
<ol>
    <li><strong>你只能在 Last-position 注入</strong>：早期的可解释性工作（比如 <a href="https://arxiv.org/abs/2304.14997">Belrose et al., 2023 的 Tuned Lens</a> 或部分 activation patching）提倡在特定 token 处读写。但在代码的自回归生成里，如果你在中间位置注入 <span style="font-family:serif;">h<sub>&ell;</sub></span>，后续符号的 KV cache 会完全错位，准确率直接掉到 0。</li>
    <li><strong>大模型需要 First-Token 截断</strong>：当我们在 14B 模型上做扫描时，发现模型容易顺着 prompt 的惯性一直生成幻觉代码，如果不只统计生成的第一个 token，会得到一团乱码。</li>
</ol>

<p>当我们用这种方式把 7B 模型里每一层的“瞬时想法”打印出来时，我们看到了这篇文章最核心的 findings。</p>

<h2>三、最让我惊讶的发现：Overthinking</h2>

<p>直接说结论：<strong>对于 18.6% 准确率的 Computing 任务，有高达 55.8% 的测试用例，模型曾在中间某一层给出了完全正确的答案。</strong></p>

<p>这里流失了近 37 个百分点的正确率。这说明很多时候，模型不是“不会做”，它只是<strong>过度计算（Overthinking）</strong>了。它在中间网络找到了答案，但在随后的层网络里——也许是被不相关的 Attention 捕捉到的噪声干扰，也许是表征坍塌——这个正确答案被覆盖掉了。</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">🔥</div>
        <div>[Placeholder: Overthinking Gap Heatmap]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 1：</strong>纵坐标是 5 种不同规模的模型，横坐标是不同的代码任务类别。颜色表示 Overthinking Gap（曾经算对的概率 减去 最终正确的概率）。你可以清晰地看到 Computing 任务在 7B 模型上呈现出巨大的红色深坑。
    </div>
</div>

<p>这种现象在不同的任务上极度不均匀分布。对于前文提到的 Conditional（控制流）任务，它的内部表征就极其稳定：只要在中间某层“想清楚”了，它就会一路平滑地保持正确直到最后一层输出。</p>

<p>换句话说，在代码场景里，<strong>“会不会做”和“最终会不会说对”，完全是两回事。</strong></p>

<p><strong>你的大模型不是不够聪明没有算清代码。它曾经掌握了答案，只不过一不小心，在到达最终层前忘了告诉你。</strong></p>

<h2>四、Information Brewing：它知道，只是没准备好告诉你</h2>

<p>那么，为什么这种遗忘只发生在算术计算（Computing）上？</p>

<p>我们引入了早期线性探测器（Linear Probing，参考 <a href="https://arxiv.org/abs/1610.01644">Alain & Bengio, 2016</a>）和 Patchscopes 进行了交叉验证，发现了一个很有意思的现象：<strong>Information Brewing（信息酝酿期）</strong>。</p>

<ul>
    <li>在 14B 模型里，如果我们用一个简单的线性分类器去看，模型的 <strong>第 2 层</strong> 就已经完全知晓了 Computing 的最终答案（Probing 准确率接近 100%）。正确信息在极浅层就已经形成了良好的线性可分表示。</li>
    <li>但如果我们用 Patchscopes 去逼迫模型“说出”答案，它一直要憋到 <strong>第 33 层</strong> 才能正确解码。</li>
</ul>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📈</div>
        <div>[Placeholder: Probing vs Patchscopes Dual Curves]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 2：</strong>14B 模型上 Probing (提前到达 100%) 和 Patchscopes (延期上升) 的准确率重叠对比。中间 31 层的长长平原，就是所谓的 Brewing 间隙。
    </div>
</div>

<p>这解释了脆弱性的来源。模型在底层提取出了答案特征，但它并不是一个“可以用来生成 Token”的格式（Output-Ready）。剩下的近 30 层前向传播，模型其实是在艰难地执行某种格式转换乃至对齐。就在这漫长且艰难的转换期内，任何外在的干扰极其容易覆盖本体，导致它走到最后一步时已经面目全非。</p>

<p>这也侧面呼应了近期 Mechanistic Interpretability 里关于内部计算冗余和层间功能漂移（比如 <a href="https://arxiv.org/abs/2309.00667">DoLa: Decoding by Contrasting Layers</a>）的一些观察。</p>

<h2>五、一个小插曲：虚假的循环理解</h2>

<p>这里还要顺手提一个很有意思的副产物。很多 Benchmarks 测试里会包含 <code>for i in range(n)</code> 的闭包或循环，当我们测试单次迭代时（<code>n=1</code>），所有模型都表现出了近 100% 的不可思议的能力。</p>

<p>但当我们把测例挪到 <code>n=2</code> 或 <code>n=3</code> 时，准确率雪崩。我们用语义将其打散成平铺的展开代码再去测时，7B 模型瞬间提升了 70 个百分点。</p>

<p>这说明：目前的 LLM 处理单次 <code>Loop</code>，根本没有动用“循环推理”。它只是利用强大的 Pattern Matching，在这特定场景把它等价降解为了单纯的数据复制（Copying）电路。这不仅是对某些榜单“理解力”的戳破，也警告了我们在分析内部机制时：<strong>有些时候模型看起来在做高难度体操，其实它只是在地下通道里抄了条近路。</strong></p>

<h2>六、我们该如何利用此现象？(The Adaptive Agent)</h2>

<p>上述这些偏向诊断性质的发现，对于我们实际上帝视角构建 Code Agent 或优化推理（Inference）有什么用处呢？</p>

<p>由于模型天然具有 Overthinking 的体质，一个最朴素的念头就是 <strong>Early Exit（提早退出推理）</strong>。其实 <a href="https://arxiv.org/abs/2207.07061">Schuster et al., 2022 (CALM)</a> 等早在 NLP 领域便引入并证实过 Early Exit 能省钱。</p>

<p>但我们的数据提供了更为精细化的 Insight。如果我们依赖固定验证集，为不同任务设计一个 Best Fixed Layer（寻找最安全的静态退出层），在 7B 模型上的表现是：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>任务</th>
            <th>默认（最后一层）准确率</th>
            <th>理想退出的准确率</th>
            <th>提升</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Computing (数据计算)</td>
            <td>18.6%</td>
            <td>45.0% (@L22)</td>
            <td><span class="highlight-success">+ 26.4 pp</span></td>
        </tr>
        <tr>
            <td>Conditional (逻辑条件)</td>
            <td>81.7%</td>
            <td>82.7% (@L26)</td>
            <td>+ 1.0 pp</td>
        </tr>
    </tbody>
</table>

<p>这才是本文真正想传递的核心理念之一：<strong>收益是极度 Task-Aware（任务依赖）的。</strong>如果你对所有的代码逻辑一刀切地去设计 Early Exit，你会搞砸一半的稳定性。真正的可靠 Agent：在检测到在做“条件判断”时让大模型推到最后，在检测到“复杂中间数值追源”时，截取模型在黄金期里的内部决议。</p>

<p>比起追求无休止的数据拟合与模型尺寸堆叠，探究大模型内部的思考折返跑（Overthinking 和 Brewing），也许能让我们用低得多的成本，拿回原本就属于它的智慧。</p>

<h2>引用 (Citation)</h2>
<p>Cited as:</p>
<blockquote>
  <p>Guo, Yifu; Chen, Siyue. (Mar 2026). Your LLM Understood the Code — Then Forgot the Answer. Yifu Guo's Blog. https://ericguo1019.com/blog/code_understanding_llm/.</p>
</blockquote>

<p>Or</p>

<pre><code>@article{guo2026codeunderstanding,
  title   = "Your LLM Understood the Code — Then Forgot the Answer",
  author  = "Guo, Yifu and Chen, Siyue",
  journal = "ericguo1019.com",
  year    = "2026",
  month   = "Mar",
  url     = "https://ericguo1019.com/blog/code_understanding_llm/"
}</code></pre>

<h2>参考文献 (References)</h2>
<ol>
    <li>Ghandeharioun, Asma, et al. <a href="https://arxiv.org/abs/2401.06102">"Patchscopes: A unifying framework for inspecting hidden representations of language models."</a> ICML (2024).</li>
    <li>Alain, Guillaume, and Yoshua Bengio. <a href="https://arxiv.org/abs/1610.01644">"Understanding intermediate layers using linear classifier probes."</a> arXiv preprint arXiv:1610.01644 (2016).</li>
    <li>Belrose, Nora, et al. <a href="https://arxiv.org/abs/2304.14997">"Eliciting latent predictions from transformers with the tuned lens."</a> arXiv preprint arXiv:2304.14997 (2023).</li>
    <li>Chuang, Yung-Sung, et al. <a href="https://arxiv.org/abs/2309.00667">"DoLa: Decoding by Contrasting Layers Improves Factuality in Large Language Models."</a> ICLR (2024).</li>
    <li>Schuster, Tal, et al. <a href="https://arxiv.org/abs/2207.07061">"Confident Adaptive Language Modeling."</a> NeurIPS (2022).</li>
</ol>

<h2 id="appendix-a">附录 A：Patchscopes 在代码任务中的方法论适配</h2>
<p>Patchscopes 最初被设计用于自然语言中的事实提取（如实体属性）。将其迁移至多步代码执行追踪时，我们遇到了极其严重的不适配，并做出了以下核心改造：</p>
<ol>
    <li><strong>必须使用 Last-position 注入</strong>：早期的可解释性工作（如 <a href="https://arxiv.org/abs/2304.14997">Belrose et al., 2023 的 Tuned Lens</a>）提倡在特定 prompt token 处读写。但在代码的自回归生成里，若你在 <code>x_pos</code> 注入 <span style="font-family:serif;">h<sub>&ell;</sub></span>，由于后续符号（如 <code>&rarr;</code> 或 <code>=</code>）的 KV cache 仍基于原始上下文计算，产生的信息污染会让准确率直接暴跌至接近 0。只有在 Last-position 注入，才能无干扰地直接进入生成阶段。</li>
    <li><strong>大模型需要 First-Token 截断校正</strong>：我们发现 14B 参数的模型出现了 <code>Copying</code> 稳定性甚至弱于 <code>Computing</code> 的反常倒挂。排查后发现，大模型的“生成惯性”极强，超过 79.7% 的失效案例是因为模型顺着注入特征强行进行了长篇幻觉续写。强制截取生成的首个 Token 并进行校正后，预期的数据规律才得以恢复。</li>
    <li><strong>任务依赖的解码边界</strong>：Patchscopes 的解码能力存在极强的局限性。在高运算密度的 Computing 任务中，即便 Probing 证实答案已经在线性空间内存在（100%可分），Patchscopes 仍会有超过数十层的滞后才能成功解码。这证实了：<strong>信息被计算出来</strong>，和<strong>信息转化为可随意输出的格式</strong>，在语言模型中是两条完全独立的链路。</li>
</ol>

</div>


<!-- ================= 英文版 ================= -->
<div class="lang-en lang-content">

<div class="post-header">
    <h1 class="post-title">Your LLM Understood the Code — Then Forgot the Answer</h1>
    <div class="post-meta">
        By <strong>Yifu Guo</strong> and <strong>Siyue Chen</strong> &middot; March 2026
    </div>
</div>

<p>Today, Code Agents are increasingly capable of acting as autonomous software engineers—resolving complex issues, writing tests, and refactoring systems within real-world codebases. Yet, peel back this grandiose facade, and the bedrock of their performance relies strictly on a few atomic code deduction capabilities. But what exactly happens <em>inside</em> these models when they process such code?</p>

<p>Typically, we only see the final output—right or wrong—which masks a fascinating possibility: <strong>often, the model actually knows the answer. It computes the correct result in its early layers but actively overwrites it before reaching the final token layer.</strong></p>

<p>In this post, we apply controlled code understanding stimuli alongside the interpretability probing tool <strong><a href="https://arxiv.org/abs/2401.06102">Patchscopes</a></strong> (Ghandeharioun et al., 2024) to dissect the layer-by-layer processing timelines within Qwen2.5-Coder. We heavily adapted and modified this probe to illuminate a surprising phenomenon in a very practical setting:</p>

<ul>
    <li>For certain coding tasks, semantic truth emerges extremely early but is behaviorally unstable, getting destroyed by the model's ongoing processing (a phenomenon we term <em>Overthinking</em>).</li>
    <li>"What the model internally knows" vs. "What it can articulate" are vastly different mechanisms.</li>
    <li>This directly suggests that existing <em>Early Exit</em> strategies hold massive potential for real-world Agent scenarios—if and only if we are acutely aware of the specific task profile the model is currently executing.</li>
</ul>

<p>Code understanding turns out to be an exceptional lens for studying Reasoning Instability. Unlike the inherent fuzziness of natural language, executed code provides deterministic, verifiable semantic intermediate states, granting us incredibly clean <em>ground truth</em> signals.</p>

<h2>I. A Counterintuitive Gap: Data Flow vs Control Flow</h2>

<p>Our initial motivation came from observing behavior inside real-world Code Agents solving SWE-Bench-style issues. Under the hood, the agent relentlessly oscillates between two primitives:</p>
<ol>
    <li>Tracking data-flow: <code>x = 2; y = x + 1</code></li>
    <li>Reasoning control-flow: <code>if x > 1: return A else: return B</code></li>
</ol>

<p>When subjecting the <strong>Qwen2.5-Coder 7B</strong> parameter model to script-generated, minimalist, pure examples of these two operations, the results were severely split:</p>

<ul>
    <li><strong>Conditional (Control-flow bounds):</strong> 81.7% final accuracy.</li>
    <li><strong>Computing (Data-flow arithmetic):</strong> 18.6% final accuracy.</li>
</ul>

<p>A staggering gap. Follow-up validation—flattening out addition and subtraction into pure sequential assignments (e.g. <code>a=b; c=a</code>)—yielded similarly abysmal metrics. The root issue isn't that "math is hard." Rather, <strong>the specific data-flow tracking formatting activates an incredibly fragile internal computational pathway.</strong></p>

<h2>II. Poking Around with Patchscopes</h2>

<p>If the final answer is wrong, did it never understand it, or did it lose it along the way?</p>

<p>We leveraged <a href="https://arxiv.org/abs/2401.06102">Patchscopes</a> to find out. Conceptually, it acts as a non-invasive mind-reader: we copy the hidden state <span style="font-family:serif;">h<sub>&ell;</sub></span> of the code evaluation from layer <span style="font-family:serif;">&ell;</span> and "patch" it into a generic prompt canvas (e.g. <code>"# The answer is "</code>), forcing the model to explicitly vocalize its mid-forward-pass assumptions.</p>

<p>For those interested in exploring this space, we must highlight two massive <em>caveats</em> shaping the adaptation (detailed in <a href="#appendix-a-en">Appendix A</a>):</p>
<ol>
    <li><strong>Last-Position Execution is Non-Negotiable.</strong> Early literature (such as <a href="https://arxiv.org/abs/2304.14997">Belrose et al., 2023's Tuned Lens</a>) advocated for targeted extraction/injection at arbitrary tokens. Doing so in coding loops shatters following token KV-caches, driving accuracy flat to zero due to massive context contamination.</li>
    <li><strong>First-Token Pruning on Giants.</strong> At 14B bounds, generation momentum takes over, causing wild hallucinations spilling out endless text. If you don't aggressively crop evaluation to strictly the absolute first token emitted, you retrieve nothing but noise.</li>
</ol>

<h2>III. The Strongest Finding: Overthinking</h2>

<p>The punchline: <strong>For the Computing task suffering an 18.6% final accuracy rate, a stunning 55.8% of test subjects successfully calculated the perfect answer at some intermediate layer block.</strong></p>

<p>Nearly 37 percentage points of intelligence evaporated in transit. The model didn't fail to understand; it <strong>overthought</strong>. An answer materialized deep in the network but was actively compromised throughout later evaluations—perhaps corrupted by irrelevant Attention sweeps—wholly overwriting the truth.</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">🔥</div>
        <div>[Placeholder: Overthinking Gap Heatmap]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 1:</strong> Depicts the 5 models scales against 5 task variables. The color spectrum defines the "Overthinking Gap" (Ever Correct % subtracted by Final Correct %). The massive pit over 7B Computing tasks flashes dark red against the stability of Conditional counterparts.
    </div>
</div>

<p>Remarkably, this instability isn't universal. Under Conditional logic trials, representations harden securely. Once the answer is grasped, it sails seamlessly and stably towards final output.</p>

<p>In short: <strong>"Knowing the answer" and "being able to dictate the answer" utilize fractured pathways.</strong></p>

<h2>IV. Information Brewing: The Output-Ready Void</h2>

<p>Why does such catastrophic decay orbit Computing exclusively?</p>

<p>We correlated findings applying early classical Linear Probing frameworks (inspired by <a href="https://arxiv.org/abs/1610.01644">Alain & Bengio, 2016</a>) against our Patchscopes generation bounds, uncovering what we designate as the <strong>Information Brewing Gap</strong>.</p>

<ul>
    <li>Running linear approximations on our 14B models indicated that by superficial <strong>Layer 2</strong>, the core answers were already strictly determinable (Probing ~100%). Truth was perfectly intact and linearly separable.</li>
    <li>Yet forcibly prompting the LLM via Patchscopes stalled completely, taking up until <strong>Layer 33</strong> to start correctly articulating generated completions.</li>
</ul>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📈</div>
        <div>[Placeholder: Probing vs Patchscopes Dual Curves]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 2:</strong> Probing peaks mapped rapidly against delayed Patchscopes decoding on 14B parameters. The 31 flatland layers bridging them outlines the harsh formatting translation reality.
    </div>
</div>

<p>This massive translation lag defines vulnerability. The representation matures conceptually, but exists in a form heavily hostile toward auto-regressive generation execution (Non Output-Ready). For 30 subsequent layers, the model violently forces format adjustments to match token distributions—a vulnerable window where the slightest external semantic noise wipes the memory clean. This aligns closely with broader mechanistic findings detailing deep layer functional drifts (e.g., <a href="https://arxiv.org/abs/2309.00667">DoLa</a>).</p>

<h2>V. A Brief Aside: The Loop Illusion</h2>

<p>When measuring strict <code>Loop</code> control behaviors containing traditional <code>for i in range(n)</code> operations, all scales essentially scored perfectly at <code>n=1</code> loops.</p>

<p>Incrementing <code>n=2</code> forced total systemic breakdowns. Upon manually unspooling loops to flat linear strings for verification, scaling models snapped back to near-perfect performance. The insight is glaring: Current LLMs tackling basic iteration bypass genuine recursive loop inferences entirely. Via strong pattern recognition, they adapt single loops directly to primitive semantic <code>Copying</code> circuitry shortcuts. It warns us to remain vigilant against illusionary competence masked within Benchmarks.</p>

<h2>VI. Utilizing Instability: Task-Aware Early Exits</h2>

<p>How does diagnosing underlying instability optimize real-world autonomous coding systems?</p>

<p>Since overwriting manifests broadly, stopping inference processing artificially via an <strong>Early Exit</strong> is the natural conclusion—strategies implemented historically in broader NLP parameters per <a href="https://arxiv.org/abs/2207.07061">Schuster et al. (2022)</a> to cut inference costs.</p>

<p>Yet our observations stipulate refined rules. Determining the safest, statically optimal extraction layer bounded by Validation tests mapped out as such:</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Operation</th>
            <th>Native Default Exit</th>
            <th>Optimal Layer Exit</th>
            <th>Gained Return</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Computing</td>
            <td>18.6%</td>
            <td>45.0% (@L22)</td>
            <td><span class="highlight-success">+ 26.4 pp</span></td>
        </tr>
        <tr>
            <td>Conditional</td>
            <td>81.7%</td>
            <td>82.7% (@L26)</td>
            <td>+ 1.0 pp</td>
        </tr>
    </tbody>
</table>

<p>This is the keystone claim: <strong>Benefit yields are aggressively Task-Aware.</strong> Adopting a singular overarching blanket truncation metric ruins overall stability. Effective future Agents must dynamically read operational contexts—allowing deep deductions to sail toward completion smoothly while violently interrupting numerical data-tracing evaluations where their answers peak brightest within internal processing architectures before they succumb to self-imposed forgetfulness.</p>

<h2>Citation</h2>
<p>Cited as:</p>
<blockquote>
  <p>Guo, Yifu; Chen, Siyue. (Mar 2026). Your LLM Understood the Code — Then Forgot the Answer. Yifu Guo's Blog. https://ericguo1019.com/blog/code_understanding_llm/.</p>
</blockquote>

<p>Or</p>

<pre><code>@article{guo2026codeunderstanding,
  title   = "Your LLM Understood the Code — Then Forgot the Answer",
  author  = "Guo, Yifu and Chen, Siyue",
  journal = "ericguo1019.com",
  year    = "2026",
  month   = "Mar",
  url     = "https://ericguo1019.com/blog/code_understanding_llm/"
}</code></pre>

<h2>References</h2>
<ol>
    <li>Ghandeharioun, Asma, et al. <a href="https://arxiv.org/abs/2401.06102">"Patchscopes: A unifying framework for inspecting hidden representations of language models."</a> ICML (2024).</li>
    <li>Alain, Guillaume, and Yoshua Bengio. <a href="https://arxiv.org/abs/1610.01644">"Understanding intermediate layers using linear classifier probes."</a> arXiv preprint arXiv:1610.01644 (2016).</li>
    <li>Belrose, Nora, et al. <a href="https://arxiv.org/abs/2304.14997">"Eliciting latent predictions from transformers with the tuned lens."</a> arXiv preprint arXiv:2304.14997 (2023).</li>
    <li>Chuang, Yung-Sung, et al. <a href="https://arxiv.org/abs/2309.00667">"DoLa: Decoding by Contrasting Layers Improves Factuality in Large Language Models."</a> ICLR (2024).</li>
    <li>Schuster, Tal, et al. <a href="https://arxiv.org/abs/2207.07061">"Confident Adaptive Language Modeling."</a> NeurIPS (2022).</li>
</ol>

<h2 id="appendix-a-en">Appendix A: Methodological Adaptations to Patchscopes</h2>
<p>Patchscopes was originally designed for factual extraction (e.g., token identity and entity attributes) in natural language. Migrating it to multi-step code execution tracking revealed severe incompatibilities, leading to three crucial modifications:</p>
<ol>
    <li><strong>Last-Position Injection is Mandatory</strong>: While earlier interpretability methods (like <a href="https://arxiv.org/abs/2304.14997">Belrose et al., 2023's Tuned Lens</a>) advocated for injection at arbitrary mid-prompt tokens, doing so in auto-regressive code generation completely corrupts subsequent KV caches. If <span style="font-family:serif;">h<sub>&ell;</sub></span> is injected at <code>x_pos</code>, subsequent tokens (like <code>&rarr;</code> or <code>=</code>) fall out of sync, driving accuracy roughly to 0. Injecting strictly at the last position bypasses context contamination.</li>
    <li><strong>First-Token Pruning on Giants</strong>: Surprisingly, the 14B model initially exhibited higher instability on simple <code>Copying</code> tasks than on computing tasks. Investigation revealed that the massive generation momentum of larger models caused them to hallucinate immense continuous code blocks instead of answering the target. Forcing strict first-token cropping and evaluation was required to restore the legitimate data patterns.</li>
    <li><strong>Task-Dependent Decoding Bounds</strong>: Patchscopes' decoding capability is inherently limited. In dense computing tasks, even when linear Probing validated that the outcome already existed in the hidden dimension (100% separable), Patchscopes delayed successfully articulating that answer by dozens of layers. This confirms that <strong>computing the information</strong> and <strong>aligning the information for token emission</strong> are wholly disjointed circuits within LLMs.</li>
</ol>

</div>
</div>


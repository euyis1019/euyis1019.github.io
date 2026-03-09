---
layout: default
title: "Your LLM Understood the Code — Then Forgot the Answer"
date: 2026-03-09
permalink: /blog/code_understanding_llm/
categories: [Research, Interpretability, Code]
excerpt: "Patchscopes 揭示代码大模型内部动态差异 | Patchscopes reveals dynamic differences in internal processing of Code LLMs."
---

<style>
/* Academic / Lil'Log inspired styling for the blog post */
.post-container {
    max-width: 800px;
    margin: 0 auto;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    line-height: 1.7;
    color: #333;
    padding: 0 15px;
}

.post-header {
    text-align: center;
    margin-bottom: 2.5rem;
    padding-bottom: 1.5rem;
    border-bottom: 1px solid #eaecef;
}

.post-title {
    font-size: 2.2em;
    font-weight: 700;
    color: #111;
    margin-bottom: 0.5rem;
    line-height: 1.3;
}

.post-meta {
    font-size: 1.0em;
    color: #666;
    font-style: italic;
    margin-top: 1rem;
}

.post-meta a {
    color: #0366d6;
    text-decoration: none;
}

/* Headings */
.post-container h2 {
    font-size: 1.6em;
    font-weight: 600;
    margin-top: 2.5rem;
    margin-bottom: 1rem;
    color: #24292e;
    border-bottom: 1px solid #eaecef;
    padding-bottom: 0.3em;
}

.post-container h3 {
    font-size: 1.3em;
    font-weight: 600;
    margin-top: 2rem;
    margin-bottom: 1rem;
    color: #24292e;
}

.post-container p {
    margin-bottom: 1.2rem;
}

/* Emphasized terms */
.post-container em {
    color: #000;
    font-weight: 500;
}

/* Inline Code */
.post-container code {
    background-color: rgba(27,31,35,0.05);
    border-radius: 3px;
    font-size: 85%;
    margin: 0;
    padding: 0.2em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
    color: #d73a49;
}

/* Tables */
.academic-table {
    width: 100%;
    margin: 2rem 0;
    border-collapse: collapse;
    font-size: 0.95em;
}
.academic-table th, .academic-table td {
    padding: 10px 12px;
    text-align: left;
    border-bottom: 1px solid #e1e4e8;
}
.academic-table th {
    font-weight: 600;
    background-color: #f6f8fa;
    color: #24292e;
}

/* Figure Placeholder */
.figure-container {
    margin: 2.5rem 0;
    text-align: center;
}
.figure-box {
    width: 100%;
    border: 1px dashed #bbb;
    background-color: #f6f8fa;
    border-radius: 6px;
    padding: 3rem 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #586069;
    font-family: monospace;
}
.figure-icon {
    font-size: 2.5em;
    margin-bottom: 1rem;
    opacity: 0.6;
}
.figure-caption {
    margin-top: 1rem;
    font-size: 0.9em;
    color: #586069;
    line-height: 1.5;
    text-align: left;
    border-left: 3px solid #dfe2e5;
    padding-left: 1rem;
}

/* Blockquotes */
.post-container blockquote {
    padding: 0 1em;
    color: #6a737d;
    border-left: 0.25em solid #dfe2e5;
    margin: 1.5rem 0;
}

/* Bilingual Toggle */
.lang-toggle {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 15px;
}
.lang-btn {
    background: transparent;
    border: 1px solid #d1d5da;
    color: #24292e;
    padding: 4px 12px;
    margin-left: 8px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.85em;
    transition: all 0.2s;
}
.lang-btn.active {
    background: #0366d6;
    color: white;
    border-color: #0366d6;
}

/* Lists */
.post-container ul, .post-container ol {
    margin-bottom: 1.2rem;
    padding-left: 2em;
}
.post-container li {
    margin-bottom: 0.25em;
}

/* Highlighted numbers */
.highlight-num {
    color: #d73a49;
    font-weight: 600;
}
.highlight-success {
    color: #28a745;
    font-weight: 600;
}
</style>

<div class="post-container">

<div class="lang-toggle">
    <button class="lang-btn active" onclick="switchLang('zh')">中文</button>
    <button class="lang-btn" onclick="switchLang('en')">English</button>
</div>

<!-- ================= 中文版 ================= -->
<div class="lang-zh lang-content">

<div class="post-header">
    <h1 class="post-title">Your LLM Understood the Code — Then Forgot the Answer</h1>
    <div class="post-meta">
        By <strong>Yifu Guo</strong> and <strong>Siyue Chen</strong> &middot; March 2026
    </div>
</div>

<p>代码大模型在 SWE-bench 风格的评测上每隔几个月就会刷新记录，但我们对这些模型 <em>内部是如何处理代码</em> 的了解，依然非常有限。最终准确率告诉我们模型会不会，但并不告诉我们模型 <em>怎么</em> 会——或者更重要的，<em>为什么不会</em>。</p>

<p>本文记录了一系列基于 Patchscopes 的逐层可解释性实验。我们发现：对于代码 LLM 来说，不同类型的代码任务不只是“难度不同”，而是激活了截然不同的内部处理模式——拥有完全不同的信息成熟时序、稳定性签名和编码格式。</p>

<p>最惊人的发现之一：在某些任务上，模型曾经在中间层“算对了”，却在后续层将正确答案主动覆盖，最终输出错误。</p>

<h2>一、一个让人不安的对比</h2>

<p>把 <strong>Qwen2.5-Coder 7B</strong> 放在两类等难度任务上：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>任务</th>
            <th>最终准确率</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Conditional（控制流：if/else 条件判断）</td>
            <td><span class="highlight-success">81.7%</span></td>
        </tr>
        <tr>
            <td>Computing（数据流：变量追踪 + 算术运算）</td>
            <td><span class="highlight-num">18.6%</span></td>
        </tr>
    </tbody>
</table>

<p>差距高达 63 个百分点。这不是因为这两组题目难度不同——我们后面会通过 <em>Difficulty-Matched</em> 实验证明，即使在赋值链难度相同的情况下，Computing 格式依然更差。</p>

<p>这个差距在 Code Agent 的实际场景里不是抽象的。当一个 agent 在 resolve issue 时，它反复在做两件事：<strong>追踪数据流</strong>（这个变量链式赋值之后是什么值？）和<strong>推理控制流</strong>（这个条件成立，代码走哪条分支？）。如果模型处理这两类操作的内部机制完全不同，agent 的失败模式就会有很强的 <em>task-specificity</em>，而不是随机分布。</p>

<h2>二、任务设计</h2>

<p>我们设计了五类可控的代码理解题目，作为 interpretability 实验的 <em>controlled stimuli</em>：</p>

<p><strong>数据流组：</strong></p>
<ul>
    <li><code>Copying</code>：纯链式变量赋值追踪（例：<code>u = 7; t = u; d = t</code>，问 <code>d</code> 的值）</li>
    <li><code>Computing</code>：链式赋值 + 基本算术（例：<code>a = 2; b = a + 1; c = b + 2</code>，问 <code>c</code> 的值）</li>
</ul>

<p><strong>控制流组：</strong></p>
<ul>
    <li><code>Conditional</code>：if/else 条件判断（执行哪条分支后变量的值？）</li>
    <li><code>Loop</code>：带循环的变量追踪（迭代 n 次后的结果）</li>
    <li><code>Short-circuit</code>：短路求值（<code>and</code>/<code>or</code> 的惰性求值结果）</li>
</ul>

<p>这些题目是程序生成的，答案由 Python 解释器计算保证 ground truth 确定性，且均约束为单数字答案（0-9），以获得干净的 Patchscopes 信号。实验覆盖 <strong>Qwen2.5-Coder 0.5B / 1.5B / 3B / 7B / 14B</strong> 五个规模。</p>

<h2>三、Patchscopes：给模型做逐层 MRI</h2>

<p>本文的核心方法是 <strong>Patchscopes</strong> (Ghandeharioun et al., 2024)，一种通过将模型中间层的 hidden state 注入到另一个 target prompt 来探测该层信息的工具。</p>

<p>具体配置：</p>
<ul>
    <li><strong>Source</strong>：代码题目的 prompt，提取最后 token 位置的 <span style="font-family:serif;">h<sub>&ell;</sub></span>（第 <span style="font-family:serif;">&ell;</span> 层 hidden state）</li>
    <li><strong>Target</strong>：<code>"# The value of x is \""</code> — 一个标准化的解码 prompt</li>
    <li><strong>注入位置</strong>：Target prompt 的 <strong>last position</strong></li>
    <li><strong>评估</strong>：在 target prompt forward 完成后，用 LM head 看第一个 token 是否等于正确答案</li>
</ul>

<p><strong>三个代码场景的方法论适配：</strong></p>
<ol>
    <li><strong>必须用 last-position 注入。</strong> x-position 注入几乎完全失效，因为注入后后续 token（如 <code>&rarr;</code>）的 KV cache 仍基于原始 context 计算，产生信息污染。</li>
    <li><strong>Patchscopes 的解码能力是 task-dependent 的。</strong> 对于高运算密度的 Computing 任务，PS 无法截获 Probing 早早就能探测到的信息，说明“存在”不等于“可解码”。</li>
    <li><strong>大模型需要 first-token 校正。</strong> 14B 模型出现了被生成惯性带偏为长文本幻觉的问题，需依靠 first-token evaluation 修正。</li>
</ol>


<h2>四、主要发现：两种截然不同的处理动力学</h2>

<p>以 7B 模型为例，五个任务的完整数字画像如下：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>维度</th>
            <th>Copying</th>
            <th>Computing</th>
            <th>Conditional</th>
            <th>Loop</th>
            <th>Short-circuit</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Final Acc</td>
            <td>79%</td>
            <td><span class="highlight-num">18.6%</span></td>
            <td>81.7%</td>
            <td>51.6%</td>
            <td>85%</td>
        </tr>
        <tr>
            <td>Ever Correct</td>
            <td>100%</td>
            <td>55.8%</td>
            <td>90%</td>
            <td>59.4%</td>
            <td>95.6%</td>
        </tr>
        <tr>
            <td>Overthinking Gap</td>
            <td>21pp</td>
            <td><span class="highlight-num">37.2pp</span></td>
            <td>8.3pp</td>
            <td>7.8pp</td>
            <td>10.6pp</td>
        </tr>
        <tr>
            <td>Instability</td>
            <td>&approx;0</td>
            <td><span class="highlight-num">0.356</span></td>
            <td>0.22</td>
            <td>0.03</td>
            <td>0.04</td>
        </tr>
        <tr>
            <td>曲线形态</td>
            <td>阶跃式</td>
            <td>闪烁式</td>
            <td>渐进式</td>
            <td>阶跃式</td>
            <td>渐进式</td>
        </tr>
    </tbody>
</table>

<p><em>数据流组内部的分裂最令人惊讶。</em> 两个同属“数据流”的任务，却有着完全相反的动力学特征：<code>Copying</code> 在浅层没有信号，但一旦出现就极度稳定（instability &approx; 0）；而 <code>Computing</code> 极早出现信号，但极度不稳定（instability = 0.356）。</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📊</div>
        <div>[Placeholder: Layer-wise Accuracy Comparison]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 1:</strong> Layer-wise accuracy curves for all five tasks on the 7B model. Notice the step-wise jump of Copying in the late layers versus the flickering oscillating pattern of Computing in mid-layers. Conditional rises smoothly and progressively.
    </div>
</div>

<p><strong>这不是难度问题。</strong>在 <em>Difficulty-Matched</em> 实验中，即使将 Computing 的运算密度降为 <code>d=0</code>（仅为连等赋值），7B 的 Final Acc 也只有 29.2%。这证明代码语法格式本身，激活了更不稳定的处理链路。</p>


<h2>五、Overthinking：算对了，又忘了</h2>

<p>在 Computing 任务上，<strong>7B 模型有 55.8% 的样本在某个中间层曾经输出了正确答案</strong>，但最终层只有 18.6% 正确——这意味着有 <strong>37.2pp</strong> 的正确答案在推理过程中被“想没了”。</p>

<p>在全部 25 个 (Model, Task) 组合上，我们验证了：<strong>25/25 均存在比最终层准确率更高的退出层</strong>（Binomial test p &lt; 10<sup>-7</sup>）。其中，7B 的 Computing 是同模型 Conditional 遗忘现象的 4.5 倍。控制流组整体的 gap 极小，而伴随模型尺寸扩大（如 14B），整体 overthinking 也有所收窄。</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">🔥</div>
        <div>[Placeholder: Overthinking Gap Heatmap]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 2:</strong> Overthinking Gap heatmap (5 models &times; 5 tasks). Shows the exact gap between "Ever Correct" and "Final Acc", highlighting the extreme 37.2pp gap for 7B Computing.
    </div>
</div>

<h2>六、Information Brewing：知道不等于说出来</h2>

<p>对于 Computing 极端遗忘现象的解释，离不开 <em>Information Brewing</em> 假说。我们比对了纯线性 Probing 和 Patchscopes 生成的结果：</p>

<p>在 14B 的 Computing 实验里：</p>
<ul>
    <li>Probing Onset：层数 2（极浅层即可用线性分类器读出确切答案）</li>
    <li>PS 解码起飞：层数 33</li>
    <li><strong>Gap：31 层</strong></li>
</ul>

<p>信息以线性可识别的方式存在于 hidden state 中（Probing 能读出），但并不是以可以直接输出的格式编码的。模型由于需要近 30 层将“计算结果”翻译转换成“Output-ready”表征。这个孕育转换期极度脆弱，一不小心就会被后续 Attention 的噪声完全覆盖（从而 Overthinking）。而像 <code>Copying</code> 任务，其翻译的 Brewing 缝隙几乎为 0 层。</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📈</div>
        <div>[Placeholder: Probing vs Patchscopes Dual Curves]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 3:</strong> Dual curves for 14B model tracking information extractability. The massive horizontal lag (~31 layers) between Probing success and Patchscopes success illustrates the "Information Brewing" period.
    </div>
</div>

<h2>七、Loop 的伪循环理解</h2>

<p>当分析 <code>Loop</code> 这个控制流任务时，出现了戏剧性的一幕。<code>iter=1</code> 时（只执行一次循环），四款较大参数模型给出了全对的精准响应。但一旦 <code>iter&gt;1</code>，它的准确率便发生雪崩式崩溃。</p>

<p>当我们实施展开（Unrolled Loop）后，7B 在 <code>iter=3</code> 阶段骤升 70pp。结论直接呼出欲出：<strong>模型在所谓解决单次循环时，并非应用“循环推断电路”，而是借道了 <code>Copying</code> 的数据流动链路</strong>（Pattern Matching）。一旦超越了浅层的模式识别，缺乏真实迭代理解的劣势便尽现无疑。</p>

<h2>八、Task-Aware Early Exit：外科干预</h2>

<p>如果在特定的代码分析任务中，模型倾向于过度推论直到自我覆写而遗忘答案，那么能否截断推理？依靠 <strong>Best Fixed Layer Exit</strong>（验证集寻找全局极佳层停止前向传播），我们得到了这样的 7B 数据：</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>任务</th>
            <th>原生准确率</th>
            <th>最佳退出层</th>
            <th>退出后准确率</th>
            <th>净收益</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Copying</td>
            <td>79.0%</td>
            <td>L23</td>
            <td><span class="highlight-success">99.0%</span></td>
            <td>+20.0pp</td>
        </tr>
        <tr>
            <td>Computing</td>
            <td>18.6%</td>
            <td>L22</td>
            <td><span class="highlight-success">45.0%</span></td>
            <td>+26.4pp</td>
        </tr>
        <tr>
            <td>Conditional</td>
            <td>81.7%</td>
            <td>L26</td>
            <td>82.7%</td>
            <td>+1.0pp</td>
        </tr>
    </tbody>
</table>

<p>收益最大的任务正是 overthinking 最深的灾区（数据流组）。这种 task-specificity 证明，如果不加甄别对所有代码统一启用单一停顿准则，是不理智的，只有感知了具体任务才能获取收益。</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📊</div>
        <div>[Placeholder: Early-Exit Strategy Bar Chart]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 4:</strong> Strategy chart of 7B model grouping Final (baseline), Best Fixed Layer, Correct-k, and Oracle setups, showing drastic improvement on Computing using Best Fixed.
    </div>
</div>

<h2>九、对 Code Agent 的意义</h2>

<ol>
    <li><strong>数据计算追踪的底层脆弱性</strong>：若你设计的 Agent 屡次在一个涉及数据流极长的大项目中解题失败落榜，其实并非 context retrieval 做错了事，或是模型根本没算对。它算对了，但在随后的过度自省与繁杂的代码文本生成前被彻底涂抹了。</li>
    <li><strong>Task-type Aware 策略</strong>：7B 对待条件控制 80%+ 而单纯追踪计算才不到 20%。当我们在 SWE-Bench 这种掺杂了各种场景的验证集里讨论“平均通过率”时，我们将不可避免地落入混淆内部机制的陷阱里。</li>
    <li><strong>Adaptive 路由前景</strong>：在未来，借由更复杂的 Early-Exit 变体（如 <code>correct-k</code> 或探测到强内部特征分类便拦截），能够帮助大模型规避后续自发推演造成的自我推翻。</li>
</ol>

<h2>十、局限性与展望</h2>

<p>本文的工作也有所局限：
目前实验颗粒度主要驻足在 Layer-level 层级而并未深入至 Attention/MLP 独立运作的计算切片；另外也仅针对了 Qwen 家族。受控刺激实验对于上以万计的库级别项目还原度也较有限。</p>

<p>但这依然带来了底层的方法论启发：当我们凝视一份高分的基准榜单时，看到的只是扁平的“结果”。在这个数字背后，是阶跃、闪烁、酝酿、以及深层生生不息又被重启乃至覆写的信息链条。</p>

<p><strong>你的大模型不是不够聪明没有算清代码。它曾经掌握了答案，只不过一不小心，在到达最终层前忘了告诉你。</strong></p>

</div>


<!-- ================= 英文版 ================= -->
<div class="lang-en lang-content" style="display:none;">

<div class="post-header">
    <h1 class="post-title">Your LLM Understood the Code — Then Forgot the Answer</h1>
    <div class="post-meta">
        By <strong>Yifu Guo</strong> and <strong>Siyue Chen</strong> &middot; March 2026
    </div>
</div>

<p>Code LLMs shatter records on SWE-bench style leaderboards every few months, yet our understanding of <em>how</em> these models process code internally remains profoundly limited. Final accuracy indicates whether a model succeeds, but leaves us blind to the mechanisms of <em>why</em> it fails.</p>

<p>This post details a series of layer-wise interpretability experiments using Patchscopes. We discover that across LLMs, different code tasks do not merely scale on a uniform axis of difficulty; rather, they activate completely heterogeneous processing regimes characterized by disparate information maturation timelines, stability fingerprints, and encoding formats.</p>

<p>Perhaps most strikingly: on certain task profiles, models correctly infer the answer mid-forward-pass, only to actively overwrite and "forget" it prior to emitting tokens.</p>

<h2>I. A Disquieting Gap</h2>

<p>Consider <strong>Qwen2.5-Coder 7B</strong> subjected to identically-sized task scopes:</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Task</th>
            <th>Final Accuracy</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Conditional (Control-flow logic reasoning)</td>
            <td><span class="highlight-success">81.7%</span></td>
        </tr>
        <tr>
            <td>Computing (Data-flow arithmetic tracking)</td>
            <td><span class="highlight-num">18.6%</span></td>
        </tr>
    </tbody>
</table>

<p>This gaping chasm of 63 percentage points is not governed purely by algorithmic difficulty. As we will observe in the <em>Difficulty-Matched</em> section, ensuring structural constraints are isomorphic across test cases demonstrates that the computing syntax itself invokes a profoundly fragile cognitive loop.</p>

<p>When an autonomous code agent tackles a PR, it perpetually bounces between these two axes: <strong>tracking data flows</strong> and <strong>resolving control logic</strong>. Uncovering the internal fissures between these cognitive styles allows us to systematically address task-specific failure modes rather than blindly treating the model as a black box evaluator.</p>

<h2>II. Task Design</h2>

<p>We craft five dimensions of controlled code understanding benchmarks mapped as <em>controlled stimuli</em> for the interpretability lens:</p>

<p><strong>Data-Flow Group:</strong></p>
<ul>
    <li><code>Copying</code>: Chain-assignment variable tracking.</li>
    <li><code>Computing</code>: Chain-assignments heavily injected with arithmetic.</li>
</ul>

<p><strong>Control-Flow Group:</strong></p>
<ul>
    <li><code>Conditional</code>: Standard if/else structural outcomes.</li>
    <li><code>Loop</code>: Deterministic finite loop structures.</li>
    <li><code>Short-circuit</code>: Lazy boolean inferences (<code>and</code>/<code>or</code> predicates).</li>
</ul>

<p>Generated strictly via abstract scripts avoiding data contamination and yielding deterministic single-digit truths (0-9), preventing generative auto-regressive decoding anomalies mapping from 0.5B to 14B model sizes.</p>

<h2>III. Patchscopes Mechanics</h2>

<p>We leverage <strong>Patchscopes</strong> (Ghandeharioun et al., 2024), shifting the hidden state representation <span style="font-family:serif;">h<sub>&ell;</sub></span> from the source code extraction directly onto the parsing target wrapper to explicitly read out intermediate layer convictions without meddling external probes.</p>

<p>Our operational adjustments applied to Code LLMs:</p>
<ol>
    <li><strong>Last-position targeting constraint.</strong> Mid-prompt targeting causes massive KV-cache semantic decay.</li>
    <li><strong>Task-dependent decoding limits.</strong> In intense computing tasks, the decoding process stumbles well past probing recognitions marking that "knowing" vastly pre-dates "decoding logic".</li>
    <li><strong>First-token truncations.</strong> On larger models (viz. 14B), the intrinsic context completion capabilities yield heavy narrative hallucinations bypassing zero-shot answers, necessitating discrete token cropping boundaries.</li>
</ol>

<h2>IV. Polarized Dynamics Across Two Domains</h2>

<p>Profiling identical behaviors on the 7B tier outlines the structural difference.</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Dimension</th>
            <th>Copying</th>
            <th>Computing</th>
            <th>Conditional</th>
            <th>Loop</th>
            <th>Short-circuit</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Final Acc</td>
            <td>79%</td>
            <td><span class="highlight-num">18.6%</span></td>
            <td>81.7%</td>
            <td>51.6%</td>
            <td>85%</td>
        </tr>
        <tr>
            <td>Ever Correct</td>
            <td>100%</td>
            <td>55.8%</td>
            <td>90%</td>
            <td>59.4%</td>
            <td>95.6%</td>
        </tr>
        <tr>
            <td>Overthinking Gap</td>
            <td>21pp</td>
            <td><span class="highlight-num">37.2pp</span></td>
            <td>8.3pp</td>
            <td>7.8pp</td>
            <td>10.6pp</td>
        </tr>
        <tr>
            <td>Instability</td>
            <td>&approx;0</td>
            <td><span class="highlight-num">0.356</span></td>
            <td>0.22</td>
            <td>0.03</td>
            <td>0.04</td>
        </tr>
        <tr>
            <td>Curve Shape</td>
            <td>Stepped</td>
            <td>Flickering</td>
            <td>Progressive</td>
            <td>Stepped</td>
            <td>Progressive</td>
        </tr>
    </tbody>
</table>

<p><em>The data-flow fissure commands massive attention.</em> Whilst <code>Copying</code> manifests via step-wise stability once achieving recognition (instability &approx; 0), <code>Computing</code> surges prematurely yet suffers drastic internal turbulence—a phenomena labeled internal <em>flickering</em>.</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📊</div>
        <div>[Placeholder: Layer-wise Accuracy Comparison]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 1:</strong> Layer-wise accuracy traces mapped sequentially. Shows Copying jumping confidently around the 80% progression depth mark while Computing shudders intensely upon entry to mid-zones.
    </div>
</div>

<p>Applying structural controls via the <em>Difficulty-Matched</em> validation where math density drops to <code>d=0</code>, the model's accuracy settles at a mere 29.2% showing syntax alone dictates the pathway and internal chaos, regardless of complex numerical operation.</p>

<h2>V. Overthinking: Got the Answer, Scrubbed It</h2>

<p>At 7B scale on Computing inputs, precision hits 55.8% somewhere in the middle-tiers yet ultimately sinks to 18.6% upon finalizing generations. An overwhelming <strong>37.2 percentage points</strong> are calculated right, and actively destroyed via subsequent reasoning.</p>

<p>Every single variation of the 25 cross grid variables validated possessed an intermediate layer vastly outshining the model's final inference generation (Binomial test p &lt; 10<sup>-7</sup>). It’s not experimental noise, it is an architectural reality. While Conditional logic logic remains solid across depths, scaling bounds up to 14B somewhat assuages the <em>forgetful phenomenon</em>.</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">🔥</div>
        <div>[Placeholder: Overthinking Gap Heatmap]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 2:</strong> Visualizing the severity. 7B vs Computing burns a staggering visual delta relative to Conditional inference or 14B stability.
    </div>
</div>

<h2>VI. Information Brewing: Knowledge &neq; Output Readiness</h2>

<p>Explaining such erratic behavior requires looking at linear Probing validations versus generation. Taking the 14B computing results:</p>

<ul>
    <li>Probing onset bounds: Layer 2. Information is cleanly separable instantaneously.</li>
    <li>PS generation alignment bounds: Layer 33.</li>
    <li><strong>Resulting Gap: 31 Layers.</strong></li>
</ul>

<p>Information exists, but it demands severe translation sequences through deep network spaces to encode to conversational autoregressive distributions. We term this duration <em>Information Brewing</em>. When these pathways take dozens of layers to articulate the numerical insight to semantic speech, the inherent fragility is constantly at risk from subsequent attention distortions mapping, hence—forgetfulness.</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📈</div>
        <div>[Placeholder: Probing vs Patchscopes Dual Curves]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 3:</strong> The sheer separation of Probing metrics detecting validity rapidly while generation thresholds suffer prolonged delays vividly maps the "Brewing" concept.
    </div>
</div>

<h2>VII. Loop: The Semantic Copying Illusion</h2>

<p>A curious edge case occurred dissecting simple <code>Loop</code> operations. At flat iterations <code>iter=1</code>, models scored heavily up toward perfect marks across all mid-to-high size networks. Stepping to <code>iter&gt;1</code> crushed returns instantly.</p>

<p>Testing semantic equality operations revealed parsing circuits routed short static configurations seamlessly through logical <code>Copying</code> pattern matching paths and immediately decayed once requiring true recursive evaluation. Current language networks fundamentally bypass structural comprehension via superficial proxy semantic mapping.</p>

<h2>VIII. Task-Aware Early Exits: Surgical Stops</h2>

<p>Identifying decay forces strategic interference designs. Evaluating a <strong>Best Fixed Layer Exit</strong> on the 7B parameters reveals deep potential for surgical precision halts:</p>

<table class="academic-table">
    <thead>
        <tr>
            <th>Task</th>
            <th>Native Accuracy</th>
            <th>Optimal Exit</th>
            <th>Post-Exit Accuracy</th>
            <th>Net Yield</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Copying</td>
            <td>79.0%</td>
            <td>L23</td>
            <td><span class="highlight-success">99.0%</span></td>
            <td>+20.0pp</td>
        </tr>
        <tr>
            <td>Computing</td>
            <td>18.6%</td>
            <td>L22</td>
            <td><span class="highlight-success">45.0%</span></td>
            <td>+26.4pp</td>
        </tr>
        <tr>
            <td>Conditional</td>
            <td>81.7%</td>
            <td>L26</td>
            <td>82.7%</td>
            <td>+1.0pp</td>
        </tr>
    </tbody>
</table>

<p>Halting computational tracking layers rapidly curtails destruction forces. Universal implementation, however, lacks foresight. Treating logic and arithmetic homogeneously applies heavy penalties to tasks demanding depth. This forces the demand for code evaluation engines operating strictly dynamically on <em>Task-Aware</em> principles.</p>

<div class="figure-container">
    <div class="figure-box">
        <div class="figure-icon">📊</div>
        <div>[Placeholder: Early-Exit Strategy Bar Chart]</div>
    </div>
    <div class="figure-caption">
        <strong>Figure 4:</strong> Chart showing the net impact over multiple inference routing methodologies.
    </div>
</div>

<h2>IX. Reflections on Code Agents</h2>

<ol>
    <li>When SWE-Bench tasks derail mid-evaluations under deep nested functions, don't just blame prompt constraints. Mathematical mapping pathways simply crumble internally beneath cascading layers.</li>
    <li>Looking blindly at overall logic ratings paints a false picture of underlying mechanism resilience.</li>
    <li>Adaptive routing implementations checking internal representation clustering stability mid-flight could be pivotal in creating highly reactive inference termination modules prior to over-thinking.</li>
</ol>

<h2>X. Future Limitations and Focus</h2>

<p>Granular testing still orbits massive macro layer analyses leaving strict circuit-level routing uncharacterized, primarily tested on Qwen-coded dimensions exclusively. Shifting paradigms into expansive codebase mappings marks fundamental future trajectories.</p>

<p>Behind flattened competitive leaderboard metrics lies a complex reality of step-functions, delayed articulations, rapid overwrites, and fading realizations mapping in the darkest networks architectures.</p>

<p><strong>The model didn't miss what the code was doing, it simply calculated it—then actively buried the truth right before completion.</strong></p>

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

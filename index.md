---
layout: home
title: 论文阅读室
posts_limit: 3
show_excerpts: true
entries_layout: list
---

<section class="survey-hero" aria-labelledby="survey-title">
  <p class="survey-eyebrow">PAPER READING NOTES · MEMORY-EFFICIENT TRAINING</p>
  <h1 id="survey-title">训练深度网络，<br><em>如何把显存还回来？</em></h1>
  <p class="survey-dek">三篇论文，从 checkpoint 的理论边界、任意 DAG 的重计算优化，到运行时 liveness 管理，串起一条面向 Transformer 重复 block 的显存优化路线。</p>

  <div class="survey-meta-grid" aria-label="阅读主题信息">
    <div>
      <span>阅读主题</span>
      <strong>Activation Memory</strong>
    </div>
    <div>
      <span>论文数量</span>
      <strong>3 papers · 2016—2020</strong>
    </div>
    <div>
      <span>落点</span>
      <strong>liveness.py / DP / ILP</strong>
    </div>
  </div>
</section>

<div class="survey-layout">
  <aside class="survey-sidebar" aria-label="本页目录">
    <p class="survey-sidebar-title">ON THIS PAGE</p>
    <nav>
      <a href="#summary"><span>01</span>一句话总结</a>
      <a href="#problem"><span>02</span>问题背景</a>
      <a href="#map"><span>03</span>三篇论文的位置</a>
      <a href="#sublinear"><span>04</span>理论：Sublinear Memory</a>
      <a href="#checkmate"><span>05</span>优化：Checkmate</a>
      <a href="#superneurons"><span>06</span>运行时：SuperNeurons</a>
      <a href="#takeaways"><span>07</span>对当前工作的启示</a>
    </nav>
    <p class="survey-sidebar-note">建议阅读顺序：先看共同抽象，再进入对应论文的完整笔记。</p>
  </aside>

  <div class="survey-body">
    <section id="summary" class="survey-section survey-summary">
      <p class="survey-section-kicker">01 / 一句话总结</p>
      <h2>显存优化的核心，是把“保存”改写成“可重建”。</h2>
      <div class="survey-callout">
        <p>训练时不必让每个 activation 一直留在 GPU 上。只保留少量有价值的状态，在需要时重新计算；再配合生命周期分析、显式内存约束和调度优化，就能在计算时间与显存峰值之间做出可解释的交换。</p>
      </div>
      <div class="survey-stats" aria-label="三篇论文共同关注的维度">
        <div><strong>O(√n)</strong><span>分段 checkpoint 的内存尺度</span></div>
        <div><strong>MILP</strong><span>任意 DAG 的调度表达</span></div>
        <div><strong>Liveness</strong><span>运行时判断 tensor 是否还活跃</span></div>
      </div>
    </section>

    <section id="problem" class="survey-section">
      <p class="survey-section-kicker">02 / 问题背景</p>
      <h2>训练的峰值显存，往往不是参数占用决定的。</h2>
      <p>前向传播产生的中间 activation 要等到反向传播用完之后才能释放。网络越深，保存下来的 tensor 越多；当模型进入 Transformer 这类重复 block 结构后，问题会从“单个算子太大”变成“整个计算图的生命周期太长”。</p>

      <div class="survey-flow" aria-label="训练中 activation 的生命周期">
        <div><b>Forward</b><small>产生 activation</small></div>
        <span aria-hidden="true">→</span>
        <div class="survey-flow-hot"><b>Save</b><small>等待 backward</small></div>
        <span aria-hidden="true">→</span>
        <div><b>Backward</b><small>读取并释放</small></div>
        <span aria-hidden="true">→</span>
        <div class="survey-flow-cool"><b>Recompute</b><small>需要时再建</small></div>
      </div>

      <p class="survey-note">因此，问题可以拆成三个相互连接的子问题：哪些 tensor 值得保存？什么时候可以释放？如果不保存，重算的顺序和代价是什么？</p>
    </section>

    <section id="map" class="survey-section">
      <p class="survey-section-kicker">03 / 三篇论文的位置</p>
      <h2>从理论边界，到通用求解，再到运行时系统。</h2>
      <p>三篇论文并不是三个互相平行的技巧，而是逐渐把问题从“能不能省”推进到“如何安排”，最后落到“系统怎样执行”。</p>

      <div class="survey-paper-map">
        <a href="#sublinear" class="survey-paper-row survey-paper-row-green">
          <span class="survey-paper-number">01</span>
          <span><strong>Training Deep Nets with Sublinear Memory Cost</strong><small>Chen et al. · 2016 · 理论与动态规划</small></span>
          <b>看边界 ↗</b>
        </a>
        <a href="#checkmate" class="survey-paper-row survey-paper-row-purple">
          <span class="survey-paper-number">02</span>
          <span><strong>Checkmate</strong><small>Jain et al. · MLSys 2020 · 任意 DAG 与 MILP</small></span>
          <b>看调度 ↗</b>
        </a>
        <a href="#superneurons" class="survey-paper-row survey-paper-row-blue">
          <span class="survey-paper-number">03</span>
          <span><strong>SuperNeurons</strong><small>Wang et al. · 2018 · liveness 与动态内存池</small></span>
          <b>看运行时 ↗</b>
        </a>
      </div>
    </section>

    <section id="sublinear" class="survey-section survey-section-green">
      <p class="survey-section-kicker">04 / 理论：Checkpointing</p>
      <div class="survey-section-heading">
        <div>
          <h2>Training Deep Nets with Sublinear Memory Cost</h2>
          <p class="survey-paper-subtitle">把深度换成重计算：少存 checkpoint，其余 activation 按需恢复。</p>
        </div>
        <a class="survey-source-link" href="https://arxiv.org/abs/1604.06174" target="_blank" rel="noreferrer">原论文 ↗</a>
      </div>
      <p>Chen 等人先把深层网络视为一条重复的链：前向时每隔一段保存 checkpoint，反向走到某个区间时，从最近的 checkpoint 重新执行前向，恢复当前需要的 activation。</p>
      <div class="survey-formula">
        <span>分段 checkpoint 的直觉</span>
        <strong>M(k) = O(n / k) + O(k)</strong>
        <em>取 k = √n，可得到 O(√n) 的内存尺度；递归策略还能继续逼近 O(log n)。</em>
      </div>
      <div class="survey-two-column">
        <div>
          <h3>为什么适合 Transformer block？</h3>
          <p>重复 block 让每一段的计算模式近似一致：状态转移可以被抽象成相同的 template，checkpoint 位置和重算区间因此可以交给动态规划选择，而不必为每个算子写一套完全不同的规则。</p>
        </div>
        <div>
          <h3>代价是什么？</h3>
          <p>显存下降并不是免费的。重算区间越长，额外 forward 越多；checkpoint 越密，显存越高但时间越省。工程实现还要考虑 in-place 操作、共享 buffer 和 backward 对输入的真实需求。</p>
        </div>
      </div>
      <a class="survey-read-link" href="{{ '/memory-efficient-training/sublinear-memory/' | relative_url }}">阅读完整笔记 →</a>
    </section>

    <section id="checkmate" class="survey-section survey-section-purple">
      <p class="survey-section-kicker">05 / 优化：Tensor Rematerialization</p>
      <div class="survey-section-heading">
        <div>
          <h2>Checkmate：把重计算写成一个调度问题。</h2>
          <p class="survey-paper-subtitle">不再假设计算图是链，而是直接在 arbitrary DAG 上优化 tensor 的生命周期。</p>
        </div>
        <a class="survey-source-link" href="https://proceedings.mlsys.org/paper_files/paper/2020/file/0b816ae8f06f8dd3543dc3d9ef196cab-Paper.pdf" target="_blank" rel="noreferrer">原论文 ↗</a>
      </div>
      <p>Checkmate 的价值在于把不同 tensor size、不同计算成本、不同依赖关系和显存上限放进同一个模型。它不只问“这个 tensor 要不要 checkpoint”，而是问“在每个阶段，应该保存、重算、释放还是保持不动”。</p>
      <div class="survey-variable-grid">
        <div><b>R</b><span>保留 tensor</span></div>
        <div><b>S</b><span>保存 checkpoint</span></div>
        <div><b>U</b><span>执行重算</span></div>
        <div><b>FREE</b><span>释放内存</span></div>
      </div>
      <div class="survey-formula survey-formula-purple">
        <span>调度的约束</span>
        <strong>Σ size(tensor) · live(tensor, stage) ≤ capacity</strong>
        <em>目标是在不超过显存容量的前提下，最小化总计算时间或重算代价。</em>
      </div>
      <div class="survey-two-column">
        <div>
          <h3>它给当前工作的启发</h3>
          <p>profile-based cost model 很适合把 kernel 的实际时间、tensor 大小和显存预算接进求解器，作为“机制正确之后”的真实代价函数。</p>
        </div>
        <div>
          <h3>为什么不直接照搬？</h3>
          <p>完整 MILP 的变量和约束会随节点、边与时间阶段迅速膨胀。对 LLM 规模的重复 Transformer，先抽取 block template，再用结构化 IR 配合 DP/ILP，更有机会保持可求解性。</p>
        </div>
      </div>
      <a class="survey-read-link" href="{{ '/memory-efficient-training/checkmate/' | relative_url }}">阅读完整笔记 →</a>
    </section>

    <section id="superneurons" class="survey-section survey-section-blue">
      <p class="survey-section-kicker">06 / 运行时：Liveness Management</p>
      <div class="survey-section-heading">
        <div>
          <h2>SuperNeurons：让 runtime 知道 tensor 什么时候已经没用了。</h2>
          <p class="survey-paper-subtitle">通过 liveness analysis、统一 tensor pool 和 cost-aware recomputation，降低真实执行中的 peak memory。</p>
        </div>
        <a class="survey-source-link" href="https://arxiv.org/abs/1801.04380" target="_blank" rel="noreferrer">原论文 ↗</a>
      </div>
      <p>SuperNeurons 更接近正在写的 <code>liveness.py</code>：从计算图推导每个 tensor 的 live range，判断它是否还会被后续算子使用，再把已经结束生命周期的空间回收到统一的内存池中。必要时，系统还可以在 offload、prefetch 和 recompute 之间按成本做选择。</p>
      <div class="survey-liveness" aria-label="tensor 生命周期示意">
        <div class="survey-liveness-axis"><span>产生</span><span>中间阶段</span><span>反向读取</span><span>释放</span></div>
        <div><b>x₀</b><i class="life-long"></i><small>跨 block</small></div>
        <div><b>a₁</b><i class="life-medium"></i><small>短期 live</small></div>
        <div><b>w</b><i class="life-join"></i><small>共享输入</small></div>
        <div><b>a₂</b><i class="life-short"></i><small>可回收</small></div>
      </div>
      <div class="survey-two-column">
        <div>
          <h3>对应到实现</h3>
          <p>先做静态 liveness，再将 live set 映射到 tensor pool；之后把每个候选 recompute 的时间、内存收益和带宽成本放进统一的 cost model。</p>
        </div>
        <div>
          <h3>与前两篇的关系</h3>
          <p>Sublinear Memory 给出“重算可行”的理论直觉，Checkmate 给出“如何安排”的全局视角，SuperNeurons 则说明这些判断怎样进入真实 runtime。</p>
        </div>
      </div>
      <a class="survey-read-link" href="{{ '/memory-efficient-training/superneurons/' | relative_url }}">阅读完整笔记 →</a>
    </section>

    <section id="takeaways" class="survey-section survey-conclusion">
      <p class="survey-section-kicker">07 / 对当前工作的启示</p>
      <h2>把 Transformer 的重复结构，变成求解器看得懂的语言。</h2>
      <ol class="survey-next-steps">
        <li><strong>抽取结构：</strong>把重复 block 表达成 template，而不是把整个模型摊平成一个巨大 DAG。</li>
        <li><strong>分析生命周期：</strong>从 forward/backward 的使用关系推导 live set，明确每个 tensor 的释放窗口。</li>
        <li><strong>选择策略：</strong>用 DP 处理重复链上的 checkpoint，用 ILP 表达少量跨 block 的全局约束。</li>
        <li><strong>接入 profile：</strong>让 tensor size、kernel 时间、重算次数与显存 capacity 共同决定最终 schedule。</li>
      </ol>
      <div class="survey-takeaway">
        <span>最终结论</span>
        <p>这三篇论文共同说明：显存优化不是简单地“多存还是少存”，而是让系统同时理解结构、生命周期和代价。你的潜在贡献，正是把通用 DAG 的复杂性收敛到 Transformer block 的结构化求解空间。</p>
      </div>
    </section>
  </div>
</div>

<div class="home-section-heading survey-notes-heading">
  <div>
    <span class="section-kicker">FULL NOTES / 03 PAPERS</span>
    <h2>进入每篇论文的完整笔记</h2>
  </div>
</div>

---
layout: page
title: 论文阅读室
classes:
  - home-reference
---

<div class="reference-home">
  <header class="reference-hero">
    <div class="reference-badge">📚 论文 · 笔记</div>
    <h1>训练系统论文笔记</h1>
    <p>Activation · Liveness · Tensor Rematerialization · GPU Memory</p>
    <div class="reference-stats">
      <div class="reference-stat-pill">论文 <strong>3</strong> 篇</div>
      <div class="reference-stat-pill">📝 阅读笔记 <strong>3</strong></div>
      <div class="reference-stat-pill">🧠 Checkpointing <strong>1</strong></div>
      <div class="reference-stat-pill">⚙️ Recompute <strong>3</strong></div>
      <div class="reference-stat-pill">💾 显存调度 <strong>3</strong></div>
    </div>
  </header>

  <div class="reference-tabs-wrap">
    <div class="reference-tabs" role="tablist" aria-label="论文分类">
      <button class="reference-tab active" type="button" role="tab" aria-selected="true" data-filter="all">
        全部 <span class="reference-tab-count">3</span>
      </button>
      <button class="reference-tab" type="button" role="tab" aria-selected="false" data-filter="foundation">
        <span class="reference-tab-dot dot-green"></span>理论基础 <span class="reference-tab-count">1</span>
      </button>
      <button class="reference-tab" type="button" role="tab" aria-selected="false" data-filter="optimization">
        <span class="reference-tab-dot dot-purple"></span>优化求解 <span class="reference-tab-count">1</span>
      </button>
      <button class="reference-tab" type="button" role="tab" aria-selected="false" data-filter="runtime">
        <span class="reference-tab-dot dot-blue"></span>运行时系统 <span class="reference-tab-count">1</span>
      </button>
    </div>
  </div>

  <div class="reference-search-wrap">
    <input class="reference-search-input" id="reference-search" type="search" placeholder="搜索论文或笔记……（标题 / 关键词 / 方法）" autocomplete="off" />
    <span class="reference-search-icon" aria-hidden="true">⌕</span>
  </div>
  <div class="reference-search-count" id="reference-search-count" aria-live="polite"></div>

  <section class="reference-panel" aria-label="论文列表">
    <div class="reference-category-header">
      <h2>显存优化 · 三篇关键论文</h2>
      <span>从理论、求解到运行时</span>
    </div>

    <div class="reference-paper-list" id="reference-paper-list">
      <a class="reference-paper-card" data-category="foundation" data-search="training deep nets sublinear memory cost chen checkpoint activation recomputation dynamic programming transformer memory 2016 arxiv 1604.06174" href="{{ '/memory-efficient-training/sublinear-memory/' | relative_url }}">
        <div>
          <div class="reference-card-meta">
            <span class="reference-tag tag-year">2016</span>
            <span class="reference-tag tag-source">arXiv:1604.06174</span>
            <span class="reference-tag tag-green">理论基础</span>
            <span class="reference-tag tag-cyan">Checkpointing</span>
          </div>
          <div class="reference-card-title">Training Deep Nets with Sublinear Memory Cost</div>
          <div class="reference-card-subtitle">Chen et al. · Activation Recomputation · Dynamic Programming</div>
          <div class="reference-card-summary">用少量 checkpoint 替代长期保存 activation，把深层网络的内存从线性压到次线性。重点是 recompute 的基本理论、重复 block 为什么适合 DP，以及 checkpoint 数量和额外计算量之间的关系。</div>
        </div>
        <div class="reference-card-arrow" aria-hidden="true">→</div>
      </a>

      <a class="reference-paper-card" data-category="optimization" data-search="checkmate tensor rematerialization arbitrary dag milp memory capacity recompute schedule profile cost model jain mlsys 2020" href="{{ '/memory-efficient-training/checkmate/' | relative_url }}">
        <div>
          <div class="reference-card-meta">
            <span class="reference-tag tag-year">2020</span>
            <span class="reference-tag tag-source">MLSys 2020</span>
            <span class="reference-tag tag-purple">优化求解</span>
            <span class="reference-tag tag-violet">MILP</span>
          </div>
          <div class="reference-card-title">Checkmate：Breaking the Memory Wall with Optimal Tensor Rematerialization</div>
          <div class="reference-card-subtitle">Jain et al. · Arbitrary DAG · Tensor Rematerialization</div>
          <div class="reference-card-summary">把任意 DAG 上的 tensor lifetime、memory capacity、recompute schedule 和 profile-based cost model 统一建模为 MILP。它是当前工作的直接 baseline，也解释了为什么 LLM 规模上需要结构化 IR 加 DP/ILP。</div>
        </div>
        <div class="reference-card-arrow" aria-hidden="true">→</div>
      </a>

      <a class="reference-paper-card" data-category="runtime" data-search="superneurons dynamic gpu memory management liveness analysis unified tensor pool cost aware recomputation offload prefetch wang 2018 arxiv 1801.04380" href="{{ '/memory-efficient-training/superneurons/' | relative_url }}">
        <div>
          <div class="reference-card-meta">
            <span class="reference-tag tag-year">2018</span>
            <span class="reference-tag tag-source">arXiv:1801.04380</span>
            <span class="reference-tag tag-blue">运行时系统</span>
            <span class="reference-tag tag-slate">Liveness</span>
          </div>
          <div class="reference-card-title">SuperNeurons：Dynamic GPU Memory Management for Training Deep Neural Networks</div>
          <div class="reference-card-subtitle">Wang et al. · Liveness Analysis · Unified Tensor Pool</div>
          <div class="reference-card-summary">从计算图推导 tensor 生命周期，结合统一 tensor pool、offload/prefetch 和 cost-aware recomputation，降低真实 runtime 的 peak memory。它最直接对应当前正在写的 <code>liveness.py</code>。</div>
        </div>
        <div class="reference-card-arrow" aria-hidden="true">→</div>
      </a>
    </div>

    <div class="reference-empty" id="reference-empty" hidden>没有找到匹配的论文。试试标题、作者、方法名或关键词。</div>
  </section>

  <footer class="reference-footer">三篇论文，一条显存优化路线：从“保存什么”到“什么时候重算”。</footer>
</div>

<script>
  (() => {
    const tabs = [...document.querySelectorAll('.reference-tab')];
    const cards = [...document.querySelectorAll('.reference-paper-card')];
    const search = document.getElementById('reference-search');
    const count = document.getElementById('reference-search-count');
    const empty = document.getElementById('reference-empty');
    let activeFilter = 'all';

    function render() {
      const query = search.value.trim().toLowerCase();
      let visible = 0;
      cards.forEach((card) => {
        const matchesFilter = activeFilter === 'all' || card.dataset.category === activeFilter;
        const matchesSearch = !query || card.dataset.search.includes(query);
        const show = matchesFilter && matchesSearch;
        card.hidden = !show;
        if (show) visible += 1;
      });
      empty.hidden = visible !== 0;
      count.textContent = query ? `找到 ${visible} 条结果` : '';
    }

    tabs.forEach((tab) => {
      tab.addEventListener('click', () => {
        activeFilter = tab.dataset.filter;
        tabs.forEach((item) => {
          const active = item === tab;
          item.classList.toggle('active', active);
          item.setAttribute('aria-selected', active ? 'true' : 'false');
        });
        render();
      });
    });

    search.addEventListener('input', render);
  })();
</script>

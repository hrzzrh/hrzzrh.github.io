---
title: "Training Deep Nets with Sublinear Memory Cost：把计算换成显存"
date: 2026-09-18 09:00:00 +0800
categories:
  - memory-efficient-training
tags:
  - Checkpointing
  - Activation Recomputation
  - Dynamic Programming
paper_url: "https://arxiv.org/abs/1604.06174"
excerpt: "保存少量 checkpoint，反向时重算被丢弃的 activation；线性链网络的 memory 可以降到 O(√n)，递归方案甚至可以逼近 O(log n)。"
---

<div class="paper-note-header">
  <span>基础工作 · 2016</span>
  <a href="{{ page.paper_url }}" target="_blank" rel="noreferrer">阅读原文 ↗</a>
</div>

<div class="paper-note-lead">
  <div>
    <span>一句话</span>
    <p>不把所有 forward activation 都存到 backward，而是只保留少量 segment endpoint，需要时从最近 checkpoint 重新跑 forward。</p>
  </div>
  <div>
    <span>与你的工作</span>
    <p>Transformer 的重复 block 可以天然提供 stage boundary；checkpoint 数量、block size 与额外重算量可以交给 DP 搜索。</p>
  </div>
</div>

## 我的问题

当网络深度增加时，训练显存为什么会先被 activation 而不是参数打满？如果不保存全部中间结果，反向传播还能不能得到完全相同的梯度？

## 关键机制：分段保存，局部重算

论文先从线性链网络出发。假设一个有 `n` 层的网络被切成 `k` 个 segment：

1. forward 时只保存每个 segment 的边界 checkpoint。
2. backward 处理某个 segment 时，从它的 checkpoint 重新执行 forward。
3. 立即用重算出的局部 activation 做 backward，然后释放局部结果。

<div class="paper-flow" aria-label="分段重计算流程">
  <div><b>checkpoint</b><small>保留边界</small></div>
  <span>→</span>
  <div class="recompute"><b>segment forward</b><small>需要时重算</small></div>
  <span>→</span>
  <div class="backward"><b>local backward</b><small>局部反传</small></div>
</div>

这里的关键不是“少存几个 tensor”这么简单，而是把训练过程重新安排成一张带有额外 forward 节点的 gradient graph。论文还讨论了 in-place operation 和 memory sharing：如果一个输入没有其他未完成的依赖，它的存储就可以被输出复用。

## 最重要的公式：checkpoint 数量是旋钮

额外 memory 由两部分组成：跨 segment 保存的 checkpoint 数量，以及一次 segment 内部反向所需的局部 activation。近似写成：

<div class="paper-formula">
  <span>MEMORY MODEL</span>
  <strong>M(k) = O(n/k) + O(k)</strong>
  <em>k = √n  ⇒  M = O(√n)</em>
</div>

当 `k = √n` 时，两项达到平衡；论文指出，这只需要增加大约一次 forward pass 的计算成本。更激进地递归应用同样的思想，可以得到：

<div class="paper-formula paper-formula-soft">
  <span>RECURSIVE VIEW</span>
  <strong>g(n) = k + g(n / (k + 1))</strong>
  <em>k = 1  ⇒  g(n) = O(log n)</em>
</div>

这给出的不是通常训练配置，而是一个很有价值的边界：memory 和额外计算之间存在可控的连续 trade-off。

## 为什么重复 block 适合 DP

论文主分析的是近似线性的计算图，但它把一个 segment 看成 bulk operator，再递归优化内部子图。Transformer 的重复 block 具有相似结构：

| 结构特征 | 对 planner 的意义 |
| --- | --- |
| block 输入/输出边界稳定 | 可以把 boundary 当作 checkpoint candidate |
| block 内部拓扑重复 | 可以缓存同一 template 的 cost 与 size |
| 每层 activation 形状相近 | DP 的状态空间更容易压缩 |
| block 数量很大 | 全图展开后，结构化求解比逐 node 搜索更有价值 |

<div class="paper-callout paper-callout-green">
  <span>对结构化 IR 的启发</span>
  <p>IR 可以先只暴露 block boundary、activation size、recompute cost 和依赖摘要；DP 在“哪些边界保留”上工作，不必重新理解每一次展开后的整张 DAG。</p>
</div>

## 实验与边界

论文报告在 1,000-layer residual network 上把 feature-map memory 从 48G 降到 7G，额外运行时间约 30%；对长序列 LSTM 也观察到了明显的 memory reduction。

但它依赖几个简化：

- 图结构主要按 linear chain 或可分段的结构处理；
- 层的 memory / compute cost 没有被完整地当作非均匀变量；
- residual、fan-in、fan-out 的生命周期不能只靠等长切段解释。

这正是 Checkmate 需要解决的问题：把 arbitrary DAG、不同 tensor 大小和不同重算成本统一进约束优化。

## 我带走的

checkpoint 数量不是实现细节，而是一个可以被显式搜索的资源预算。对当前工作来说，第一版 planner 不一定要追求全局最优；先证明“重复 block 的边界能把问题压缩成 DP 状态”本身，就已经是很有价值的结果。

<div class="paper-takeaway">
  <strong>下一步</strong>
  <p>用 4–8 个重复 Transformer block 做一个最小实验：比较等长 checkpoint、按 memory 均衡 checkpoint 与 DP 选择的 peak memory 和额外 forward 次数。</p>
</div>

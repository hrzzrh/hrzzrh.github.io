---
title: "Checkmate：用 MILP 找到任意 DAG 的重计算计划"
date: 2026-09-17 09:00:00 +0800
categories:
  - memory-efficient-training
tags:
  - Tensor Rematerialization
  - MILP
  - Arbitrary DAG
paper_url: "https://proceedings.mlsys.org/paper_files/paper/2020/file/0b816ae8f06f8dd3543dc3d9ef196cab-Paper.pdf"
excerpt: "Checkmate 将 tensor lifetime、memory capacity 和 profile-based compute cost 写进 MILP，输出一段可执行的 allocate / compute / deallocate schedule。"
---

<div class="paper-note-header paper-note-header-purple">
  <span>直接 baseline · MLSys 2020</span>
  <a href="{{ page.paper_url }}" target="_blank" rel="noreferrer">阅读原文 PDF ↗</a>
</div>

<div class="paper-note-lead">
  <div>
    <span>一句话</span>
    <p>把“哪些 tensor 在什么时候保留、什么时候重算”形式化成一个带正确性约束和显存预算的 MILP。</p>
  </div>
  <div>
    <span>与你的工作</span>
    <p>它是最直接的 baseline：你的潜在区别不是重新发明 rematerialization，而是利用 Transformer block template 缩小求解空间。</p>
  </div>
</div>

## 我的问题

当计算图不再是线性链，而是包含 residual、skip connection、fan-out 和 join 时，如何同时回答三个问题：

- 哪些 tensor 必须跨阶段保留？
- 哪些 tensor 可以在 backward 前重算？
- 在给定显存预算下，哪种 schedule 的额外时间最小？

## Checkmate 的抽象：stage × node

Checkmate 不直接搜索一个模糊的“省显存策略”，而是把一次 training iteration 投影成多个 stage。核心变量可以这样理解：

| 变量 | 含义 | 决策层 |
| --- | --- | --- |
| `R[t,k]` | stage `t` 是否计算 node `k` | recompute choice |
| `S[t,i]` | tensor `i` 是否作为下一个 stage 的 checkpoint | checkpoint choice |
| `U[t,k]` | 执行 node `k` 后的 resident memory | lifetime accounting |
| `FREE[t,i,k]` | tensor `i` 是否能在 `k` 后释放 | deallocation |

这组变量把 tensor 的生命周期显式放进了优化问题：存活不是一个静态属性，而是由未来的 users、checkpoint 选择和当前 stage 的执行决定。

## 约束：正确性先于最优性

目标函数不是简单地最小化重算次数，而是用硬件 profile 得到的 `Cᵢ` 最小化一次 iteration 的计算时间。关键约束包括：

<div class="paper-formula paper-formula-purple">
  <span>OBJECTIVE</span>
  <strong>min Σ Cᵢ · R[t,i]</strong>
  <em>最小化 profile-based runtime</em>
</div>

<div class="paper-formula paper-formula-purple">
  <span>MEMORY CAPACITY</span>
  <strong>U[t,k] ≤ M<sub>budget</sub></strong>
  <em>任意时刻 resident tensors 都必须放进显存</em>
</div>

<div class="paper-formula paper-formula-purple">
  <span>STATE UPDATE</span>
  <strong>U[t,k+1] = U[t,k] − freed + R[t,k+1] · M<sub>k+1</sub></strong>
  <em>释放多少、再分配多少，逐步更新 resident memory</em>
</div>

`FREE` 变量尤其重要：它确保一个 tensor 只有在没有下游依赖、没有被选为下一个 checkpoint 时才能释放，也避免同一 tensor 被重复释放。

## 解出来的是一段程序

得到 `R / S / FREE` 后，Checkmate 会用 row-major scan 生成具体 execution plan：

<div class="paper-flow paper-flow-purple" aria-label="Checkmate execution plan">
  <div><b>allocate</b><small>virtual register</small></div>
  <span>→</span>
  <div class="recompute"><b>compute</b><small>materialize tensor</small></div>
  <span>→</span>
  <div class="backward"><b>deallocate</b><small>release register</small></div>
  <span>↺</span>
  <div><b>next stage</b><small>必要时重算</small></div>
</div>

这意味着优化器的输出不是一个“理论上可行的 checkpoint 集合”，而是一段可以解释执行、也可以重新编码成静态 computation graph 的 schedule。

## 为什么 arbitrary-DAG MILP 对 LLM scale 不够理想

它的通用性来自显式表示每个 node、edge 与 stage；代价是变量和约束随图规模增长。论文给出的完整 ILP 有 `O(|V||E|)` 规模的变量与约束，绝大多数案例可以在分钟内求解，但大 batch 或极紧 memory budget 可能触及一小时的 time limit。

<div class="paper-callout paper-callout-purple">
  <span>潜在区别</span>
  <p>不把 Transformer 展开图视作一张完全任意的 DAG，而是先抽取 repeated block template：把“结构”交给 IR，把“预算内选点”交给 DP / ILP，只在必要范围内回到 tensor-level schedule。</p>
</div>

## 论文结果与我关注的接口

Checkmate 使用 accelerator-specific 的 profile-based cost model，并报告了最高 5.1× 更大的输入尺寸。它真正值得复用的不是某个 solver 参数，而是下面这个接口边界：

| 输入 | 求解器 | 输出 |
| --- | --- | --- |
| DAG、tensor size、compute cost、memory budget | correctness + capacity + time optimization | allocate / compute / deallocate schedule |
| Transformer block IR | DP 或局部 ILP | block boundary checkpoint plan |

对当前工作，最自然的工程拆分是：先让 `liveness.py` 产出可信的 lifetime 与 peak memory，再让 planner 只负责在候选边界上做策略选择。

## 我带走的

Checkmate 把“显存优化”从 heuristic 变成了一个可以被验证的调度问题。你的潜在研究空间在于：Transformer 的重复结构可能允许一个比 arbitrary-DAG MILP 更小、更可解释的 search space。

<div class="paper-takeaway paper-takeaway-purple">
  <strong>下一步</strong>
  <p>把同一组 block 展开成 full DAG 与 structured IR 两个版本，比较求解时间、peak memory 和 schedule quality，而不是只比较最终显存数字。</p>
</div>

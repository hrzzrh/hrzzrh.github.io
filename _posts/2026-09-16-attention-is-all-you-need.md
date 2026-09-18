---
title: "Attention Is All You Need：把序列建模改写成关系建模"
date: 2026-09-16 09:00:00 +0800
categories:
  - deep-learning
tags:
  - Transformer
  - Attention
  - Architecture
paper_url: "https://arxiv.org/abs/1706.03762"
excerpt: "用自注意力替代循环结构，让序列建模第一次真正围绕‘关系’而不是‘顺序’展开。"
---

<div class="paper-note-header">
  <span>论文摘要</span>
  <a href="{{ page.paper_url }}" target="_blank" rel="noreferrer">阅读原文 ↗</a>
</div>

## 我的问题

如果不再按时间步逐个处理 token，模型如何理解一个序列里远距离的依赖关系？

## 关键观点

Transformer 把信息交互的基本单位从“状态传递”换成了“位置之间的注意力”。多头注意力让模型可以在不同子空间里同时寻找关系，残差连接和位置编码则补上了优化与顺序信息。

## 我带走的

架构的突破往往来自重新定义信息交互的方式，而不只是堆叠更大的模型。以后读到新的模型结构时，我会先问：它允许哪些信息更容易相遇？

## 还想追问

注意力矩阵里的关系到底有多少是稳定、可解释的结构，又有多少只是任务和数据共同诱导出的捷径？

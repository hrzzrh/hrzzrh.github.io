---
layout: page
title: 笔记
permalink: /notes/
show_excerpts: true
entries_layout: list
---

<div class="listing-intro">
  <span class="section-kicker">READING LOG / 01</span>
  <p>每篇笔记都只保留最值得带走的部分：它解决了什么、证据在哪里，以及我还想继续追问什么。</p>
</div>

<div class="entries-list notes-page-list">
  {% for entry in site.posts %}
    {% include entry.html %}
  {% endfor %}
</div>

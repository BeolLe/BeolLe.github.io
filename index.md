---
layout: page
title: Ronny's Blog
---

공부용 깃허브 블로그입니다.

<style>
  .rpg-card {
    background: #1e1e1e;
    color: #e0e0e0;
    padding: 20px;
    border-radius: 10px;
    border: 2px solid #4a4a4a;
    font-family: 'Courier New', Courier, monospace;
    width: 100%;
    max-width: 400px;
    margin: 30px auto;
    box-shadow: 0 4px 8px rgba(0,0,0,0.5);
  }
  .rpg-title {
    text-align: center;
    border-bottom: 1px solid #4a4a4a;
    margin-bottom: 15px;
    padding-bottom: 5px;
    font-weight: bold;
    color: #ffd700;
    font-size: 1.2em;
  }
  .stat-row {
    margin-bottom: 15px;
  }
  .stat-label {
    display: flex;
    justify-content: space-between;
    font-size: 0.9em;
    margin-bottom: 5px;
    font-weight: bold;
  }
  .progress-bg {
    background: #333;
    height: 14px;
    border-radius: 7px;
    overflow: hidden;
    position: relative;
    border: 1px solid #555;
  }
  .progress-fill {
    height: 100%;
    border-radius: 7px 0 0 7px;
    transition: width 0.5s ease-in-out;
  }
  /* 스탯 색상 */
  .str-color { background: linear-gradient(90deg, #ff4d4d, #ff1a1a); }
  .int-color { background: linear-gradient(90deg, #4da6ff, #0073e6); }
  .vit-color { background: linear-gradient(90deg, #4dff88, #00b33c); }
  .dex-color { background: linear-gradient(90deg, #ffb84d, #e68a00); }
</style>

{% assign str_tags = "build" | split: "," %}
{% assign int_tags = "knowledge" | split: "," %}
{% assign vit_tags = "workout" | split: "," %}
{% assign dex_tags = "problem" | split: "," %}

{% assign str_exp = 0 %}
{% for t in str_tags %}{% assign str_exp = str_exp | plus: site.tags[t].size %}{% endfor %}

{% assign int_exp = 0 %}
{% for t in int_tags %}{% assign int_exp = int_exp | plus: site.tags[t].size %}{% endfor %}

{% assign vit_exp = 0 %}
{% for t in vit_tags %}{% assign vit_exp = vit_exp | plus: site.tags[t].size %}{% endfor %}

{% assign dex_exp = 0 %}
{% for t in dex_tags %}{% assign dex_exp = dex_exp | plus: site.tags[t].size %}{% endfor %}

{% assign max_exp = 10 %}

<div class="rpg-card">
  <div class="rpg-title">RONNY'S STATUS</div>

  <div class="stat-row">
    <div class="stat-label">
      <span>STR (Build)</span>
      <span>{{ str_exp }} / {{ max_exp }}</span>
    </div>
    <div class="progress-bg">
      <div class="progress-fill str-color" style="width: calc(({{ str_exp }} / {{ max_exp }}) * 100%);"></div>
    </div>
  </div>

  <div class="stat-row">
    <div class="stat-label">
      <span>INT (Knowledge)</span>
      <span>{{ int_exp }} / {{ max_exp }}</span>
    </div>
    <div class="progress-bg">
      <div class="progress-fill int-color" style="width: calc(({{ int_exp }} / {{ max_exp }}) * 100%);"></div>
    </div>
  </div>

  <div class="stat-row">
    <div class="stat-label">
      <span>VIT (Workout)</span>
      <span>{{ vit_exp }} / {{ max_exp }}</span>
    </div>
    <div class="progress-bg">
      <div class="progress-fill vit-color" style="width: calc(({{ vit_exp }} / {{ max_exp }}) * 100%);"></div>
    </div>
  </div>

   <div class="stat-row">
    <div class="stat-label">
      <span>DEX (Problem)</span>
      <span>{{ dex_exp }} / {{ max_exp }}</span>
    </div>
    <div class="progress-bg">
      <div class="progress-fill dex-color" style="width: calc(({{ dex_exp }} / {{ max_exp }}) * 100%);"></div>
    </div>
  </div>
</div>
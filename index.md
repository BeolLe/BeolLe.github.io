---
layout: page
title: Ronny's Blog
---

공부용 깃허브 블로그입니다.

<style>
  /* 3단 레이아웃 컨테이너 */
  .dashboard-container {
    display: flex;
    align-items: flex-start; /* 위쪽 라인 맞춤 */
    gap: 20px;
    max-width: 1000px;
    margin: 40px auto;
    background: #121212; /* 전체 배경 어둡게 */
    padding: 20px;
    border-radius: 15px;
    border: 1px solid #333;
    box-shadow: 0 10px 30px rgba(0,0,0,0.5);
    color: #e0e0e0;
    font-family: 'Courier New', Courier, monospace;
  }

  /* 1. 왼쪽: 캐릭터 영역 */
  .col-left {
    flex: 0 0 200px; /* 너비 고정 */
    text-align: center;
  }
  .char-img {
    width: 100%;
    border-radius: 10px;
    border: 3px solid #444;
  }
  .char-level {
    margin-top: 10px;
    background: #ffd700;
    color: #000;
    font-weight: bold;
    padding: 5px;
    border-radius: 5px;
  }

  /* 2. 중앙: 스탯 바 영역 */
  .col-center {
    flex: 1; /* 남은 공간의 절반 차지 */
    padding: 0 10px;
    border-right: 1px dashed #444; /* 오른쪽 구분선 */
  }

  /* 3. 오른쪽: 설명 영역 */
  .col-right {
    flex: 1; /* 남은 공간의 절반 차지 */
    padding-left: 10px;
    font-size: 0.85em;
    display: flex;
    flex-direction: column;
    justify-content: space-around; /* 세로 간격 균등 분배 */
    height: 100%; /* 높이 꽉 채우기 */
  }

  /* 공통 스타일 */
  .section-title {
    text-align: center;
    color: #888;
    margin-bottom: 15px;
    font-size: 0.9em;
    border-bottom: 1px solid #333;
    padding-bottom: 5px;
  }

  .stat-row { margin-bottom: 15px; }
  .stat-label {
    display: flex; justify-content: space-between;
    font-size: 0.9em; margin-bottom: 5px; font-weight: bold;
  }
  .progress-bg {
    background: #222; height: 12px; border-radius: 6px;
    overflow: hidden; border: 1px solid #444;
  }
  .progress-fill { height: 100%; border-radius: 6px 0 0 6px; }

  /* 설명 텍스트 스타일 */
  .desc-row {
    margin-bottom: 15px;
    padding: 8px;
    background: #1e1e1e;
    border-radius: 5px;
    border-left: 3px solid #555;
  }
  .desc-title { color: #ffd700; font-weight: bold; margin-right: 5px; }
  .desc-detail { color: #aaa; display: block; margin-top: 3px; font-size: 0.9em;}

  /* 스탯 색상 */
  .str-color { background: #ff4d4d; border-color: #ff4d4d; }
  .int-color { background: #4da6ff; border-color: #4da6ff; }
  .vit-color { background: #4dff88; border-color: #4dff88; }
  .dex-color { background: #ffb84d; border-color: #ffb84d; }

  /* 모바일 대응 */
  @media (max-width: 768px) {
    .dashboard-container { flex-direction: column; align-items: center; }
    .col-center, .col-right { width: 100%; border: none; }
    .col-left { margin-bottom: 20px; }
  }
</style>

{% assign str_tags = "build" | split: "," %}
{% assign int_tags = "knowledge" | split: "," %}
{% assign vit_tags = "workout" | split: "," %}
{% assign dex_tags = "problem" | split: "," %}

{% assign str_raw = 0 %}{% for t in str_tags %}{% assign str_raw = str_raw | plus: site.tags[t].size %}{% endfor %}
{% assign int_raw = 0 %}{% for t in int_tags %}{% assign int_raw = int_raw | plus: site.tags[t].size %}{% endfor %}
{% assign vit_raw = 0 %}{% for t in vit_tags %}{% assign vit_raw = vit_raw | plus: site.tags[t].size %}{% endfor %}
{% assign dex_raw = 0 %}{% for t in dex_tags %}{% assign dex_raw = dex_raw | plus: site.tags[t].size %}{% endfor %}

{% assign max_stat = 10 %}

{% if str_raw > 10 %}{% assign str_step = 2 %}{% else %}{% assign str_step = 1 %}{% endif %}
{% if int_raw > 10 %}{% assign int_step = 2 %}{% else %}{% assign int_step = 1 %}{% endif %}
{% if vit_raw > 10 %}{% assign vit_step = 2 %}{% else %}{% assign vit_step = 1 %}{% endif %}
{% if dex_raw > 10 %}{% assign dex_step = 2 %}{% else %}{% assign dex_step = 1 %}{% endif %}

{% assign str_val = str_raw | divided_by: str_step %}
{% assign int_val = int_raw | divided_by: int_step %}
{% assign vit_val = vit_raw | divided_by: vit_step %}
{% assign dex_val = dex_raw | divided_by: dex_step %}


<div class="dashboard-container">
  
  <div class="col-left">
    <img src="/img/profile.png" alt="My Character" class="char-img" onerror="this.src='https://via.placeholder.com/200x250?text=No+Image'">
    <div class="char-level">TOTAL LV. {{ str_val | plus: int_val | plus: vit_val | plus: dex_val }}</div>
  </div>

  <div class="col-center">
    <div class="section-title">STATS INFO</div>

    <div class="stat-row">
      <div class="stat-label"><span>STR</span> <span>{{ str_val }} / {{ max_stat }}</span></div>
      <div class="progress-bg"><div class="progress-fill str-color" style="width: calc(({{ str_val }} / {{ max_stat }}) * 100%);"></div></div>
    </div>

    <div class="stat-row">
      <div class="stat-label"><span>INT</span> <span>{{ int_val }} / {{ max_stat }}</span></div>
      <div class="progress-bg"><div class="progress-fill int-color" style="width: calc(({{ int_val }} / {{ max_stat }}) * 100%);"></div></div>
    </div>

    <div class="stat-row">
      <div class="stat-label"><span>VIT</span> <span>{{ vit_val }} / {{ max_stat }}</span></div>
      <div class="progress-bg"><div class="progress-fill vit-color" style="width: calc(({{ vit_val }} / {{ max_stat }}) * 100%);"></div></div>
    </div>

    <div class="stat-row">
      <div class="stat-label"><span>DEX</span> <span>{{ dex_val }} / {{ max_stat }}</span></div>
      <div class="progress-bg"><div class="progress-fill dex-color" style="width: calc(({{ dex_val }} / {{ max_stat }}) * 100%);"></div></div>
    </div>
  </div>

  <div class="col-right">
    <div class="section-title">DETAILS</div>

    <div class="desc-row" style="border-left-color: #ff4d4d;">
      <span class="desc-title">STR (Build)</span>
      <span class="desc-detail">무언가 만들어본 경험</span>
      <span class="desc-detail" style="font-size:0.8em; color:#666;">
        👉 현재 {{ str_step }}회당 1pt (총 {{ str_raw }}회)
      </span>
    </div>

    <div class="desc-row" style="border-left-color: #4da6ff;">
      <span class="desc-title">INT (Know)</span>
      <span class="desc-detail">새로운 지식 습득</span>
      <span class="desc-detail" style="font-size:0.8em; color:#666;">
        👉 현재 {{ int_step }}회당 1pt (총 {{ int_raw }}회)
      </span>
    </div>

    <div class="desc-row" style="border-left-color: #4dff88;">
      <span class="desc-title">VIT (Work)</span>
      <span class="desc-detail">꾸준한 운동 기록</span>
      <span class="desc-detail" style="font-size:0.8em; color:#666;">
        👉 현재 {{ vit_step }}회당 1pt (총 {{ vit_raw }}회)
      </span>
    </div>

    <div class="desc-row" style="border-left-color: #ffb84d;">
      <span class="desc-title">DEX (Prob)</span>
      <span class="desc-detail">문제 해결 및 디버깅</span>
      <span class="desc-detail" style="font-size:0.8em; color:#666;">
        👉 현재 {{ dex_step }}회당 1pt (총 {{ dex_raw }}회)
      </span>
    </div>

  </div>

</div>
---
layout: page
title: 오늘의 목표
permalink: /goals/
topnav: topnav
---

<div id="today-goal-box" style="margin-bottom:20px;"></div>

<div id="today-timetable-box" style="margin-bottom:30px; padding: 10px; background-color: #f9f9f9; border-radius: 5px; display: none;">
  <h3 style="margin-top: 0;">📅 오늘의 시간표</h3>
  <ul id="timetable-list" style="list-style: none; padding-left: 5px;"></ul>
</div>

<h2>이번 달 달력</h2>
<div id="goals-calendar"></div>

<p style="font-size:12px; margin-top:8px;">
  ✔ 달성 &nbsp; ● 목표 있음 &nbsp; ✘ 실패 &nbsp; ○ 목표 없음
</p>

<script>
  // 1. 데이터 로드 (Liquid 구문)
  const GOALS = {
    {% for goal in site.goals %}
      "{{ goal.date | date: '%Y-%m-%d' }}": {
        text: {{ goal.content | strip_newlines | jsonify }},
        status: {{ goal.status | default: 'planned' | jsonify }},
        timetable: {% if goal.timetable %}{{ goal.timetable | jsonify }}{% else %}[]{% endif %}
      }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

  // 2. KST 날짜 계산 함수 보정
  function getKST() {
    const now = new Date();
    // 현재 시간의 타임스탬프에 9시간 추가
    return new Date(now.getTime() + (9 * 60 * 60 * 1000));
  }

  function getKSTDateStr(dateObj) {
    // getKST()로 생성된 객체는 이미 9시간이 더해졌으므로 getUTC 계열 함수 사용이 안전함
    const y = dateObj.getUTCFullYear();
    const m = String(dateObj.getUTCMonth() + 1).padStart(2, '0');
    const d = String(dateObj.getUTCDate()).padStart(2, '0');
    return `${y}-${m}-${d}`;
  }

  // 3. 오늘의 목표 및 시간표 렌더링
  function renderTodayGoal() {
    const today = getKSTDateStr(getKST());
    const box = document.getElementById("today-goal-box");
    const timetableContainer = document.getElementById("today-timetable-box");
    const timetableList = document.getElementById("timetable-list");
    const goal = GOALS[today] || null;

    if (!goal) {
      box.innerHTML = "📋 오늘의 목표: <strong>미정!</strong>";
      timetableContainer.style.display = "none";
      return;
    }

    let icon = (goal.status === "done") ? "✅" : (goal.status === "missed" ? "❌" : "📝");

    box.innerHTML = `
      <div>
        ${icon} 오늘의 목표: <strong>${goal.text}</strong><br>
        <label style="font-size:14px; margin-top:6px; display:inline-block;">
          <input type="checkbox" disabled ${goal.status === "done" ? "checked" : ""}>
        </label>
      </div>
    `;

    if (goal.timetable && goal.timetable.length > 0) {
      timetableContainer.style.display = "block";
      timetableList.innerHTML = goal.timetable.map(item => 
        `<li style="margin-bottom: 5px;"><strong>${item.time}</strong> : ${item.task}</li>`
      ).join('');
    } else {
      timetableContainer.style.display = "none";
    }
  }

  // 4. 달력 렌더링 (12월 자동 갱신 및 정렬 보정)
  function buildCalendar() {
    const todayKST = getKST();
    const year = todayKST.getUTCFullYear();
    const month = todayKST.getUTCMonth(); // 0-11

    const container = document.getElementById("goals-calendar");
    container.innerHTML = "";

    const first = new Date(year, month, 1);
    const lastDate = new Date(year, month + 1, 0).getDate();

    const table = document.createElement("table");
    table.style.borderCollapse = "collapse";
    table.style.width = "100%";
    table.style.tableLayout = "fixed"; // 정렬 핵심 속성

    table.innerHTML = `
      <tr>
        <th style="text-align: center; padding: 10px 0;">일</th>
        <th style="text-align: center; padding: 10px 0;">월</th>
        <th style="text-align: center; padding: 10px 0;">화</th>
        <th style="text-align: center; padding: 10px 0;">수</th>
        <th style="text-align: center; padding: 10px 0;">목</th>
        <th style="text-align: center; padding: 10px 0;">금</th>
        <th style="text-align: center; padding: 10px 0;">토</th>
      </tr>
    `;

    let row = document.createElement("tr");

    for (let i = 0; i < first.getDay(); i++) {
      row.appendChild(document.createElement("td"));
    }

    function markFor(dateStr) {
      const g = GOALS[dateStr];
      if (!g) return "○";
      if (g.status === "done") return "✔";
      if (g.status === "missed") return "✘";
      return "●";
    }

    for (let d = 1; d <= lastDate; d++) {
      const cell = document.createElement("td");
      cell.style.padding = "10px 0";
      cell.style.textAlign = "center";
      cell.style.borderTop = "1px solid #eee";

      const ds = `${year}-${String(month+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
      const mark = markFor(ds);

      // 오늘 날짜 강조
      if (ds === getKSTDateStr(getKST())) {
        cell.style.backgroundColor = "#fff9db";
      }

      cell.innerHTML = `<strong>${d}</strong><br><span style="font-size:12px">${mark}</span>`;
      row.appendChild(cell);

      if ((first.getDay() + d) % 7 === 0) {
        table.appendChild(row);
        row = document.createElement("tr");
      }
    }

    // 빈칸 채우기
    while (row.children.length > 0 && row.children.length < 7) {
      row.appendChild(document.createElement("td"));
    }

    table.appendChild(row);
    container.appendChild(table);
  }

  // 실행
  renderTodayGoal();
  buildCalendar();
</script>
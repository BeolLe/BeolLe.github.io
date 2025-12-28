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
  // 1) Jekyll 컬렉션 -> JS 객체 (timetable 추가)
  const GOALS = {
    {% for goal in site.goals %}
      "{{ goal.date | date: '%Y-%m-%d' }}": {
        text: {{ goal.content | strip_newlines | jsonify }},
        status: {{ goal.status | default: 'planned' | jsonify }},
        timetable: {% if goal.timetable %}{{ goal.timetable | jsonify }}{% else %}[]{% endif %}
      }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  };

  function getKST() {
    const now = new Date();
    return new Date(now.getTime() + 9 * 60 * 60 * 1000); // UTC+9
  }

  function getKSTDateStr(dateObj) {
    return dateObj.toISOString().slice(0,10); // YYYY-MM-DD
  }

  // 2) 오늘의 목표 및 시간표 렌더링
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

    let icon = "";
    if (goal.status === "done") icon = "✅";
    else if (goal.status === "missed") icon = "❌";
    else icon = "📝";

    box.innerHTML = `
      <div>
        ${icon} 오늘의 목표: <strong>${goal.text}</strong><br>
        <label style="font-size:14px; margin-top:6px; display:inline-block;">
          <input type="checkbox" disabled
            ${goal.status === "done" ? "checked" : ""}>
          (status: ${goal.status})
        </label>
      </div>
    `;

    // 시간표 렌더링 로직 추가
    if (goal.timetable && goal.timetable.length > 0) {
      timetableContainer.style.display = "block";
      timetableList.innerHTML = goal.timetable.map(item => 
        `<li style="margin-bottom: 5px;"><strong>${item.time}</strong> : ${item.task}</li>`
      ).join('');
    } else {
      timetableContainer.style.display = "none";
    }
  }

  // 3) 달력 렌더링 (기존 코드 유지)
  function buildCalendar() {
    const todayKST = getKST();
    const year = todayKST.getFullYear();
    const month = todayKST.getMonth();

    const container = document.getElementById("goals-calendar");
    container.innerHTML = "";

    const first = new Date(year, month, 1);
    const lastDate = new Date(year, month + 1, 0).getDate();

    const table = document.createElement("table");
    table.style.borderCollapse = "collapse";
    table.style.width = "100%";
    table.innerHTML = `
      <tr>
        <th>일</th><th>월</th><th>화</th><th>수</th>
        <th>목</th><th>금</th><th>토</th>
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
      cell.style.padding = "6px";
      cell.style.textAlign = "center";

      const ds = `${year}-${String(month+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
      const mark = markFor(ds);

      cell.innerHTML = `${d}<br><span style="font-size:12px">${mark}</span>`;
      row.appendChild(cell);

      if ((first.getDay() + d) % 7 === 0) {
        table.appendChild(row);
        row = document.createElement("tr");
      }
    }

    table.appendChild(row);
    container.appendChild(table);
  }

  renderTodayGoal();
  buildCalendar();
</script>
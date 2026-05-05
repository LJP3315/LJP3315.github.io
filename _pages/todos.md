---
layout: single
title: "待办事项"
permalink: /todos/
---

<style>
  .todo-list {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .todo-item {
    display: flex;
    align-items: flex-start;
    padding: 16px 20px;
    margin-bottom: 8px;
    border-radius: 8px;
    background: #ffffff;
    border: 1px solid #e1e4e8;
    transition: box-shadow 0.2s;
  }

  .todo-item:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  .todo-icon {
    font-size: 1.5em;
    margin-right: 16px;
    flex-shrink: 0;
    line-height: 1.4;
  }

  .todo-content {
    flex: 1;
    min-width: 0;
  }

  .todo-name {
    font-size: 1.05em;
    font-weight: 500;
    color: #24292e;
    margin: 0 0 4px 0;
  }

  .todo-date {
    font-size: 0.85em;
    color: #6a737d;
    margin: 0;
  }

  .todo-countdown {
    flex-shrink: 0;
    font-size: 0.95em;
    font-weight: 600;
    text-align: center;
    padding: 6px 14px;
    border-radius: 20px;
    min-width: 70px;
  }

  /* 未到期 - 蓝色 */
  .todo-countdown.upcoming {
    color: #0366d6;
    background: #f1f8ff;
  }

  /* 紧急（7天内）- 红色 */
  .todo-countdown.urgent {
    color: #b31d28;
    background: #ffeef0;
  }

  /* 已过期 - 灰色 */
  .todo-item.expired {
    opacity: 0.55;
  }

  .todo-item.expired .todo-name {
    text-decoration: line-through;
    color: #959da5;
  }

  .todo-countdown.expired {
    color: #959da5;
    background: #f6f8fa;
  }

  .todo-section-title {
    font-size: 0.8em;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: #959da5;
    margin: 24px 0 8px 0;
    padding-bottom: 4px;
    border-bottom: 1px solid #e1e4e8;
  }

  .todo-section-title:first-child {
    margin-top: 0;
  }

  .todo-empty {
    text-align: center;
    color: #959da5;
    padding: 40px 20px;
    font-style: italic;
  }
</style>

<div class="todo-page">
  <p style="color: #6a737d; margin-bottom: 20px;">自动计算剩余天数，数据维护在 <code>_data/todos.yml</code></p>

  <div id="todo-container"></div>
</div>

<script>
(function () {
  var todos = [
    {% for todo in site.data.todos %}
      { name: {{ todo.name | jsonify }}, date: "{{ todo.date }}", icon: {{ todo.icon | default: "📌" | jsonify }} }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  var today = new Date();
  today.setHours(0, 0, 0, 0);

  var upcoming = [];
  var expired = [];

  todos.forEach(function (todo) {
    var target = new Date(todo.date + "T00:00:00");
    var diff = Math.ceil((target - today) / (1000 * 60 * 60 * 24));
    todo.daysLeft = diff;
    if (diff >= 0) {
      upcoming.push(todo);
    } else {
      expired.push(todo);
    }
  });

  // 按日期排序
  upcoming.sort(function (a, b) { return a.daysLeft - b.daysLeft; });
  expired.sort(function (a, b) { return b.daysLeft - a.daysLeft; });

  var container = document.getElementById("todo-container");
  var html = "";

  function renderSection(title, items) {
    if (items.length === 0) return "";
    var s = '<h3 class="todo-section-title">' + title + "</h3>";
    s += '<ul class="todo-list">';
    items.forEach(function (todo) {
      var isExpired = todo.daysLeft < 0;
      var isUrgent = !isExpired && todo.daysLeft <= 7;

      var itemClass = "todo-item";
      var countdownClass = "todo-countdown";

      if (isExpired) {
        itemClass += " expired";
        countdownClass += " expired";
      } else if (isUrgent) {
        countdownClass += " urgent";
      } else {
        countdownClass += " upcoming";
      }

      var label = "";
      if (isExpired) {
        label = "已过期 " + Math.abs(todo.daysLeft) + " 天";
      } else if (todo.daysLeft === 0) {
        label = "就是今天";
      } else {
        label = "还剩 " + todo.daysLeft + " 天";
      }

      // 格式化日期显示
      var dateParts = todo.date.split("-");
      var dateStr = dateParts[0] + " 年 " + parseInt(dateParts[1]) + " 月 " + parseInt(dateParts[2]) + " 日";

      s += '<li class="' + itemClass + '">';
      s += '  <span class="todo-icon">' + todo.icon + '</span>';
      s += '  <div class="todo-content">';
      s += '    <p class="todo-name">' + todo.name + '</p>';
      s += '    <p class="todo-date">' + dateStr + '</p>';
      s += '  </div>';
      s += '  <span class="' + countdownClass + '">' + label + '</span>';
      s += '</li>';
    });
    s += "</ul>";
    return s;
  }

  if (todos.length === 0) {
    html = '<p class="todo-empty">暂无待办事项</p>';
  } else {
    html += renderSection("进行中", upcoming);
    html += renderSection("已过期", expired);
  }

  container.innerHTML = html;
})();
</script>

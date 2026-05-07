---
layout: single
title: "时间轴"
permalink: /timeline/
---

<style>
  .timeline-wrapper {
    max-width: 700px;
    margin: 0 auto;
  }

  .timeline-desc {
    color: #6a737d;
    margin-bottom: 24px;
    font-size: 0.9em;
  }

  /* 日期分组 */
  .timeline-date-group {
    margin-bottom: 28px;
  }

  .timeline-date-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 4px;
    margin-bottom: 10px;
    cursor: pointer;
    user-select: none;
  }

  .timeline-date-header:hover .timeline-date-text {
    color: #24292e;
  }

  .timeline-date-text {
    font-size: 0.88em;
    font-weight: 600;
    color: #586069;
    transition: color 0.15s;
    letter-spacing: 0.01em;
  }

  .timeline-date-count {
    font-size: 0.75em;
    color: #959da5;
    background: #f6f8fa;
    padding: 2px 8px;
    border-radius: 10px;
  }

  .timeline-date-toggle {
    font-size: 0.7em;
    color: #959da5;
    transition: transform 0.2s ease;
    display: inline-block;
  }

  .timeline-date-group.collapsed .timeline-date-toggle {
    transform: rotate(-90deg);
  }

  /* 时间轴容器 */
  .timeline-list {
    position: relative;
    padding-left: 28px;
    margin-left: 6px;
    border-left: 2px solid #e1e4e8;
  }

  .timeline-date-group.collapsed .timeline-list {
    display: none;
  }

  /* 单条提交 */
  .timeline-item {
    position: relative;
    padding: 0 0 16px 24px;
  }

  .timeline-item:last-child {
    padding-bottom: 0;
  }

  .timeline-dot {
    position: absolute;
    left: -33px;
    top: 6px;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #fff;
    border: 2px solid #d1d5da;
    z-index: 1;
    transition: border-color 0.15s, background 0.15s;
  }

  .timeline-item:first-child .timeline-dot {
    border-color: #0366d6;
    background: #0366d6;
  }

  .timeline-item:hover .timeline-dot {
    border-color: #0366d6;
  }

  .timeline-card {
    display: flex;
    align-items: flex-start;
    padding: 10px 14px;
    border-radius: 8px;
    background: #ffffff;
    border: 1px solid #e1e4e8;
    transition: box-shadow 0.15s, border-color 0.15s;
    text-decoration: none;
    color: inherit;
    gap: 12px;
  }

  .timeline-card:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.07);
    border-color: #d1d5da;
  }

  .timeline-sha {
    flex-shrink: 0;
    font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
    font-size: 0.78em;
    color: #0366d6;
    background: #f1f8ff;
    padding: 2px 6px;
    border-radius: 4px;
    min-width: 52px;
    text-align: center;
  }

  .timeline-body {
    flex: 1;
    min-width: 0;
  }

  .timeline-msg {
    font-size: 0.92em;
    color: #24292e;
    line-height: 1.45;
    margin: 0 0 2px 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .timeline-meta {
    font-size: 0.78em;
    color: #959da5;
  }

  /* 加载更多 */
  .timeline-more {
    text-align: center;
    margin-top: 24px;
  }

  .timeline-more-btn {
    display: inline-block;
    padding: 8px 24px;
    font-size: 0.88em;
    color: #0366d6;
    background: #f1f8ff;
    border: 1px solid #c8e1ff;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.15s;
  }

  .timeline-more-btn:hover {
    background: #ddeeff;
  }

  .timeline-more-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .timeline-more-info {
    font-size: 0.78em;
    color: #959da5;
    margin-top: 6px;
  }

  /* 空状态 */
  .timeline-empty {
    text-align: center;
    padding: 50px 20px;
    color: #959da5;
  }

  .timeline-empty-icon {
    font-size: 2.5em;
    margin-bottom: 12px;
  }

  .timeline-empty-text {
    font-size: 0.95em;
    line-height: 1.6;
  }

  .timeline-empty-hint {
    font-size: 0.82em;
    color: #b1bac4;
    margin-top: 8px;
  }

  /* 响应式 */
  @media (max-width: 480px) {
    .timeline-list {
      padding-left: 22px;
      margin-left: 4px;
    }
    .timeline-item {
      padding-left: 18px;
    }
    .timeline-dot {
      left: -29px;
    }
    .timeline-sha {
      display: none;
    }
  }
</style>

<div class="timeline-wrapper">
  <p class="timeline-desc">
    最近提交记录 &middot; 数据由 GitHub Actions 自动更新
  </p>

  <div id="timeline-container"></div>
</div>

<script>
(function () {
  // ==================== 数据注入 ====================
  var commits = {{ site.data.commits | jsonify }};

  // ==================== 配置 ====================
  var INITIAL_SHOW = 7;     // 首次显示条数
  var LOAD_MORE = 10;       // 每次加载更多条数
  var WEEKDAYS = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];

  // ==================== 状态 ====================
  var shownCount = 0;

  // ==================== 工具函数 ====================
  function formatDateKey(dateStr) {
    return dateStr.substring(0, 10); // "2026-05-07"
  }

  function formatDateLabel(dateKey) {
    var parts = dateKey.split("-");
    return parseInt(parts[1]) + " 月 " + parseInt(parts[2]) + " 日";
  }

  function getWeekday(dateStr) {
    var d = new Date(dateStr);
    return WEEKDAYS[d.getDay()];
  }

  function formatTime(dateStr) {
    var d = new Date(dateStr);
    var h = String(d.getHours()).padStart(2, "0");
    var m = String(d.getMinutes()).padStart(2, "0");
    return h + ":" + m;
  }

  function isToday(dateKey) {
    var now = new Date();
    var y = String(now.getFullYear());
    var mo = String(now.getMonth() + 1).padStart(2, "0");
    var d = String(now.getDate()).padStart(2, "0");
    return dateKey === y + "-" + mo + "-" + d;
  }

  // ==================== 按日期分组 ====================
  function groupByDate(items) {
    var groups = [];
    var map = {};
    items.forEach(function (c) {
      var dk = formatDateKey(c.date);
      if (!map[dk]) {
        var g = { dateKey: dk, label: formatDateLabel(dk), weekday: getWeekday(c.date), items: [] };
        map[dk] = g;
        groups.push(g);
      }
      map[dk].items.push(c);
    });
    return groups;
  }

  // ==================== 渲染 ====================
  function renderTimeline(append) {
    var container = document.getElementById("timeline-container");
    var sliceEnd = Math.min(shownCount, commits.length);
    var visible = commits.slice(0, sliceEnd);

    if (visible.length === 0) {
      container.innerHTML =
        '<div class="timeline-empty">' +
        '  <div class="timeline-empty-icon">&#128197;</div>' +
        '  <div class="timeline-empty-text">暂无提交记录</div>' +
        '  <div class="timeline-empty-hint">推送代码后 GitHub Actions 会自动更新数据</div>' +
        '</div>';
      return;
    }

    var groups = groupByDate(visible);
    var html = "";

    groups.forEach(function (g, gi) {
      var todayTag = isToday(g.dateKey) ? "（今天）" : "";
      var collapsed = (gi > 0) ? " collapsed" : ""; // 第一组默认展开，其余折叠

      html += '<div class="timeline-date-group' + collapsed + '">';
      html += '  <div class="timeline-date-header" data-group="' + gi + '">';
      html += '    <span class="timeline-date-text">' + g.label + " " + g.weekday + todayTag + '</span>';
      html += '    <span>';
      html += '      <span class="timeline-date-count">' + g.items.length + ' 条</span>';
      html += '      <span class="timeline-date-toggle">&#9660;</span>';
      html += '    </span>';
      html += '  </div>';
      html += '  <div class="timeline-list">';

      g.items.forEach(function (c) {
        html += '<div class="timeline-item">';
        html += '  <span class="timeline-dot"></span>';
        html += '  <a class="timeline-card" href="' + c.url + '" target="_blank" rel="noopener">';
        html += '    <span class="timeline-sha">' + c.sha + '</span>';
        html += '    <div class="timeline-body">';
        html += '      <p class="timeline-msg">' + escapeHtml(c.message) + '</p>';
        html += '      <span class="timeline-meta">' + formatTime(c.date) + '</span>';
        html += '    </div>';
        html += '  </a>';
        html += '</div>';
      });

      html += '  </div>';
      html += '</div>';
    });

    // 加载更多按钮
    if (shownCount < commits.length) {
      var remaining = commits.length - shownCount;
      var loadCount = Math.min(LOAD_MORE, remaining);
      html += '<div class="timeline-more">';
      html += '  <button class="timeline-more-btn" id="btn-load-more">加载更多</button>';
      html += '  <div class="timeline-more-info">还有 ' + remaining + ' 条记录</div>';
      html += '</div>';
    } else if (commits.length > INITIAL_SHOW) {
      html += '<div class="timeline-more">';
      html += '  <div class="timeline-more-info">已显示全部 ' + commits.length + ' 条记录</div>';
      html += '</div>';
    }

    container.innerHTML = html;

    // 绑定日期组折叠
    container.querySelectorAll(".timeline-date-header").forEach(function (header) {
      header.addEventListener("click", function () {
        this.parentElement.classList.toggle("collapsed");
      });
    });

    // 绑定加载更多
    var loadBtn = document.getElementById("btn-load-more");
    if (loadBtn) {
      loadBtn.addEventListener("click", function () {
        shownCount += LOAD_MORE;
        renderTimeline(true);
      });
    }
  }

  function escapeHtml(str) {
    var div = document.createElement("div");
    div.appendChild(document.createTextNode(str));
    return div.innerHTML;
  }

  // ==================== 初始化 ====================
  shownCount = INITIAL_SHOW;
  renderTimeline(false);
})();
</script>

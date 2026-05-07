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
    margin-bottom: 28px;
    font-size: 0.9em;
  }

  /* 年份标题 */
  .timeline-year {
    font-size: 1.3em;
    font-weight: 700;
    color: #24292e;
    margin: 32px 0 6px 0;
    padding-bottom: 6px;
    border-bottom: 2px solid #e1e4e8;
  }

  .timeline-year:first-child {
    margin-top: 0;
  }

  /* 月份分组 */
  .timeline-month-group {
    margin-bottom: 24px;
  }

  .timeline-month-header {
    font-size: 0.88em;
    font-weight: 600;
    color: #586069;
    padding: 0 4px;
    margin-bottom: 12px;
    cursor: pointer;
    user-select: none;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .timeline-month-header:hover {
    color: #24292e;
  }

  .timeline-month-toggle {
    font-size: 0.7em;
    color: #959da5;
    transition: transform 0.2s ease;
    display: inline-block;
  }

  .timeline-month-group.collapsed .timeline-month-toggle {
    transform: rotate(-90deg);
  }

  /* 时间轴容器 */
  .timeline-list {
    position: relative;
    padding-left: 24px;
    margin-left: 6px;
    border-left: 2px solid #e1e4e8;
  }

  .timeline-month-group.collapsed .timeline-list {
    display: none;
  }

  /* 单条文章 */
  .timeline-item {
    position: relative;
    padding: 0 0 20px 20px;
  }

  .timeline-item:last-child {
    padding-bottom: 0;
  }

  .timeline-dot {
    position: absolute;
    left: -29px;
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
    display: block;
    padding: 12px 16px;
    border-radius: 8px;
    background: #ffffff;
    border: 1px solid #e1e4e8;
    transition: box-shadow 0.15s, border-color 0.15s;
    text-decoration: none;
    color: inherit;
  }

  .timeline-card:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.07);
    border-color: #d1d5da;
  }

  .timeline-card-title {
    font-size: 0.95em;
    font-weight: 500;
    color: #0366d6;
    margin: 0 0 6px 0;
    line-height: 1.45;
  }

  .timeline-card:hover .timeline-card-title {
    color: #005cc5;
  }

  .timeline-card-excerpt {
    font-size: 0.85em;
    color: #6a737d;
    margin: 0 0 8px 0;
    line-height: 1.5;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }

  .timeline-card-meta {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  }

  .timeline-card-date {
    font-size: 0.78em;
    color: #959da5;
  }

  .timeline-card-tag {
    font-size: 0.72em;
    color: #0366d6;
    background: #f1f8ff;
    padding: 1px 8px;
    border-radius: 10px;
    text-decoration: none;
    transition: background 0.15s;
  }

  .timeline-card-tag:hover {
    background: #ddeeff;
  }

  .timeline-card-cat {
    font-size: 0.72em;
    color: #57606a;
    background: #f6f8fa;
    padding: 1px 8px;
    border-radius: 10px;
  }

  /* 统计信息 */
  .timeline-stats {
    text-align: center;
    color: #959da5;
    font-size: 0.82em;
    margin-top: 24px;
    padding-top: 16px;
    border-top: 1px solid #e1e4e8;
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

  /* 响应式 */
  @media (max-width: 480px) {
    .timeline-list {
      padding-left: 20px;
      margin-left: 4px;
    }
    .timeline-item {
      padding-left: 16px;
    }
    .timeline-dot {
      left: -25px;
    }
    .timeline-card-excerpt {
      -webkit-line-clamp: 1;
    }
  }
</style>

<div class="timeline-wrapper">
  <p class="timeline-desc">
    所有博客文章按时间排列，点击可跳转阅读
  </p>

  <div id="timeline-container"></div>
</div>

<script>
(function () {
  // ==================== 数据注入（Jekyll 构建时生成） ====================
  var posts = [
    {% for post in site.posts %}
      {
        title: {{ post.title | jsonify }},
        url: {{ post.url | jsonify }},
        date: "{{ post.date | date: '%Y-%m-%d' }}",
        excerpt: {{ post.excerpt | strip_html | strip | jsonify }},
        categories: [
          {% for cat in post.categories %}
            {{ cat | jsonify }}{% unless forloop.last %},{% endunless %}
          {% endfor %}
        ],
        tags: [
          {% for tag in post.tags %}
            {{ tag | jsonify }}{% unless forloop.last %},{% endunless %}
          {% endfor %}
        ]
      }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  // ==================== 配置 ====================
  var MONTHS = ["1月", "2月", "3月", "4月", "5月", "6月", "7月", "8月", "9月", "10月", "11月", "12月"];
  var WEEKDAYS = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];

  // ==================== 工具函数 ====================
  function escapeHtml(str) {
    var div = document.createElement("div");
    div.appendChild(document.createTextNode(str));
    return div.innerHTML;
  }

  function getWeekday(dateStr) {
    var d = new Date(dateStr + "T00:00:00");
    return WEEKDAYS[d.getDay()];
  }

  // ==================== 按年-月分组 ====================
  function groupByMonth(items) {
    var yearMap = {};
    items.forEach(function (p) {
      var parts = p.date.split("-");
      var year = parts[0];
      var month = parseInt(parts[1]);
      var key = year + "-" + month;
      if (!yearMap[year]) yearMap[year] = {};
      if (!yearMap[year][key]) {
        yearMap[year][key] = { year: year, month: month, label: MONTHS[month - 1], items: [] };
      }
      yearMap[year][key].items.push(p);
    });
    // 转为数组，按年倒序，月倒序
    var years = Object.keys(yearMap).sort(function (a, b) { return b - a; });
    var result = [];
    years.forEach(function (y) {
      var months = Object.values(yearMap[y]).sort(function (a, b) { return b.month - a.month; });
      result.push({ year: y, months: months });
    });
    return result;
  }

  // ==================== 渲染 ====================
  function render() {
    var container = document.getElementById("timeline-container");

    if (posts.length === 0) {
      container.innerHTML =
        '<div class="timeline-empty">' +
        '  <div class="timeline-empty-icon">&#128221;</div>' +
        '  <div class="timeline-empty-text">还没有文章</div>' +
        '</div>';
      return;
    }

    var groups = groupByMonth(posts);
    var html = "";

    groups.forEach(function (yg, yi) {
      // 年份标题
      html += '<div class="timeline-year">' + yg.year + '</div>';

      yg.months.forEach(function (mg, mi) {
        var collapsed = (yi === 0 && mi === 0) ? "" : " collapsed";

        html += '<div class="timeline-month-group' + collapsed + '">';
        html += '  <div class="timeline-month-header" data-group="' + yi + "-" + mi + '">';
        html += '    <span>' + mg.label + ' (' + mg.items.length + ' 篇)</span>';
        html += '    <span class="timeline-month-toggle">&#9660;</span>';
        html += '  </div>';
        html += '  <div class="timeline-list">';

        mg.items.forEach(function (p) {
          var weekday = getWeekday(p.date);
          var dateParts = p.date.split("-");
          var dateLabel = parseInt(dateParts[1]) + " 月 " + parseInt(dateParts[2]) + " 日 " + weekday;

          // 标签
          var metaHtml = '<span class="timeline-card-date">' + dateLabel + '</span>';
          if (p.categories.length > 0) {
            metaHtml += '<span class="timeline-card-cat">' + escapeHtml(p.categories[0]) + '</span>';
          }
          p.tags.forEach(function (t) {
            metaHtml += '<span class="timeline-card-tag">' + escapeHtml(t) + '</span>';
          });

          html += '<div class="timeline-item">';
          html += '  <span class="timeline-dot"></span>';
          html += '  <a class="timeline-card" href="' + p.url + '">';
          html += '    <p class="timeline-card-title">' + escapeHtml(p.title) + '</p>';
          if (p.excerpt) {
            html += '    <p class="timeline-card-excerpt">' + escapeHtml(p.excerpt) + '</p>';
          }
          html += '    <div class="timeline-card-meta">' + metaHtml + '</div>';
          html += '  </a>';
          html += '</div>';
        });

        html += '  </div>';
        html += '</div>';
      });
    });

    // 统计
    html += '<div class="timeline-stats">共 ' + posts.length + ' 篇文章</div>';

    container.innerHTML = html;

    // 绑定月份折叠
    container.querySelectorAll(".timeline-month-header").forEach(function (header) {
      header.addEventListener("click", function () {
        this.parentElement.classList.toggle("collapsed");
      });
    });
  }

  // ==================== 初始化 ====================
  render();
})();
</script>

---
title: "上传博客文章指北"
date: 2026-05-05
categories: blog
tags: 博客框架
---

PS：干货满满♥

## 一、创建文章文件

在 `_posts/` 目录下新建 `.md` 文件，文件名格式：

```
YYYY-MM-DD-english-slug.md
```

**命名规则**：
- 日期部分 `YYYY-MM-DD` 必须与 front matter 中的 `date` 一致
- slug 部分用**英文 + 连字符**，不要用中文（否则 URL 会被百分号编码成乱码）
- 示例：`2026-05-21-mysql-connection.md`

## 二、文章 Front Matter

每篇文章顶部必须包含 `---` 包裹的元数据：

```yaml
---
title: "文章中文标题"
date: 2026-05-21
categories: tech           # 必填：tech / blog / life
tags: JavaEE               # 必填：技术关键词
mathjax: true              # 可选：有数学公式时加上这行
description: "文章摘要"     # 推荐：用于 SEO 搜索结果展示
---
```

**分类对照表**（URL 用英文，页面自动显示中文）：

| 填写值 | 显示 | 用途 |
|--------|------|------|
| `tech` | 技术 | JavaEE、Python、算法等技术学习笔记 |
| `blog` | 博客 | 博客搭建、配置、优化相关 |
| `life` | 生活 | 规划、健身等非技术内容 |

## 三、插入图片

图片放在 `assets/images/` 目录下，引用方式：

```markdown
![]({{ site.baseurl }}/assets/images/图片名.png)
```

**注意**：GitHub Pages 无法使用本地绝对路径，必须用 `site.baseurl` 前缀。

## 四、数学公式

在 front matter 中加入 `mathjax: true`，然后用 LaTeX 语法写公式：

```markdown
行内公式：$E = mc^2$

块级公式：
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

不加 `mathjax: true` 的页面不会加载 MathJax，不影响加载速度。

## 五、更新待办事项

编辑 `_data/todos.yml`，每条格式：

```yaml
- name: "事项名称"
  date: 2026-06-01
  icon: "📚"
```

- `name`：事项描述
- `date`：截止日期（过期会自动划掉并置灰）
- `icon`：一个 emoji 作为图标

待办页面 `/todos/` 会自动计算剩余天数并分"进行中 / 已过期"两组显示。

## 六、时间轴

时间是自动生成的 —— 只要文章在 `_posts/` 下且有正确的 `date`，就会出现在 `/timeline/` 页面，按年月分组，无需手动维护。

## 七、检查清单

发文章之前确认：

- [ ] 文件名使用英文 slug
- [ ] `date` 与文件名日期一致
- [ ] `categories` 填了 `tech` / `blog` / `life`
- [ ] `tags` 填了至少一个标签
- [ ] 有公式时加了 `mathjax: true`
- [ ] 图片路径用了 `site.baseurl`
- [ ] 本地 `bundle exec jekyll serve` 预览通过

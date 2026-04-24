# Blog Navigation & Homepage Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add category nav, tag cloud, about page, footer with social links, category-aware prev/next, and update the homepage to show the latest post.

**Architecture:** `_data/nav.yml` drives the nav bar rendered in `default.html`. Each category gets a `.md` file using `_layouts/category.html`. The tag cloud is a Liquid include linked to `/archive/` anchors. Prev/next logic in `navlinks.html` filters by `site.categories[cat]` when a post has a category, falling back to global `page.previous`/`page.next` otherwise.

**Tech Stack:** Jekyll 4.4, Liquid templating, HTML/CSS. No new gems required.

---

### Task 1: Nav data file + nav bar in default layout

**Files:**
- Create: `_data/nav.yml`
- Modify: `_layouts/default.html`
- Modify: `css/override.css`

- [ ] **Step 1: Create `_data/nav.yml`**

```yaml
- label: Projects
  url: /projects/
  category: projects
- label: Growing Things
  url: /growing-things/
  category: growing-things
- label: Biohacking
  url: /biohacking/
  category: biohacking
- label: Climbing
  url: /climbing/
  category: climbing
```

- [ ] **Step 2: Update `_layouts/default.html` to include nav bar**

Replace the `<header class="inner">` block with:

```html
  <div id="header_wrap" class="outer">
    <header class="inner">
      <h1 id="project_title"><a href="{{ '/' | relative_url }}">{{ site.title }}</a></h1>
      <h2 id="project_tagline">{{ site.description }}</h2>
      <nav class="site-nav">
        {% for item in site.data.nav %}
          {% assign is_active = false %}
          {% if page.url == item.url %}
            {% assign is_active = true %}
          {% elsif page.categories contains item.category %}
            {% assign is_active = true %}
          {% endif %}
          <a href="{{ item.url }}"{% if is_active %} class="active"{% endif %}>{{ item.label }}</a>
        {% endfor %}
        <a href="/about/"{% if page.url == '/about/' %} class="active"{% endif %}>About</a>
      </nav>
    </header>
  </div>
```

- [ ] **Step 3: Add nav CSS to `css/override.css`**

Append to the end of the file:

```css
/* Site nav */
nav.site-nav {
  margin-top: 14px;
}

nav.site-nav a {
  color: rgba(255, 255, 255, 0.7);
  text-decoration: none;
  margin-right: 18px;
  font-size: 0.85em;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

nav.site-nav a:hover,
nav.site-nav a.active {
  color: #fff;
  text-decoration: underline;
}
```

- [ ] **Step 4: Verify nav renders**

With `bundle exec jekyll serve` running, run:
```bash
curl -s http://127.0.0.1:4000/ | grep -A 20 'site-nav'
```
Expected: `<nav class="site-nav">` with links to `/projects/`, `/growing-things/`, `/biohacking/`, `/climbing/`, `/about/`.

- [ ] **Step 5: Commit**

```bash
git add _data/nav.yml _layouts/default.html css/override.css
git commit -m "feat: add category nav bar"
```

---

### Task 2: Footer with social links and copyright

**Files:**
- Modify: `_includes/footer.html`
- Modify: `css/override.css`

- [ ] **Step 1: Update `_includes/footer.html`**

```html
<div class="footer-social">
  {% if site.github_username %}
  <a href="https://github.com/{{ site.github_username }}">GitHub</a>
  {% endif %}
  {% if site.instagram_username %}
  <a href="https://instagram.com/{{ site.instagram_username }}">Instagram</a>
  {% endif %}
  {% if site.linkedin_username %}
  <a href="https://linkedin.com/in/{{ site.linkedin_username }}">LinkedIn</a>
  {% endif %}
</div>
<p class="footer-copyright">&copy; {{ site.time | date: '%Y' }} {{ site.author }}</p>
```

> **Note:** `_config.yml` currently has `linkedin_username: "linkedinuser"` as a placeholder. Update it with your real LinkedIn username (or comment it out) before deploying.

- [ ] **Step 2: Add footer CSS to `css/override.css`**

Append:

```css
/* Footer */
#footer_wrap {
  margin-top: 40px;
}

.footer-social a {
  color: rgba(255, 255, 255, 0.7);
  text-decoration: none;
  margin-right: 14px;
  font-size: 0.9em;
}

.footer-social a:hover {
  color: #fff;
  text-decoration: underline;
}

.footer-copyright {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.8em;
  margin-top: 8px;
}
```

- [ ] **Step 3: Verify footer renders**

```bash
curl -s http://127.0.0.1:4000/ | grep -A 10 'footer-social'
```
Expected: links to GitHub and Instagram (and LinkedIn if username is set).

- [ ] **Step 4: Commit**

```bash
git add _includes/footer.html css/override.css
git commit -m "feat: add footer with social links and copyright"
```

---

### Task 3: Category layout + category pages + post front matter

**Files:**
- Create: `_layouts/category.html`
- Create: `projects.md`, `growing-things.md`, `biohacking.md`, `climbing.md`
- Modify: `_posts/2025-12-23-two-years-with-gardyn-3.0.md`

- [ ] **Step 1: Create `_layouts/category.html`**

```html
---
layout: default
---
<h2>{{ page.title }}</h2>
<ul class="post-list">
  {% assign cat_posts = site.categories[page.category] %}
  {% if cat_posts %}
    {% for post in cat_posts %}
    <li>
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      &mdash;
      <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
    </li>
    {% endfor %}
  {% else %}
    <li>No posts yet.</li>
  {% endif %}
</ul>
```

- [ ] **Step 2: Create `projects.md`**

```markdown
---
layout: category
title: Projects
category: projects
---
```

- [ ] **Step 3: Create `growing-things.md`**

```markdown
---
layout: category
title: Growing Things
category: growing-things
---
```

- [ ] **Step 4: Create `biohacking.md`**

```markdown
---
layout: category
title: Biohacking
category: biohacking
---
```

- [ ] **Step 5: Create `climbing.md`**

```markdown
---
layout: category
title: Climbing
category: climbing
---
```

- [ ] **Step 6: Add front matter to existing post**

Add a front matter block to the top of `_posts/2025-12-23-two-years-with-gardyn-3.0.md`:

```yaml
---
layout: post
title: "Two Years with Gardyn 3.0"
date: 2025-12-23
categories: growing-things
tags:
  - gardyn
  - hydroponics
  - plant care
author: Mike Faith
---
```

The rest of the file is unchanged.

- [ ] **Step 7: Verify category pages and post**

```bash
curl -s http://127.0.0.1:4000/growing-things/ | grep -A 5 'post-list'
```
Expected: list item linking to the Gardyn post.

```bash
curl -s http://127.0.0.1:4000/growing-things/ | grep 'Gardyn'
```
Expected: post title appears in the list.

- [ ] **Step 8: Add post-list CSS to `css/override.css`**

Append:

```css
/* Category page post list */
ul.post-list {
  list-style: none;
  padding: 0;
}

ul.post-list li {
  margin: 8px 0;
}

ul.post-list .post-date {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.85em;
}
```

- [ ] **Step 9: Commit**

```bash
git add _layouts/category.html projects.md growing-things.md biohacking.md climbing.md _posts/2025-12-23-two-years-with-gardyn-3.0.md css/override.css
git commit -m "feat: add category layout, category pages, and post front matter"
```

---

### Task 4: About page + updated homepage

**Files:**
- Create: `about.md`
- Modify: `index.md`

- [ ] **Step 1: Create `about.md`**

```markdown
---
layout: default
title: About
---

Hi, I'm Mike.

I have a lot of random interests.

Here you'll find me prattling on about them.
```

- [ ] **Step 2: Update `index.md` to show latest post**

```markdown
---
layout: default
title: Home
---

{% assign latest = site.posts.first %}
{% if latest %}
<div class="latest-post">
  <h2><a href="{{ latest.url | relative_url }}">{{ latest.title | escape }}</a></h2>
  <p class="post-meta">{{ latest.date | date: "%B %-d, %Y" }}</p>
  {{ latest.content }}
</div>
{% endif %}

{% include tag-cloud.html %}
```

> `tag-cloud.html` is created in Task 5. Jekyll will warn about a missing include until then — that's fine.

- [ ] **Step 3: Verify about page**

```bash
curl -s http://127.0.0.1:4000/about/ | grep "Hi, I'm Mike"
```
Expected: the text appears in rendered HTML.

- [ ] **Step 4: Verify homepage shows latest post title**

```bash
curl -s http://127.0.0.1:4000/ | grep 'Two Years'
```
Expected: post title appears as a heading link.

- [ ] **Step 5: Commit**

```bash
git add about.md index.md
git commit -m "feat: add about page and update homepage to show latest post"
```

---

### Task 5: Tag cloud + archive anchors

**Files:**
- Create: `_includes/tag-cloud.html`
- Modify: `archive.md`
- Modify: `css/override.css`

- [ ] **Step 1: Create `_includes/tag-cloud.html`**

```html
{% if site.tags.size > 0 %}
<div class="tag-cloud">
  <h3>Tags</h3>
  {% assign max_count = 0 %}
  {% for tag in site.tags %}
    {% if tag[1].size > max_count %}
      {% assign max_count = tag[1].size %}
    {% endif %}
  {% endfor %}
  {% for tag in site.tags %}
    {% assign count = tag[1].size %}
    {% assign ratio = count | times: 100 | divided_by: max_count %}
    {% if ratio >= 67 %}
      {% assign size_class = "tag-large" %}
    {% elsif ratio >= 33 %}
      {% assign size_class = "tag-medium" %}
    {% else %}
      {% assign size_class = "tag-small" %}
    {% endif %}
    <a href="/archive/#{{ tag[0] | slugify }}" class="tag {{ size_class }}">{{ tag[0] }}</a>
  {% endfor %}
</div>
{% endif %}
```

- [ ] **Step 2: Add tag anchors to `archive.md`**

Replace the existing `<h3>{{ tag[0] }}</h3>` line with:

```html
<h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
```

Full updated `archive.md`:

```markdown
---
layout: default
title: Blog Archive
---

{% for tag in site.tags %}
  <h3 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
```

- [ ] **Step 3: Add tag cloud CSS to `css/override.css`**

Append:

```css
/* Tag cloud */
.tag-cloud {
  margin-top: 32px;
  padding-top: 20px;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.tag-cloud h3 {
  margin-bottom: 12px;
}

.tag-cloud .tag {
  display: inline-block;
  margin: 4px 6px;
  color: rgba(255, 255, 255, 0.75);
  text-decoration: none;
}

.tag-cloud .tag:hover {
  color: #fff;
  text-decoration: underline;
}

.tag-cloud .tag-large  { font-size: 1.25em; }
.tag-cloud .tag-medium { font-size: 1.0em; }
.tag-cloud .tag-small  { font-size: 0.85em; }
```

- [ ] **Step 4: Verify tag cloud on homepage**

```bash
curl -s http://127.0.0.1:4000/ | grep 'tag-cloud'
```
Expected: `<div class="tag-cloud">` with tag links.

- [ ] **Step 5: Verify archive anchor**

```bash
curl -s http://127.0.0.1:4000/archive/ | grep 'id='
```
Expected: `<h3 id="gardyn">` (or whatever slugified tag name).

- [ ] **Step 6: Commit**

```bash
git add _includes/tag-cloud.html archive.md css/override.css
git commit -m "feat: add tag cloud and archive anchors"
```

---

### Task 6: Category-aware prev/next navigation

**Files:**
- Modify: `_includes/navlinks.html`

- [ ] **Step 1: Replace `_includes/navlinks.html`**

```html
<hr>
<div class="post_navi">
  {%- assign cat = page.categories.first -%}
  {%- assign prev_post = nil -%}
  {%- assign next_post = nil -%}
  {%- if cat -%}
    {%- assign cat_posts = site.categories[cat] -%}
    {%- assign post_found = false -%}
    {%- assign post_idx = 0 -%}
    {%- for post in cat_posts -%}
      {%- if post.url == page.url -%}
        {%- assign post_idx = forloop.index0 -%}
        {%- assign post_found = true -%}
      {%- endif -%}
    {%- endfor -%}
    {%- if post_found -%}
      {%- if post_idx > 0 -%}
        {%- assign next_idx = post_idx | minus: 1 -%}
        {%- assign next_post = cat_posts[next_idx] -%}
      {%- endif -%}
      {%- assign last_idx = cat_posts.size | minus: 1 -%}
      {%- if post_idx < last_idx -%}
        {%- assign prev_idx = post_idx | plus: 1 -%}
        {%- assign prev_post = cat_posts[prev_idx] -%}
      {%- endif -%}
    {%- endif -%}
  {%- else -%}
    {%- assign prev_post = page.previous -%}
    {%- assign next_post = page.next -%}
  {%- endif -%}

  {%- if prev_post -%}
  <a class="post_navi-item nav_prev" href="{{ prev_post.url }}" title="{{ prev_post.title }}">
    <div class="post_navi-arrow">&lt;</div>
    <div class="post_navi-label">Previous Post</div>
    <div><span>{{ prev_post.title }}</span></div>
  </a>
  {%- endif -%}
  {%- if next_post -%}
  <a class="post_navi-item nav_next" href="{{ next_post.url }}" title="{{ next_post.title }}">
    <div class="post_navi-arrow">&gt;</div>
    <div class="post_navi-label">Next Post</div>
    <div><span>{{ next_post.title }}</span></div>
  </a>
  {%- endif -%}
</div>
```

**Logic notes:**
- `site.categories[cat]` is sorted newest-first by Jekyll. Index 0 = newest.
- `next_post` = lower index = more recent post.
- `prev_post` = higher index = older post.
- `post_found` flag avoids falsy-zero bug: `{% if post_idx %}` would be false when index is 0.
- Falls back to global `page.previous`/`page.next` when post has no category.

- [ ] **Step 2: Verify nav renders on the Gardyn post**

```bash
curl -s http://127.0.0.1:4000/2025/12/23/two-years-with-gardyn-3.0/ | grep 'post_navi'
```
Expected: `<div class="post_navi">` is present. Since this is the only post in the category, neither prev nor next link will render — that's correct behaviour.

- [ ] **Step 3: Commit**

```bash
git add _includes/navlinks.html
git commit -m "feat: category-aware prev/next post navigation"
```

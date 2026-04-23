# Blog Navigation & Homepage Design

**Date:** 2026-04-23
**Status:** Approved

## Overview

Add category-based navigation, a tag cloud, an about page, a footer, and update the homepage to show the latest post. Prev/next post navigation becomes category-aware.

## File Changes

### New files
- `_data/nav.yml` — category nav items (label + url)
- `_layouts/category.html` — lists posts in a given category
- `_includes/tag-cloud.html` — tag cloud rendered on the homepage
- `code.md` — Code category page
- `growing-things.md` — Growing Things category page
- `biohacking.md` — Biohacking category page
- `climbing.md` — Climbing category page
- `about.md` — About page (contains current index content)

### Modified files
- `_layouts/default.html` — adds nav bar and footer includes
- `_layouts/post.html` — category-aware prev/next
- `_includes/footer.html` — social links + copyright
- `_includes/navlinks.html` — category-aware prev/next logic
- `index.md` — shows latest post + tag cloud

## Navigation

`_data/nav.yml` defines nav items as a list of `label` + `url` pairs. The nav bar renders in the slate `#header_wrap`, below title and tagline. A fixed "About" link is appended after the data-driven category links. The currently active page link is highlighted via CSS.

Initial categories:
```yaml
- label: Code
  url: /code/
- label: Growing Things
  url: /growing-things/
- label: Biohacking
  url: /biohacking/
- label: Climbing
  url: /climbing/
```

Adding a new category requires: one new entry in `nav.yml` + one new category `.md` file.

## Category Pages

Each category page (e.g. `code.md`) uses `layout: category` and declares its category slug in front matter:

```yaml
---
layout: category
title: Code
category: code
---
```

The `category.html` layout loops through `site.categories[page.category]` and renders a date + title link list, newest first. No excerpts.

Posts are associated with a category via front matter: `categories: code`. The value must match the slug used in `nav.yml` and the category page's `category:` field exactly. The `archive.md` tag anchors must use the same slugify filter as the tag cloud links.

## Homepage

`index.md` uses `layout: default`. Content:
1. Renders the most recent post (`site.posts.first`) inline — title (as heading), date, and full content — without share buttons or prev/next nav.
2. Below the post, renders `{% include tag-cloud.html %}`.

## Tag Cloud

`tag-cloud.html` loops through `site.tags` and renders each tag as a link to `/archive/#{{ tag[0] | slugify }}`. Tags appear in 3 size tiers based on post count relative to the most-used tag:
- ≥ 67% of max count → large
- ≥ 33% of max count → medium
- < 33% → small

## About Page

`about.md` uses `layout: default`, `title: About`. Contains the text currently in `index.md`: "Hi, I'm Mike. I have a lot of random interests. Here you'll find me prattling on about them."

## Footer

Rendered by `_includes/footer.html`, included in `default.html`. Contains:
- Social icons for GitHub, Instagram, and LinkedIn, pulled from `site.github_username`, `site.instagram_username`, `site.linkedin_username` in `_config.yml`. Icons only render if the corresponding config value is set.
- Copyright line: `© {{ site.time | date: '%Y' }} {{ site.author }}`

## Prev/Next Post Navigation

`navlinks.html` implements category-aware prev/next:

1. Read `page.categories.first` as the current category.
2. If a category exists, filter to `site.categories[category]` (sorted newest-first by Jekyll). Loop to find the current post's index, then take `index - 1` (newer) and `index + 1` (older) as next/prev.
3. If no category is set, fall back to Jekyll's global `page.previous` / `page.next`.
4. If no previous or next post exists (boundary of the list), that button is omitted entirely.

The buttons remain at the bottom of each post via `post.html`.

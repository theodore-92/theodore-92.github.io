# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bundle install                  # install Ruby gems
bundle exec jekyll serve        # local preview at http://localhost:4000
bundle exec jekyll build        # build to _site/
```

## Architecture

Jekyll static blog deployed to GitHub Pages via `main` branch. Theme: [Hydeout](https://github.com/fongandrew/hydeout) loaded as a `remote_theme` — theme source is not in this repo.

**Posts** go in `_posts/YYYY-MM-DD-title.md` with this front matter:
```markdown
---
layout: post
title: "제목"
date: 2026-06-25
tags: [태그1, 태그2]
---
```

**Images** must use the `img.html` include, never raw `<img>` tags:
```liquid
{% include img.html name="2026-06-24-post/file.png" alt="설명" caption="캡션(선택)" %}
```
Image files live in `assets/images/posts/<YYYY-MM-DD-post>/`. The include resolves the base URL from `site.image_base_url` in `_config.yml` — changing that one value migrates all images site-wide.

**Comments** (`_includes/comments.html`) override Hydeout's Disqus default. Comments are disabled until `isso_url` is set in `_config.yml`.

**CSS** customization is done via SASS variables declared before `@import "hydeout"` in `assets/css/main.scss`. Key variables: `$sidebar-bg-color`, `$sidebar-fg-color`, `$link-color`, `$layout-reverse`.

**Sidebar layout direction** is controlled by `hydeout.layout_reverse` in `_config.yml` (and `$layout-reverse` in SCSS must match).

The `_site/` directory is the Jekyll build output — do not edit files there directly.
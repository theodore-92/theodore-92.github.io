# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow

**Never create commits.** Leave all changes uncommitted in the working tree — the user reviews and commits everything themselves.

**No headless-browser automation via debug-protocol flags.** Company policy blocks launching `msedge`/Chrome with `--remote-debugging-port` (or other CDP-automation flags), so Puppeteer/Playwright-style scripted interaction (simulating real keypresses, clicking, etc.) is not available in this environment. Plain `--headless --screenshot` / `--dump-dom` runs (no debug port) still work for visually spot-checking a change. Beyond that, present the diff and let the user test manually rather than trying more elaborate browser automation.

## Commands

```bash
bundle install                  # install Ruby gems
bundle exec jekyll build        # build to _site/
```

```bash
bundle exec jekyll serve --future  # local preview at http://localhost:4000
```

## Architecture

Jekyll static blog deployed to GitHub Pages via `main` branch. Theme: [Hydeout](https://github.com/fongandrew/hydeout) loaded as a `remote_theme` — theme source is not in this repo. Local files under `_layouts/` with the same name as a theme layout (e.g. `post.html`) override that theme layout; this is how post-specific additions (like the TOC nav below) get injected without vendoring the whole theme.

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

**Comments** (`_includes/comments.html`) override Hydeout's Disqus default with a custom Cloudflare Workers + D1 backend (nested replies, password-protected edit/delete, live or post-submit markdown preview). Enabled by setting `comments_api_url` in `_config.yml`; `turnstile_sitekey` optionally adds Turnstile verification once the API starts rate-limiting a client. The comment form's markdown textarea (`#cs-content`) is shared between the "실시간 미리보기"/"제출 후 렌더링" toggle — switching modes only shows/hides the `#cs-preview` panel, it does not swap textareas, so content always stays in sync. `renderMd()` runs content through `preprocessUnderscoreEmphasis()` before `marked.parse()`: CommonMark disallows underscore emphasis (`_x_`, `__x__`) mid-word, and Korean text has no spaces between words, so `_기울임_` next to Hangul silently failed to render — the preprocessor rewrites underscore emphasis to asterisk emphasis (which has no such restriction) while leaving code spans/fences untouched.

**Post TOC nav** (`_layouts/post.html` + `_includes/post-nav.html`) renders a fixed scroll-spy table of contents from a post's `h2`/`h3`/`h4` headings, generated client-side and highlighted via `IntersectionObserver`. Only applies to post pages (about/tags/index use different layouts, so they never include it) and only above the `$post-nav-breakpoint` (87.5em) in `assets/css/main.scss`, so it's desktop-only and never shows on mobile.

**CSS** customization is done via SASS variables declared before `@import "hydeout"` in `assets/css/main.scss`. Key variables: `$sidebar-bg-color`, `$sidebar-fg-color`, `$link-color`, `$layout-reverse`.

**Sidebar layout direction** is controlled by `hydeout.layout_reverse` in `_config.yml` (and `$layout-reverse` in SCSS must match).

The `_site/` directory is the Jekyll build output — do not edit files there directly.

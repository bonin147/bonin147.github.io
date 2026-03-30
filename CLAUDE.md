# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based bilingual (Korean/English) research blog hosted on GitHub Pages at bonin147.github.io. Covers AI/ML paper reviews and investment reports.

## Build & Serve

```bash
bundle install                    # Install dependencies (first time)
bundle exec jekyll serve          # Local dev server (http://localhost:4000)
bundle exec jekyll build          # Production build to _site/
```

Deployment is automatic via GitHub Pages on push to main branch.

## Architecture

- **Single gem dependency:** `github-pages` (includes Jekyll + all plugins)
- **Two layouts only:** `_layouts/default.html` (main shell, ~14K lines with inline JS) and `_layouts/post.html` (article wrapper)
- **No _includes:** All partials are inlined in `default.html`
- **No build toolchain:** Vanilla CSS (`assets/css/style.css`) and vanilla JS (embedded in `default.html`)

## Content Structure

### Post front matter template:
```yaml
---
layout: post
title: "[한국어 제목]"
title_en: "[English Title]"
date: YYYY-MM-DD
categories: [Category]
subcategory: "Paper Review|Stock Report|Index Report"
tags: [tag1, tag2]
description: "[설명]"
bilingual: true
paper_url: "[URL]"          # optional
arxiv_url: "[URL]"          # optional
paper_venue: "[Venue]"      # optional
paper_authors: "[Authors]"  # optional
---
```

### Bilingual content format in posts:
```html
<div class="lang-ko">한국어 내용</div>
<div class="lang-en">English content</div>
```

The language toggle in `default.html` shows/hides these divs and persists selection via localStorage.

### Three subcategories with index pages:
- `posts/paper-review/index.html`
- `posts/stock-report/index.html`
- `posts/index-report/index.html`

### Stock pages: Individual HTML files in `stock/` (aapl.html, msft.html, etc.)

## Key Features in default.html

The default layout contains significant inline JavaScript handling:
- **Dark mode:** Toggle with localStorage persistence
- **Table of Contents:** Auto-generated from headings, language-aware
- **Image lightbox:** Click-to-zoom on images
- **MathJax v3:** LaTeX math rendering (`$...$` inline, `$$...$$` display)
- **Responsive sidebar:** Hamburger menu on mobile

## Permalink Format

`/:year/:month/:day/:title/` — configured in `_config.yml`.

## References

PDF papers are stored in `references/` but this directory is gitignored. Images go in `assets/images/<post-slug>/`.

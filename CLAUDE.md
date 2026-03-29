# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Jekyll blog/portfolio site for Anand Khekale (https://www.anandk.dev), built on the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme. Content focuses on IBM i (AS/400) development topics.

## Commands

```bash
# Install dependencies
bundle install

# Serve locally with live reload (http://localhost:4000)
bundle exec jekyll serve

# Build static site
bundle exec jekyll build

# Preview using test/ directory content (http://localhost:4000/test/)
bundle exec rake preview
```

## Architecture

**Theme:** Uses `minimal-mistakes-jekyll` as a gem-based theme with the `contrast` skin. Theme files (layouts, includes, sass) live inside the gem, not in this repo — only overrides are stored locally.

**Key configuration files:**
- `_config.yml` — site-wide settings: theme skin, author profile, analytics (Google gtag `UA-175306064-1`), search (Lunr), permalink structure (`/:categories/:title/`), and post defaults
- `_data/navigation.yml` — top nav links (About, Posts, Books, Categories, Archive)

**Content structure:**
- `_posts/` — blog posts named `YYYY-MM-DD-title.markdown` with front matter including `layout`, `classes`, `title`, `header.og_image`, `categories`, `tags`, and `permalink`
- `_pages/` — static pages (about, books, search, sitemap) included via `include: [_pages]` in `_config.yml`
- `assets/` — images in `assets/images/forposts/` for post headers, resume PDF at `assets/Anand-Khekale-Resume.pdf`

**Layouts:** `_layouts/single.html` is a local override of the theme's single layout. All posts and pages default to `layout: single` with `author_profile: true`.

**Plugins in use:** `jekyll-paginate`, `jekyll-feed`, `jekyll-include-cache`, `jekyll-sitemap`, `jekyll-gist`, `jemoji`, `jekyll-compose`, `jekyll-redirect-from`, `jekyll-archives`

## Adding a New Post

Create `_posts/YYYY-MM-DD-post-title.markdown` with this front matter:

```yaml
---
layout: single
classes: wide
title: Post Title
header:
  og_image: /assets/images/forposts/image.png
categories:
  - ibmi
tags:
  - tag1
permalink: /:year/:month/:title:output_ext
---
```

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal blog for roelbondoc.com — a Jekyll 3.8 static site hosted on GitHub Pages with a custom domain.

## Build & Serve

```bash
bundle install            # install dependencies
bundle exec jekyll serve  # local dev server at http://localhost:4000
bundle exec jekyll build  # build to _site/
```

## Architecture

- **Jekyll 3.8** with plugins: jekyll-paginate, jekyll-sitemap, jekyll-seo-tag, jekyll-feed, jekyll-gist
- **Layouts** (`_layouts/`): `default.html` → `post.html` and `page.html`. Default includes masthead nav, newsletter signup (Buttondown), and content area.
- **Posts** (`_posts/`): Markdown files with standard Jekyll front matter. Paginated 5 per page on the index.
- **SEO**: Posts use `jekyll-seo-tag` plugin plus custom JSON-LD structured data in `post.html` layout.
- **Static assets** in `public/` — CSS in `public/css/`, JS in `public/js/`, favicons at top level of `public/`.
- **Pages**: `index.html` (paginated post list), `now.md`, `newsletter.md`, `404.md`.
- **Custom domain**: `roelbondoc.com` (configured via `CNAME` file).

## Writing Posts

Create files in `_posts/` following the naming convention `YYYY-MM-DD-title-slug.md`. Posts use `layout: post` and support `description` in front matter for SEO.

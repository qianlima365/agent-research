# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Agent Research — a Chinese-language Jekyll static site published via GitHub Pages at `https://yafeiwang.github.io/agent-research/`. The site catalogs AI Agent frameworks, open-source projects, development practices, tech stacks, and research notes.

## Build & Development Commands

```bash
# Install dependencies (Ruby + Bundler required)
bundle install

# Start local dev server with live reload
bundle exec jekyll serve --livereload

# Build for production
bundle exec jekyll build

# Clean build artifacts
bundle exec jekyll clean
```

Local site serves at `http://localhost:4000/agent-research/` (the `baseurl` is `/agent-research`).

## Architecture

### Jekyll Collections

The site defines five Jekyll collections in `_config.yml`, each mapped to a dedicated directory and a default layout/category via `defaults`:

| Collection | Directory | Category | Permalink Pattern |
|---|---|---|---|
| `frameworks` | `frameworks/` | 框架 | `/:collection/:name/` |
| `projects` | `projects/` | 开源项目 | `/:collection/:name/` |
| `development` | `development/` | 开发实践 | `/:collection/:name/` |
| `techstack` | `techstack/` | 技术栈 | `/:collection/:name/` |
| `research` | `research/` | 研究笔记 | `/:collection/:name/` |

Each collection directory contains an `index.md` that lists its members using Jekyll Liquid iteration. Adding a new markdown file to any collection directory automatically makes it appear in that section's index.

### Layouts

- `_layouts/page.html` — Custom layout used by all collection pages. Injects a category tag, title, optional date, and content into a clean HTML shell with top navigation linking to all five sections.
- `_layouts/home.html` — Custom layout for the root `index.md`.

### Styling

`assets/css/style.scss` contains the complete site stylesheet with a dark theme, custom typography, and component styling. Jekyll compiles `.scss` files automatically.

### Content Conventions

- Content is written in Chinese (zh-CN).
- All pages in collections use the `page` layout and inherit their `category` from `_config.yml` defaults — **do not** add `layout` or `category` to collection page frontmatter unless overriding.
- Collection index pages (e.g., `frameworks/index.md`) use Jekyll Liquid to auto-list child pages, filtering out the index itself by checking `item.title != "框架"`.
- The root `index.md` lists recently updated pages by checking `page.date`.

## Git Workflow

When adding or modifying content, commit and push directly after the work is done:

```bash
# Stage all changes
git add -A

# Commit with a descriptive message
git commit -m "描述本次更新的内容"

# Push to remote
git push origin main
```

GitHub Pages will rebuild and deploy the site automatically within a few minutes.

## Dependencies

- Jekyll (via GitHub Pages)
- Plugins: `jekyll-feed`, `jekyll-sitemap`
- No theme dependency — custom layouts and styles only

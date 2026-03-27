# notes

> my personal notes & blog — built with [Jekyll](https://jekyllrb.com/) and the [Beautiful Jekyll](https://beautifuljekyll.com/) theme.

## 📂 Category Structure

Posts are organized into four categories under `_posts/`:

| Directory | Category | Description |
|-----------|----------|-------------|
| `_posts/tech/` | **tech** | Programming, tools, and technical notes |
| `_posts/diary/` | **diary** | Personal thoughts and daily reflections |
| `_posts/game/` | **game** | Gaming reviews, tips, and musings |
| `_posts/work/` | **work** | Productivity, career, and work-life balance |

Draft posts that are not yet ready to publish live in `_drafts/`.

## ✍️ Writing a Post

1. Create a file in the appropriate `_posts/<category>/` directory.
2. Name it `YYYY-MM-DD-your-title.md`.
3. Add frontmatter at the top:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
category: tech   # tech | diary | game | work
tags: [tag1, tag2]
published: true
---
```

4. Write your content in Markdown below the frontmatter.

## 🚀 Local Development

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve --livereload

# Open http://localhost:4000 in your browser
```

## 🎨 Theme & Customization

- **Theme**: [Beautiful Jekyll](https://github.com/daattali/beautiful-jekyll) (`remote_theme: daattali/beautiful-jekyll`)
- **Fonts**: [Nunito](https://fonts.google.com/specimen/Nunito) & [Quicksand](https://fonts.google.com/specimen/Quicksand) via Google Fonts
- **Colors**: Soft pastel palette — pink, lavender, and mint (see `assets/css/custom.css`)

## 📦 Deployment

The site is automatically deployed to GitHub Pages via the GitHub Actions workflow in `.github/workflows/deploy.yml` on every push to `main`.

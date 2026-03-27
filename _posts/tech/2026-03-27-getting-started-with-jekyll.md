---
layout: post
title: "Getting Started with Jekyll"
date: 2026-03-27
category: tech
tags: [jekyll, github-pages, web]
published: false
---

Jekyll is a simple, blog-aware, static site generator. It takes text written in your favorite markup language and uses layouts to create a static website.

## Why Jekyll?

- **Simple**: No databases, no comment moderation, no pesky updates to install — just your content.
- **Static**: Markdown (or Textile), Liquid, HTML & CSS go in. Static sites come out ready for deployment.
- **Blog-aware**: Permalinks, categories, pages, posts, and custom layouts are all first-class citizens.

## Getting Started

1. Install Ruby and Bundler
2. Run `bundle install`
3. Run `bundle exec jekyll serve`
4. Open `http://localhost:4000` in your browser

## Writing a Post

Create a file in the `_posts/` directory named `YYYY-MM-DD-title.md` with frontmatter at the top:

```yaml
---
layout: post
title: "My Post Title"
date: 2026-03-27
category: tech
tags: [example, demo]
published: true
---
```

Happy blogging! 🎉
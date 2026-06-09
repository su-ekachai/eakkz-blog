# eakkz-blog

Personal blog built with [Zola](https://www.getzola.org/) and the [Serene](https://github.com/isunjn/serene) theme, deployed to GitHub Pages via GitHub Actions.

## Tech Stack

- **Static Site Generator:** Zola v0.22.1
- **Theme:** Serene v5.6.3
- **Deployment:** GitHub Pages (via `getzola/github-pages` action)
- **Styling:** Catppuccin color palette (Latte/Mocha), Inter + JetBrains Mono
- **Language:** English

## Prerequisites

- [Zola](https://www.getzola.org/documentation/getting-started/installation/) v0.22.1+

## Quick Start

```bash
git clone --recurse-submodules https://github.com/su-ekachai/eakkz-blog.git
cd eakkz-blog
zola serve
```

Open `http://127.0.0.1:1111` in your browser.

## Project Structure

```
.
├── config.toml          # Site configuration
├── content/
│   ├── _index.md        # Homepage
│   ├── posts/           # Blog posts
│   ├── about/           # About page
│   └── resume/          # Resume page
├── templates/
│   ├── _custom_css.html # Color theme overrides (Catppuccin)
│   └── robots.txt       # SEO robots template
├── static/              # Static assets (images, icons)
├── themes/serene/       # Theme (git submodule)
└── .github/workflows/
    └── deploy.yml       # GitHub Pages deployment
```

## Adding a New Post

Create a file in `content/posts/`:

```markdown
+++
title = "Post Title"
description = "A brief summary"
date = 2025-01-01
draft = false

[extra]
lang = "en"
toc = true
+++

Your content here.
```

## Deployment

Deployment is fully automated. Push to `main` and GitHub Actions will:

1. Check out the repo with submodules
2. Build with Zola v0.22.1
3. Deploy to GitHub Pages

Live at: https://su-ekachai.github.io/eakkz-blog

## Updating the Theme

```bash
cd themes/serene
git fetch --tags
git checkout v<new-version>
cd ../..
git add themes/serene
git commit -m "chore: bump serene theme to v<new-version>"
```

Check the [Serene CHANGELOG](https://github.com/isunjn/serene/blob/latest/CHANGELOG.md) for migration notes.

## License

Content is copyrighted. Theme is MIT licensed.

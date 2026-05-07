# CLAUDE.md

## Project Overview

Zola static blog using the Serene theme, deployed to GitHub Pages. English only.

## Commands

```bash
zola serve              # Dev server at localhost:1111 (live reload)
zola build              # Production build to public/
zola check              # Validate internal links
```

## Project Structure

| Path | Purpose |
|------|---------|
| `config.toml` | All site configuration (Zola settings + `[extra]` for theme) |
| `content/posts/` | Blog posts |
| `content/about/` | About page |
| `content/resume/` | Resume page |
| `templates/_custom_css.html` | Catppuccin color overrides and typography |
| `templates/_head_extend.html` | Custom `<head>` injections |
| `themes/serene/` | Theme (git submodule, do NOT edit directly) |
| `.github/workflows/deploy.yml` | GitHub Actions deployment |
| `static/` | Images, icons, generated CSS |

## Content Conventions

### Post Frontmatter

```toml
+++
title = "Title"
description = "Summary for SEO and cards"
date = YYYY-MM-DD
updated = YYYY-MM-DD
draft = false

[extra]
lang = "en"
toc = true         # table of contents
comment = true     # enable comments (if configured)
+++
```

## Config Structure (`config.toml`)

- **Top-level:** Zola built-in settings (base_url, compile_sass, etc.)
- **`[markdown]`:** Rendering options + `[markdown.highlighting]` for code blocks
- **`[extra]`:** Theme-specific settings (sections, navigation, footer, 404 text)

## Theme Management

The theme is a git submodule at `themes/serene/`.

- **Never edit files inside `themes/serene/`** — changes will be lost on update
- **Customize via:** `templates/_custom_css.html` for styles, `config.toml [extra]` for behavior
- **Update:** `cd themes/serene && git fetch --tags && git checkout <tag>`
- **After update:** Check CHANGELOG for breaking changes, run `zola build` to verify

## Deployment

Push to `main` triggers GitHub Actions:
1. Checkout with `submodules: recursive`
2. Build with Zola v0.22.1 (`getzola/github-pages@v1`)
3. Deploy via `actions/deploy-pages@v4`

GitHub Pages source must be set to "GitHub Actions" in repo settings.

## Gotchas

- `minify_html` is intentionally `false` — enabling it breaks some theme styles
- `build_search_index` is `false` — search is temporarily unsupported by Serene
- Highlighting uses `style = "class"` with giallo (auto-generates `giallo-light.css`/`giallo-dark.css` in `static/`)
- The `public/` directory is gitignored — it's a build artifact

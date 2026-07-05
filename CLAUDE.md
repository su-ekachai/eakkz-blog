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
| `templates/shortcodes/` | Custom shortcodes (`figure`, `pdf_embed`) |
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

### Images & assets

- **Post/page images — colocate them.** Promote a flat post to a page bundle on its first image: `content/posts/<slug>/index.md` with the image beside it (URL unchanged). Embed with the `figure` shortcode (build-time `resize_image`, lazy-loaded, optional caption):

  ```
  {{ figure(src="photo.jpg", alt="description", caption="optional", width=1000) }}
  ```

  Plain `![alt](photo.jpg)` also works (e.g. on a section's `_index.md`). Commit the full-res original; the displayed size is derived at build.
- **Site chrome** (avatar, favicon, share image): `static/img/` and `static/icon/`, referenced by the theme/config.
- **Filenames/formats:** lowercase-hyphenated; `.jpg`/`.webp` photos, `.png` screenshots, `.svg` vectors; always meaningful `alt` text.

## Writing Style

Published prose (posts, pages) must read as human-written, not
AI-generated. Apply the `blog-writing` skill (Draft, Humanize, Review
modes); its `references/blog-patterns.md` carries the full pattern list.
Hard rules that apply even when the skill is not loaded:

- **Em dashes:** avoid (hard max ~1 per 1,000 words, headings included).
- **Bold:** at most one bolded phrase per section, usually none.
- **Headings:** sentence case (`## The daily tax`); title case only for the post title.

Code blocks, quoted text, and cited examples are exempt.

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

- **Subpath:** `base_url` includes `/eakkz-blog`. Never use root-absolute paths for assets or internal links (`/img/…`, `/resume`) — they 404 in production. Use colocated relative paths, `@/file.md` internal links, or `get_url(path=…)` in templates.
- `minify_html` is intentionally `false` — enabling it breaks some theme styles
- `build_search_index` is `false` — search is temporarily unsupported by Serene
- Highlighting uses `style = "class"` with giallo (auto-generates `giallo-light.css`/`giallo-dark.css` in `static/`)
- The `public/` directory is gitignored — it's a build artifact

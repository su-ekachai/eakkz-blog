# Blog image & asset handling — design

- **Date:** 2026-06-08
- **Status:** Approved (direction); implementing welcome post + repairs in the same pass.

## Problem

The blog has no images yet, and both existing image references are broken:

- Homepage avatar `img/avatar.webp` — file missing.
- About page `![My Cat](/img/cat_photo.png)` — file missing **and** the path is
  subpath-unsafe.

`base_url` is `https://su-ekachai.github.io/eakkz-blog`. A root-absolute markdown
path like `/img/cat_photo.png` renders as `<img src="/img/cat_photo.png">`, which the
browser resolves against the **domain root** → `…github.io/img/cat_photo.png` (404),
not `…/eakkz-blog/img/...`. Same bug class as the resume PDF (fixed with `get_url`).

There is also no documented convention for where post images live or how to embed
them, which is the main thing to fix for future posts.

## Decision

Adopt the canonical Zola pattern, split by asset scope:

1. **Site chrome** (avatar, favicon, default share image) → `static/img/`,
   `static/icon/`. The theme already references these through
   `get_url(path=...)` (`home.html:44`), so they are subpath-safe.
2. **Page / post content images** → **colocated** with the page. Relative paths and
   `get_url`/`resize_image` outputs are inherently subpath-safe.
   - A post with assets becomes a **page bundle**: a folder containing `index.md`
     plus its images (e.g. `content/posts/welcome-to-my-blog/index.md` +
     `haku.jpg`). The public URL is unchanged (`/posts/welcome-to-my-blog/`).
   - A section landing page (`content/about/_index.md`) can colocate assets the same
     way — drop the image beside `_index.md` and reference it relatively.

Text-only posts stay as flat `.md` files. A post is promoted to a bundle folder the
moment it gains its first asset (YAGNI — no preemptive folders).

### Embedding: `figure` shortcode

`templates/shortcodes/figure.html`, using build-time `resize_image` on the colocated
source. Benefits over raw `![]()`: responsive/optimized output, explicit
`width`/`height` (no layout shift), `loading="lazy"`, optional caption, and
click-to-open at full resolution. Plain `![alt](file.jpg)` remains available for quick
cases and for section pages.

| Param | Required | Default | Purpose |
|-------|----------|---------|---------|
| `src` | yes | — | colocated filename (e.g. `haku.jpg`) |
| `alt` | yes | `""` | accessibility / SEO description |
| `caption` | no | — | visible `<figcaption>` |
| `width` | no | `1000` | target display width for `resize_image` (`op="fit_width"`) |

The full-resolution original stays committed in the bundle; `resize_image` writes the
display version to the build output (`processed_images/`, gitignored).

## Conventions (for future posts)

- **Location:** colocate; promote a post to `slug/index.md` on first asset.
- **Filenames:** lowercase, hyphenated, descriptive (`haku.jpg`, `pipeline-diagram.png`).
- **Format:** photos → `.jpg` (or `.webp`); screenshots / diagrams → `.png`; vectors
  → `.svg`. Keep the committed original reasonably sized (long edge ≲ 2000 px).
- **Always** provide meaningful `alt` text.
- Prefer the `figure` shortcode for post-body images so sizing/lazy-load/caption are
  handled consistently.

## Scope of this pass

1. Add `templates/shortcodes/figure.html` + theme-native `.post-figure` CSS in
   `templates/_custom_css.html`.
2. Migrate the welcome post to a bundle and embed `haku.jpg` via `figure`.
3. Repair the two broken refs: colocate the About cat photo (relative ref) and supply
   the homepage avatar. Source photo `haku.jpg` is reused as a sensible default for
   these (swap later if desired). Note: `sips` on this machine cannot emit WebP, so the
   avatar is a 256×256 `static/img/avatar.jpg` and `content/_index.md` points at
   `img/avatar.jpg`.

## Out of scope (follow-ups)

- Root-absolute **internal links** on the About page (`[Resume](/resume)`,
  `[GitHub]`, `[LinkedIn]`) — same subpath class for links, separate from images.
- The image-less `installing-timescaledb…` post stays flat `.md`.

## Verification

`zola build` succeeds; `public/posts/welcome-to-my-blog/index.html` contains the
`<figure>` with a `processed_images/…` `src` and intrinsic `width`/`height`; the About
image and avatar resolve under the `/eakkz-blog/` subpath; `zola check` shows only the
pre-existing About external-link warnings.

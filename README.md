# blender-bake-guides

Source for https://strataface.github.io/blender-bake-guides/ (GitHub Pages, Jekyll, minima theme).

## Layout
- `_config.yml` — site config (url, baseurl, theme).
- `index.md` — home page with the guide list.
- `<slug>/index.md` — one guide per folder; front matter has layout, title, description, permalink.
- `sitemap.xml` — **static** sitemap (no jekyll-sitemap plugin). Add a `<url>` entry for every new guide.
- `robots.txt` — allows all, points at the sitemap.

## Adding a guide
1. Create `<slug>/index.md` with the same front matter as the existing guide.
2. Add it to the list in `index.md` and to `sitemap.xml`.

## Search Console verification (reserved)
Note: a project site lives under `/blender-bake-guides/`, so Search Console should use a **URL-prefix property** `https://strataface.github.io/blender-bake-guides/`.
- Option A (recommended): put the `googleXXXXXXXXXXXXXXXX.html` file Google gives you in the **repository root**, next to this README. It is served at `https://strataface.github.io/blender-bake-guides/googleXXXX.html`.
- Option B: meta tag. Add `<meta name="google-site-verification" content="TOKEN">` to the page head (e.g. via `_includes/custom-head.html` with minima 2.5+, or the commented `google_site_verification` key in `_config.yml` if the theme version supports it).

## Enabling Pages (not done yet)
Settings ‣ Pages ‣ Deploy from branch ‣ `main` / root.

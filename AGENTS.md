# AGENTS.md

This is a small Hugo personal website. Keep it minimal, readable, and close to the existing Bearblog-inspired shape.

## Project Map

- `hugo.toml` is the site configuration.
- `content/` contains authored Markdown.
- `layouts/` contains local template overrides and is the source of truth for rendering.
- `themes/hugo-bearblog/` is the pinned upstream theme submodule. Do not edit it unless the task is explicitly about updating the theme.
- `static/` is copied directly to the published site root.
- `public/` is generated output and is ignored by Git.

## Commands

- Install theme submodule after a fresh clone: `git submodule update --init --recursive`
- Local preview: `hugo server --bind 127.0.0.1 --port 1313 --disableFastRender`
- Production build check: `hugo --gc --minify`
- List resolved content and URLs: `hugo list all`

## Architecture Notes

- The site uses `theme = "hugo-bearblog"`, but the important templates are overridden locally under `layouts/`.
- Main styling is inline in `layouts/partials/style.html`. `static/css/styles.css` currently is not linked by the templates.
- `content/blog/*` is routed to root-level post URLs through `[permalinks] blog = "/:slug/"`.
- `content/tech/*` keeps section URLs such as `/tech/inv_idx/`.
- Pages with `menu = "main"` appear in the top navigation. `Blog` is added by `layouts/partials/nav.html`.
- `content/hidden/` is not in the menu, but it is still published. Do not treat it as private unless the user changes the publication model.
- Taxonomy output is disabled in `hugo.toml`; tags may exist in front matter, but tag index pages are not part of the current site behavior.
- `enableGitInfo = true` and the base template prints `.Lastmod` on every page.

## Editing Rules

- Preserve the quiet personal-site feel: no heavy frameworks, no client-side app shell, no build pipeline unless the user asks.
- Prefer Markdown content changes over template changes when adding writing, notes, gallery entries, or resume updates.
- Prefer local layout overrides over direct theme edits.
- Do not edit generated files under `public/`.
- Keep content voice intact. This site mixes English and Chinese intentionally.
- When adding public pages, include concise front matter with `title` and, for dated writing, `date`.
- Run `hugo --gc --minify` after structural, template, or config changes.

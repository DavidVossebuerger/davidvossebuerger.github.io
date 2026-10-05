# David Vossebürger

Personal site built with [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod). Hosted on GitHub Pages.

**Live:** https://davidvossebuerger.github.io/

## Structure

- `content/` — Markdown pages (papers, projects, certificates)
- `layouts/` — Site overrides (extends PaperMod theme)
- `static/` — Static assets (certificates, SVGs, profile picture)
- `themes/PaperMod/` — Vendored theme (customized)
- `assets/` — CSS overrides
- `.github/workflows/hugo.yml` — GitHub Pages deploy workflow
- `config.yml` — Site config

## Local development

```bash
hugo server                    # http://localhost:1313
hugo --minify                  # production build → ./public/
```

Requires Hugo ≥ 0.165 (extended).

## Deployment

Pushes to `main` trigger `.github/workflows/hugo.yml` which builds with Hugo and deploys to GitHub Pages.

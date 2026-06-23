# TDLiDAR Docs

Source for the TDLiDAR documentation site, published with [just-the-docs](https://just-the-docs.com)
as a **GitHub Pages remote theme**.

**Live site:** https://tdlidar.github.io/DOC/

## How it builds

GitHub Pages builds this automatically — no CI, no local toolchain required. `_config.yml` pulls the
theme via `remote_theme: just-the-docs/just-the-docs`. Pushing to `main` republishes.

## Layout

| Path | Page |
|---|---|
| `index.md` | Home (overview + first patch) |
| `install.md` | Install the operator family |
| `app-guide.md` + `modes/` | App guide and per-mode pages |
| `operators.md` + `ops/` | Operator family — one page per operator |
| `osc-reference.md` | Full OSC wire spec |
| `operators-index.md` | Internal build index |

## Local preview (optional)

```sh
gem install bundler jekyll
echo "gem 'github-pages', group: :jekyll_plugins" > Gemfile
bundle install
bundle exec jekyll serve   # http://localhost:4000/DOC/
```

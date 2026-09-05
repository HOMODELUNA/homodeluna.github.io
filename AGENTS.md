# AGENTS.md — homodeluna.github.io

Personal blog / static site built with **Jekyll 4** and the **minima** theme, deployed to
GitHub Pages. Content is mostly Chinese / Esperanto / Japanese technical notes and conlang
(construction language) worldbuilding posts.

- Site title: `Domo de LunatikLandano`
- Live URL: <https://homodeluna.github.io>
- Default branch: `main` (pushing to `main` triggers the Pages deploy workflow)

---

## Project structure

```text
.
├── _config.yml          # Jekyll site config (title, theme, plugins, kramdown math)
├── Gemfile              # Ruby deps — jekyll, minima, kramdown-math-sskatex, execjs, duktape
├── Gemfile.lock         # generated; gitignored but present locally
├── _posts/              # ~23 markdown posts, YYYY-MM-DD-<title>.{md,markdown}
├── _sass/               # minima theme SCSS overrides (minima.scss + _base/_layout/
│                        #   _syntax-highlighting)
├── assets/              # images, css, js grouped by post/topic
│   ├── css/  js/        # katex.min.css / katex.min.js (math rendering)
│   ├── elsfa/ fielland/ simtober/ mathbook/ pic/ ref/   # per-topic media
│   └── *.puml / *.svg   # PlantUML diagrams (source + rendered SVG)
├── util/                # ruby helper scripts (excluded from build via _config exclude)
├── new-post.rb          # ruby helper to scaffold a new post with front-matter
├── about.markdown       # layout: page  → /pri/
├── index.markdown       # layout: home  → blog list
├── 404.html             # custom not-found page
└── .github/workflows/jekyll.yml   # build + deploy Jekyll site to Pages
```

Generated / ignored directories (do **not** edit by hand, do **not** commit):

- `_site/` — Jekyll build output (gitignored). May contain a full local build.
- `vendor/` — `bundle` install path (`.bundle/config` → `BUNDLE_PATH: vendor/bundle`).
- `.jekyll-cache/`, `.sass-cache/`, `.bundle/` — caches.

---

## Environment & common commands

Local Ruby in the dev environment: `ruby 3.2.3`, `bundler 2.4.x`. There is **no**
`.ruby-version` file; CI pins `ruby-version: '3.1'` (see workflow).

```bash
bundle install                 # install gems into vendor/bundle
bundle exec jekyll serve       # local dev server  → http://127.0.0.1:4000
bundle exec jekyll build       # production build  → ./_site
bundle exec jekyll doctor      # optional sanity check
```

Always run Jekyll through `bundle exec` so the locked versions are used.

### Create a new post

```bash
ruby new-post.rb "标题"                     # writes _posts/<today>-<标题>.md
ruby new-post.rb -c elsfa racket "标题" -q   # add categories, open in editor ($EDITOR default "code")
```

The script generates the `layout: post` front-matter block (title, date, categories).

---

## Conventions

### Posts (`_posts/`)

- Filename: `YYYY-MM-DD-<title>.md` (older posts use `.markdown`). The title segment may
  contain spaces and CJK characters; Jekyll slugifies it.
- Front-matter (matches what `new-post.rb` emits):

  ```yaml
  ---
  layout: post
  title:  伊勒斯法通用语
  date:   2024-02-15 23:23:13 +0800
  categories: elsfa
  ---
  ```

- `categories` drive URL grouping in `_site/` and the feed. Existing categories include:
  `elsfa`, `fielland`, `simtober`, `myljen`, `vsl`, `racket`, `language`, `c++`, `json`,
  `jekyll`, `update`. Some recent posts leave `categories:` empty — keep them as authored
  unless the task is explicitly to categorize.
- Markdown is rendered by **kramdown**. Math uses `math_engine: sskatex` with KaTeX
  (`assets/js/katex.min.js`, `assets/css/katex.min.css`) — `$$...$$` / `$...$` blocks.
- Reference PDFs / SVGs live in `assets/`; link with site-relative paths (`/assets/...`).

### Themes & styles (`_sass/`)

- The theme is `minima` (set in `_config.yml: theme: minima`). Customization happens in
  `_sass/minima.scss` and `_sass/minima/*.scss` — override variables here, don't patch the
  installed theme gem.
- `jekyll-remote-theme` is commented out in `_config.yml`; if enabled, it requires adding
  the plugin to `Gemfile` as well.

### Plugins & feed

- `jekyll-feed` is enabled → `feed.xml` is generated in `_site/`.
- Custom site variables live in `_config.yml` and are reachable in templates via
  `{{ site.<key> }}`.

---

## CI / deployment

`.github/workflows/jekyll.yml` runs on every push to `main` (and `workflow_dispatch`):

1. `ruby/setup-ruby@v1` (ruby 3.1, `bundler-cache: true`).
2. `bundle exec jekyll build` with `JEKYLL_ENV=production`.
3. Uploads `_site` and deploys via `actions/deploy-pages`.

There is no test suite or lint step. Verification is therefore **build-based**:

```bash
bundle exec jekyll build      # must succeed without errors
# or serve and click through changed pages
```

When touching `_config.yml` or `_sass/`, restart `jekyll serve` (config is not hot-reloaded)
and confirm the affected pages render.

---

## Gotchas

- `_site/`, `vendor/`, `Gemfile.lock`, `*.cache` are gitignored — never `git add -f` them.
- Bundle installs to `vendor/bundle` (from `.bundle/config`); a clean checkout needs
  `bundle install` before `jekyll serve` works.
- `util/` is excluded from the build (`_config.yml: exclude: [util/]`); it holds one-off
  scripts (e.g. `to-dialog.rb`, a chat-log → markdown dialog converter) with embedded
  sample data — treat as scratch, not part of the site.
- `Gemfile.lock` is gitignored yet committed locally with much newer gem versions than CI
  uses; don't rely on local `bundle update` to mirror CI behavior.
- `email:` in `_config.yml` is a placeholder (`via-email@example.com`).
- KaTeX math requires the JS/CSS assets to be present in the output; build the site and
  confirm `assets/js/katex.min.js` lands in `_site`.
- CJK / accented filenames are common; quote paths in shell commands.

---

## Editing workflow for agents

1. Inspect the relevant post / asset / `_config.yml` before changing it.
2. Make the smallest targeted edit (never regenerate `_site/` manually — rebuild it).
3. Run `bundle exec jekyll build` (and `jekyll serve` for visual checks) to confirm the
   change compiles before reporting done.
4. For new content, prefer `ruby new-post.rb` so front-matter matches existing posts.

# Kolloquium — Hybrid Plausibility Checking of Aviation Flight Data

Reveal.js slides for the Bachelor Kolloquium (Hochschule Darmstadt), built from
`slides.md` with pandoc. Based on the pandoc/reveal.js presentation template.

## Build locally

```shell
pandoc -t revealjs -s slides.md -o index.html \
  --slide-level=3 \
  --variable revealjs-url=https://cdn.jsdelivr.net/npm/reveal.js@5 \
  --variable width=1200 \
  --variable margin=0.04 \
  --include-after-body=mermaid-init.html
firefox index.html
```

## Export to PDF

```shell
npx decktape reveal index.html slides.pdf
```

## Deployment

Every push to `master`/`main` that touches `slides.md`, `styles.css`,
`mermaid-init.html`, or `assets/` triggers `.github/workflows/build_deploy.yml`:
pandoc builds `index.html` and the result is published to GitHub Pages
(the workflow enables Pages automatically on first run).

Presentation URL: `https://<user>.github.io/<repo>/`

## Files

| File | Purpose |
|---|---|
| `slides.md` | The slides (pandoc → reveal.js) |
| `assets/model_buildup.html` | Interactive Plotly comparison chart — model & metric toggles (embedded via iframe) |
| `styles.css` | Theme overrides (dark navy) |
| `mermaid-init.html` | Lazy mermaid support, injected after body |
| `script.md` | Word-for-word talk script with timings and toggle directions |
| `defense_cheatsheet.md` | Key numbers + likely Q&A for the defense |

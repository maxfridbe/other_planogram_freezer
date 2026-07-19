# FrostPlan Pro — Visual Freezer Planogram

A single-page web app for grocery store planners: arrange inventory in a top-down
view of a freezer chest with drag-and-drop, collision detection, automatic stack
calculation, and a print-ready schematic + stacking report.

- **Live site:** https://maxfridbe.github.io/other_planogram_freezer/
- **Spec:** [FRD.md](FRD.md)
- **App:** [index.html](index.html) — fully self-contained SPA (React 18 via Babel
  standalone, Tailwind CDN). Open the file directly in a browser or serve statically.

Deployment is automatic: pushes to `main` publish the repo root to GitHub Pages
via `.github/workflows/deploy-pages.yml`.

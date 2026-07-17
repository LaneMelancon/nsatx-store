# NeuroSolution ATX Supplement Store

**Maintained By:** Lane Melancon — Onn Grid, LLC **Client:** Dr. Brandon Crawford — NeuroSolution Center of Austin **Last Updated:** 2026-07-16

---

## Project Direction

This is the parent repository for the NeuroSolution Center of Austin Supplement Store project. It contains three git submodules — each an independent GitHub repo — plus top-level project documentation.

## GitHub Repositories

| Repo | URL | Purpose |
| --- | --- | --- |
| `nsatx-store` | `https://github.com/LaneMelancon/nsatx-store` | Parent repo (this file) |
| `nsatx-store-design-system` | `https://github.com/LaneMelancon/nsatx-store-design-system` | Design system — deployed to GitHub Pages |
| `nsatx-store-import-products` | `https://github.com/LaneMelancon/nsatx-store-import-products` | Shopify GraphQL bulk product import pipeline |
| `nsatx-store-shopify-theme` | `https://github.com/LaneMelancon/nsatx-store-shopify-theme` | Shopify theme — to be connected to dev store via GitHub integration |

## Directory Structure

```
nsatx-store/                  ← parent repo root
├── CLAUDE.md                 ← this file (includes this repo's Changelog)
├── .gitignore
├── .gitmodules               ← submodule config
├── design-system/            ← submodule: nsatx-store-design-system
│   ├── CLAUDE.md              ← design system overview + Changelog
│   └── DESIGN.md              ← full design system reference (tokens, type, etc.)
├── import-products/          ← submodule: nsatx-store-import-products (own CLAUDE.md + Changelog)
└── shopify-theme/            ← submodule: nsatx-store-shopify-theme (own CLAUDE.md + Changelog)
```

## Important Markdown Files

- `nsatx-store/CLAUDE.md` – (this file) **Read First, every session**
- `nsatx-store/design-system/CLAUDE.md` + `DESIGN.md` – design tokens/typography/spacing reference **Read only when working on design tokens, visual styling, or storefront/theme UI**
- `nsatx-store/shopify-theme/CLAUDE.md` – Shopify theme project guidelines and AI Toolkit/Dev MCP documentation **Read only when instructed to modify Shopify theme code**
- `nsatx-store/import-products/CLAUDE.md` – bulk product import project documentation **Read only when instructed to import products**

No file here should be read unconditionally except this one — the rest are scoped to the kind of
work actually being done, to keep session start-up cheap. Each of the 4 CLAUDE.md files (this one +
3 submodules) ends with its own `## Changelog` — a terse, dated table of what changed and why, one
row per session. Detailed rationale/debugging belongs in commit messages (`git log`/`git show` on
demand), not in the changelog — keep entries to 1-3 sentences. There is no separate memory/notes
file; each repo's own CLAUDE.md is the single source of truth for that repo's history, so a session
only needs to read the changelog(s) of the repo(s) it's actually touching.

## Working with Submodules

Each subdirectory is an independent git repo. Commit and push changes from within the submodule directory. To update the parent's submodule pointer after pushing a submodule change:

```bash
# from nsatx-store/ root
git add design-system   # or import-products / shopify-theme
git commit -m "Update <submodule> pointer"
git push
```

When cloning fresh:

```bash
git clone --recurse-submodules https://github.com/LaneMelancon/nsatx-store.git
```

## Rules

- Keep this markdown file clean and well structured as the high-level project overview
- After any session that introduces structural changes, new repos, new tooling, or completed milestones — update the relevant CLAUDE.md and add a one-line dated entry to its `## Changelog` table before closing out. Log a change in the CLAUDE.md of the repo it actually happened in (e.g. an import-products-only session logs to `import-products/CLAUDE.md`, not here) — only log here if it's cross-cutting (spans multiple submodules) or about the parent repo itself
- Use `design-system/CLAUDE.md` and `design-system/DESIGN.md` to reference design-related information for this project
- Use `import-products` only when instructed to import products into a Shopify store
- Read `/shopify-theme/CLAUDE.md` **only when instructed to modify** (change, adjust, create, delete, etc.) Shopify theme related code

## Changelog

| Date | Changes |
| --- | --- |
| 2026-07-16 | Consolidated project memory: retired the parent `MEMORY.md` file (was growing unbounded and wasn't consistently gated for reads) in favor of a `## Changelog` table in this file, matching the pattern `design-system/CLAUDE.md` and `shopify-theme/CLAUDE.md` already used independently. Gated `design-system/CLAUDE.md`/`DESIGN.md` reads to design-related work only (previously read unconditionally every session). Moved `import-products`' session history out of its narrative "Import Status" section into its own Changelog table — see that repo's CLAUDE.md for its history. |
| 2026-07-15 | Design-system → Shopify theme token bridge implemented (fonts, colors, spacing bridged via CSS custom properties at `:root`, no section/component files touched). Found an unresolved `badge_sale_text_color` contrast bug and confirmed the dev store's GitHub integration was already live. Full detail in `shopify-theme/CLAUDE.md`'s own Changelog. |

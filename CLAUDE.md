# NeuroSolution ATX Supplement Store

**Maintained By:** Lane Melancon — Onn Grid, LLC
**Client:** Dr. Brandon Crawford — NeuroSolution Center of Austin
**Last Updated:** 2026-06-29

---

## Project Direction

This is the parent repository for the NeuroSolution Center of Austin Supplement Store project. It contains three git submodules — each an independent GitHub repo — plus top-level project documentation.

## GitHub Repositories

| Repo | URL | Purpose |
| ---- | --- | ------- |
| `nsatx-store` | `https://github.com/LaneMelancon/nsatx-store` | Parent repo (this file) |
| `nsatx-store-design-system` | `https://github.com/LaneMelancon/nsatx-store-design-system` | Design system — deployed to GitHub Pages |
| `nsatx-store-import-products` | `https://github.com/LaneMelancon/nsatx-store-import-products` | Shopify GraphQL bulk product import pipeline |
| `nsatx-store-shopify-theme` | `https://github.com/LaneMelancon/nsatx-store-shopify-theme` | Shopify theme — to be connected to dev store via GitHub integration |

## Directory Structure

```
nsatx-store/                  ← parent repo root
├── CLAUDE.md                 ← this file
├── MEMORY.md                 ← local project memory
├── .gitignore
├── .gitmodules               ← submodule config
├── design-system/            ← submodule: nsatx-store-design-system
├── import-products/          ← submodule: nsatx-store-import-products
└── shopify-theme/            ← submodule: nsatx-store-shopify-theme
```

## Important Markdown Files
- `nsatx-store/CLAUDE.md` – (this file) **Read First**
- `nsatx-store/design-system/CLAUDE.md` – comprehensive design system documentation **Read Second**
- `nsatx-store/import-products/CLAUDE.md` – bulk product import project documentation **Read Third**
- `nsatx-store/shopify-theme/CLAUDE.md` – Shopify theme documentation **Read Fourth**

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
- After any session that introduces structural changes, new repos, new tooling, or completed milestones — update the relevant CLAUDE.md files to reflect the current state before closing out
- Use `design-system` to reference design-related information for this project
- Use `import-products` only when instructed to import products into Shopify

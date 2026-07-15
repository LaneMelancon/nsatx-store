# NeuroSolution ATX Supplement Store

**Maintained By:** Lane Melancon — Onn Grid, LLC **Client:** Dr. Brandon Crawford — NeuroSolution Center of Austin **Last Updated:** 2026-07-15

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
├── CLAUDE.md                 ← this file
├── MEMORY.md                 ← local project memory
├── .gitignore
├── .gitmodules               ← submodule config
├── design-system/            ← submodule: nsatx-store-design-system
│   ├── CLAUDE.md              ← design system high-level overview
│   └── DESIGN.md              ← full design system reference (tokens, type, components)
├── import-products/          ← submodule: nsatx-store-import-products
└── shopify-theme/            ← submodule: nsatx-store-shopify-theme
```

## Important Markdown Files

- `nsatx-store/CLAUDE.md` – (this file) **Read First**
- `nsatx-store/design-system/CLAUDE.md` – design system high-level overview **Read Second**
- `nsatx-store/design-system/DESIGN.md` – comprehensive design system reference (tokens, typography, components) **Read Third**
- `nsatx-store/shopify-theme/CLAUDE.md` – Shopify theme project guidelines and important AI Toolkit/Dev MCP documentation **Read Only When Instructed (see rules)**
- `nsatx-store/import-products/CLAUDE.md` – bulk product import project documentation **Read Only When Instructed to Import Products**

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
- After any session that introduces structural changes, new repos, new tooling, or completed milestones — update the relevant markdown files to reflect the current state and add a brief changlog of the session in `MEMORY.md` before closing out
- Use `design-system/CLAUDE.md` and `design-system/DESIGN.md`to reference design-related information for this project
- Use `import-products` only when instructed to import products into a Shopify store
- Read `/shopify-theme/CLAUDE.md` **only when instructed to modify** (change, adjust, create, delete, etc.) Shopify theme related code

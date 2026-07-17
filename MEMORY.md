# NSATX Store — Session Memory

## 2026-07-16 (3) — Migrated off native metaobjects to fully custom ones; 15-product test import

Follow-up to the same-day CLI-auth-removal session below. Lane updated the source CSVs (renamed
`sheet_supplement_categories.csv`→`sheet_benefits.csv`, `sheet_food_supplement_forms.csv`→`sheet_forms.csv`,
updated column headers to the new metafield keys, added `Image File`/`CDN URL` columns) and asked to
replace Shopify's native `shopify--flavor`/`shopify--medicine-supplement-form` metaobjects (used since
the original import design) with fully custom ones, renamed `supplement_category`→`supplement_benefit`,
all in Title Case, plus fix a real bug: entry handles getting random-character suffixes instead of
clean slugs.

- **Root cause of the random-suffix handle bug:** `metaobjectDefinitionCreate` was never passing
  `displayNameKey`, so Shopify fell back to an auto-generated display name for entries instead of
  using the `name`/`label` field's value, and handles derived from that auto-name picked up random
  suffixes. Fix: `01-setup-metaobjects.js` now sets `displayNameKey: 'name'` (Supplement Benefit) /
  `displayNameKey: 'label'` (Supplement Form, Supplement Flavor) on each definition, plus
  `capabilities: { adminFilterable: { enabled: true } }` on that same field ("Fields used as filter"
  in the admin UI) — both set on the definition *before* `02-setup-metaobject-entries.js` creates any
  entries, per Lane's explicit requirement. Verified live: all 43 entries this session got clean
  handles (`mango-strawberry`, `slightly-seaweed`, `food-topper`, etc.), zero random suffixes.
- Tried setting `access: { admin: 'MERCHANT_READ_WRITE', storefront: 'PUBLIC_READ' }` on the new
  definitions first — Shopify rejected it: `ADMIN_ACCESS_INPUT_NOT_ALLOWED — Admin access can only be
  specified on metaobject definitions that have an app-reserved type`. Merchant-writable is already
  the implicit default for a plain (non-`$app:`-prefixed) type; dropped `admin` from the access input,
  kept `storefront: 'PUBLIC_READ'`.
- Confirmed via [Shopify's docs](https://shopify.dev/docs/apps/build/product-merchandising/products-and-collections/metafield-linked)
  before implementing: `linkedMetafield`/`linkedMetafieldValue` on product options works fine with
  **custom** metaobject-reference metafields, not just native/standard ones — the only requirements
  are owner type `Product`, type `list.metaobject_reference`, and a merchant-writable metaobject
  definition. Verified live after the fact too: `resvero-active`, `gut-feeling`, `immunog-prp`,
  `optimag-neuro` all show correctly linked (`Formula -> custom.supplement_form`, `Flavor ->
  custom.supplement_flavor`) via `04-verify.js`.
- New architecture (all custom, all Title Case): metaobjects `Supplement Benefit` (type
  `supplement_benefit`, fields `name`/`description`/`icon`/`image`, unchanged from the old
  `supplement_category` besides the rename), `Supplement Form` and `Supplement Flavor` (types
  `supplement_form`/`supplement_flavor`, single `label` field each — simpler than the native
  metaobjects they replaced, which also required a `taxonomy_reference` field just to satisfy
  Shopify's native-definition constraints; custom ones don't need it, so `FLAVOR_TAXONOMY`/
  `FORM_TAXONOMY` mapping tables were deleted from `02-setup-metaobject-entries.js` entirely).
  Matching product metafields `custom.supplement_benefit`/`custom.supplement_form`/
  `custom.supplement_flavor` (all `list.metaobject_reference`, all namespace `custom` — previously
  `flavor`/`food-supplement-form` were namespace `shopify`).
- Also handled per Lane's explicit ask: `custom.variant_title` is now pinned via
  `metafieldDefinitionPin` (reused the existing helper already used for `product_details`) — unpinned
  metafield definitions don't show up on the variant edit page in Admin at all.
- Files changed: `import-products/api.js` untouched (transport-agnostic to the metafield rename);
  `scripts/01-setup-metaobjects.js` and `scripts/02-setup-metaobject-entries.js` rewritten;
  `scripts/03-import-products.js` updated (`OPTION_LINK_MAP`, column header constants, and
  `buildProductMetafields()` all repointed at the new custom keys); `scripts/00-wipe.js` and
  `scripts/04-verify.js` updated (wipe's `metaobjectTypes` array, verify's doc comments — its actual
  logic needed no changes since it reports whatever's linked dynamically).
- Ran for real: `00-wipe.js` (old structure — 10 products, 3 old metaobject defs, 4 metafield defs)
  → rewrote scripts → `01` → `02` → `03 --limit=10` → `03 --handles=resvero-active,gut-feeling,
  immunog-prp,optimag-neuro,energybits`. 15/15 products succeeded, all DRAFT, published to all 3
  channels, linked options verified correct.
- Product images are **still** broken (same pre-existing dead CDN URLs from the 2026-07-16 (2) entry
  below) — unrelated to this migration, not addressed this session (image upload step still deferred
  per Lane). The new `Image File`/`CDN URL` sheet columns aren't consumed by any script yet.
- Store currently has 15 test products under the new custom-metaobject structure — not yet re-wiped
  or rebuilt to the full 140.

## 2026-07-16 (2) — Removed atx-prod-ingest app auth entirely; 10-product test import via CLI

Follow-up to the same-day CLI-auth session below: removed the old Client Credentials Grant path from `import-products/api.js` outright (no more dual-mode) and ran a real end-to-end test — metaobjects, metaobject entries, and a 10-product import — through the CLI-only transport.

- `api.js` rewritten to a single ~45-line `graphql()` that always shells out to `shopify store execute`; deleted `getToken()`, the fetch-based Client Credentials flow, and the `CLIENT_ID`/`CLIENT_SECRET`/`USE_CLI` branching. No script changes needed elsewhere — all of them already only imported `graphql`.
- `import-products/CLAUDE.md` rewritten throughout: single "API Authentication" method (CLI store auth), Stores table, File Structure, Import Sequence note, Production Import Prep checklist (now just "create `.env.production` with `SHOPIFY_SHOP=neurosolution-shop`, run `shopify store auth` against it" — no app-install step), and Important Rules all updated. No functional references to `atx-prod-ingest`/Client Credentials Grant remain anywhere in the submodule (grepped clean).
- Ran the real pipeline against `atx-prod-test-01.myshopify.com`: `01-setup-metaobjects.js` → `02-setup-metaobject-entries.js` → `03-import-products.js --limit=10` → `--publish-only` → `04-verify.js`. All succeeded (10/10 products created, DRAFT, published to all 3 channels).
- Hit and fixed one unrelated blocker: `03-import-products.js` expects a manual "Supplements" collection (handle `supplements`) to already exist — it doesn't create one, only looks it up. It was missing (likely cleared by an earlier full store reset, not by `00-wipe.js`, which never touches collections). Created it via a one-off `collectionCreate` mutation; it's now permanent on the dev store.
- Re-ran `shopify store auth` mid-session with the full documented scope list (added `read_inventory`, `write_inventory`, `read_product_feeds`, `write_product_feeds`, `read_product_listings`, `write_product_listings`, `read_publications`, `write_publications`, `write_theme_code`, `read_themes`, `write_themes` on top of the original wipe-only scopes) so publishing worked without a second auth prompt going forward.
- `04-verify.js`'s product-count mismatch (10 vs expected 140) and "MISSING" linked-metafield-option warnings (for `resvero-active`/`gut-feeling`/`immunog-prp`/`optimag-neuro`) are expected — those 4 handles aren't among the first 10 products in sheet order, and we only imported 10 intentionally. Not a regression.
- **Found, unrelated to this work:** all 10 test products' images 404 — confirmed via a direct `curl` that the CDN URLs hardcoded in `sheets/atx-prod-test-01_sheet_products.csv` (e.g. `https://cdn.shopify.com/s/files/1/0789/1288/0789/files/...`) are dead independent of store/auth. Likely stale references to files uploaded before an earlier full store reset (Settings → Files isn't touched by `00-wipe.js`). Needs new image uploads or corrected CDN URLs in the source sheet before a full rebuild — not an auth or code issue.
- Store now has: full metaobject/metafield setup, all 43 metaobject entries, the `supplements` collection, and 10 live DRAFT products (broken images). Not yet re-wiped.

## 2026-07-16 — Store wipe via Shopify CLI auth (no atx-prod-ingest app credentials)

Added a second auth path to `import-products/api.js` so `00-wipe.js` (and any other import script) can hit the dev store through the Shopify CLI's `shopify store auth`/`shopify store execute` session instead of the `atx-prod-ingest` custom app's Client Credentials Grant — no client ID/secret needed for dev-store work.

- `api.js` now auto-selects transport: if `SHOPIFY_CLIENT_ID`/`SHOPIFY_CLIENT_SECRET` are unset (only `SHOPIFY_SHOP` present), `graphql()` shells out to `shopify store execute --store <shop>.myshopify.com --query ... --json [--allow-mutations]` via `node:child_process`; otherwise it falls back to the original fetch-based Client Credentials flow unchanged. Every script gets this for free since they all import `graphql()` from the same file.
- Confirmed via live testing against `atx-prod-test-01.myshopify.com`: `shopify store execute --json` prints the bare `data` payload on stdout (not `{data, errors}`), and GraphQL-level errors come back as a non-zero exit with a formatted box on stderr rather than a JSON `errors` key — `graphqlViaCli()` handles both cases; `withRetry()`'s `/THROTTLED/` regex still matches since the full stderr text is preserved in the thrown error.
- Ran `shopify store auth --store atx-prod-test-01.myshopify.com --scopes read_products,write_products,read_metaobject_definitions,write_metaobject_definitions,read_metaobjects,write_metaobjects`, then `node --env-file=.env scripts/00-wipe.js`. Store already had 0 products; wipe removed the 3 metaobject definitions (`supplement_category`, `shopify--flavor`, `shopify--medicine-supplement-form`) and 4 metafield definitions (`custom.product_details`, `custom.supplement_ingredients`, `custom.supplement_size`, `custom.variant_title`). Verified clean afterward (0 products, 0 metaobject defs, 0 metafield defs).
- Created a new `import-products/.env` with just `SHOPIFY_SHOP=atx-prod-test-01` (no app credentials) to trigger the CLI path — this is what's on disk now for the dev store.
- Production (`neurosolution-shop.myshopify.com`) is untouched by this change — it still requires `.env.production` with real `atx-prod-ingest` client credentials per the existing production-prep checklist in `import-products/CLAUDE.md`; the CLI path was only exercised against the dev store.

## 2026-07-15 — Design-system → Shopify theme token transition implemented

Executed the plan in `shopify-theme/TRANSITION.md`: bridged the design system's tokens (fonts, colors, spacing) into Tinker's CSS custom property layer, entirely via `:root` overrides — no `sections/`, `blocks/`, or per-component `snippets/` files were touched.

- Self-hosted Objective + Trade Gothic LT (14 `.woff2` files copied to `shopify-theme/assets/`, `snippets/brand-fonts.liquid` added with `@font-face` + sparse preloading).
- Added `shopify-theme/assets/design-tokens.css` (verbatim copy of `design-system/css/tokens.css`) and `shopify-theme/snippets/token-bridge.liquid`, overriding ~25 Tinker color/font/spacing variables at `:root`.
- Replaced the theme's 6-key `color_palette` with a 13-key design-system ramp in `settings_schema.json`/`settings_data.json`; remapped the 6 settings that referenced the old `color1`–`color4` keys.
- Hid (via `visible_if`, not deletion) the 4 vestigial font-picker settings.
- Realigned `assets/base.css`'s local hover/transition variables to the design system's duration/easing tokens.
- `shopify theme check` passed clean (329 files, no offenses). Verified via `shopify theme dev` against the dev store (`atx-prod-test-01.myshopify.com`) — confirmed fonts/design-tokens.css resolve with 200s, and the token-bridge overrides render after `theme-styles-variables`/`color-palette` in the cascade as intended.
- Committed inside `shopify-theme/` and updated the parent submodule pointer. **Not pushed** — GitHub integration on `atx-prod-test-01.myshopify.com` is confirmed live (auto-deploys on push to `main`), so a push needs explicit sign-off first.

**Found during verification (unresolved):** `badge_sale_text_color` and the newly-remapped `badge_sale_background_color` both resolve to the `foreground` palette key, making "Sale" badge text invisible (0 contrast). The background remap was an explicit, confirmed choice in `TRANSITION.md`; the text-color side wasn't part of that decision. Needs a manual fix in the Theme Editor (point `badge_sale_text_color` at `background` instead).

**Also found:** `nsatx-store/CLAUDE.md`/`shopify-theme/CLAUDE.md` said GitHub integration was "pending setup" — it's actually already connected and live (`nsatx-store-shopify-theme/main` is the published theme on the dev store). Docs updated to reflect this.

**Left running:** a `shopify theme dev` background process against `atx-prod-test-01.myshopify.com` for manual visual/cross-browser review. Preview: `https://atx-prod-test-01.myshopify.com/?preview_theme_id=159364939925`.

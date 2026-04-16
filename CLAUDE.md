# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MARILILU is a single-file Israeli swimwear brand web app — a mobile-first PWA (max-width 430px, RTL/Hebrew) deployed on Vercel, connected to GitHub at `itamar1199/marililu`.

The entire app lives in **`index.html`** — one file containing all HTML, CSS, and JavaScript. There is no build step, no npm, no bundler.

## Development

Open `index.html` directly in a browser, or use any static file server:

```bash
npx serve .
# or
python -m http.server
```

Deploy by pushing to `main` — Vercel auto-deploys. **Always commit and push after every change** so the live site updates.

## Architecture

**Single file: `index.html`** (~2400 lines)

- **CSS** (~1300 lines): design tokens in `:root`, then per-screen and per-component styles.
- **HTML**: `#app` shell (max-width 430px) containing four `.screen` sections. Only the `.active` screen is visible. Navigation overlays (hamburger, drawer, modals, sheets) live outside `#app`.
- **JavaScript** (~500 lines, inline `<script>`): no framework, plain DOM manipulation.

**Four screens** (toggled via `navigate(screenName)`):
1. `screen-home` — Hero, About, Designer card, Testimonials, Instagram grid, Contact form
2. `screen-custom` — 4-step custom order wizard (fabric → color → cut → size)
3. `screen-collection` — Filterable product grid + inspiration strip
4. `screen-cart` — Cart list, order summary, checkout form

**Navigation:**
- Fixed hamburger button (top-right, `#hamburger-btn`) opens a slide-in drawer (`#nav-drawer`) from the right.
- `navigate(screen)` handles screen switching with directional slide animations (`slide-from-right` / `slide-from-left`) based on `screenOrder` indices. It also saves the current screen to `localStorage` so it's restored on refresh.
- Drawer contains: logo, 4 nav items, contact links (WhatsApp, Instagram, email placeholder).

**State:**
- `cart[]` — array of cart item objects, persisted in `localStorage` (`marililu_cart`). Call `saveCart()` after every mutation.
- `customOrder{}` — wizard selections (`fabric`, `color`, `colorHex`, `cut`, `size`)
- `currentStep` — integer 0–3 for the custom order wizard
- `currentScreen` — string, tracks active screen for slide direction
- `sheetProductId` / `sheetSelectedSize` — product quick-view sheet state

**Overlays / modals (outside `#app`):**
- `#success-modal` — order/contact confirmation
- `#toast` — brief notification strip
- `#sheet-overlay` + `#product-sheet` — product quick-view bottom sheet (opens on product card tap, has size picker)
- `#size-guide-overlay` — size chart modal (triggered from step 4 of custom wizard)
- `#nav-overlay` + `#nav-drawer` — navigation drawer

**Backend:** Supabase (anon key in source). One table: `leads` — written on contact form and checkout via `saveLead()`.

**Email:** EmailJS configured at top of `<script>` with three placeholder constants (`_EJS_PUBLIC_KEY`, `_EJS_SERVICE_ID`, `_EJS_TEMPLATE_ID`). The code no-ops safely until placeholders are replaced.

**PWA:** `manifest.json` at root — name, theme color, SVG icon. `<link rel="manifest">` and Apple meta tags in `<head>`.

## Assets

| File/Folder | Purpose |
|---|---|
| `logo.svg` | Red square with "MARILILU" text (Cormorant Garamond, pink on red) |
| `manifest.json` | PWA manifest |
| `og-image.jpg` | OG/social preview image (1200×630, dark bg + red logo square) — must be JPEG, not SVG |
| `designer.jpg` | Designer portrait photo shown in the home screen designer card |
| `imgs/` | All product and fabric photos with clean English filenames |
| `תמונות בגד ים/` | Raw source photos (WhatsApp exports) — copy to `imgs/` with clean names before using |

**`imgs/` naming convention:**
- `prod-{name}.jpg` — product photos for collection screen (coral, stripe, lavender, emerald, leopard, rust, navy)
- `fabric-{name}.jpg` — fabric texture photos for wizard step 1 (solid, stripe, leopard, tiger)

## Products

Products are defined in the `products` array in `<script>`. Each object: `{ id, name, type, price, desc, color, img }`. No Unsplash URLs — all `img` fields point to `/imgs/prod-*.jpg`.

Current products (all type `סט מלא`, ₪380):
- The Coral, The Stripe, The Lavender, The Emerald, The Leopard, The Rust, The Navy

`renderProducts()` renders `<img>` tags directly (no placeholder slots). `openProductSheet()` also renders the image directly into the sheet.

Filter tabs: הכל / סטים / הדפסים / צבע אחיד — `patternIds` array in `renderProducts()` controls which product IDs count as patterned.

## Custom Order Wizard

4 steps, each a `.step-panel` div shown/hidden by `showStep()`:

| Step | ID | Content |
|---|---|---|
| 1 — Fabric | `step-0` | 4 fabric cards with real texture photos: חלק / פסים / הדפס נמר / הדפס זברה |
| 2 — Color | `step-1` | Color swatches matching actual available colors: קורל, חרדל שרוף, לבנדר, ירוק כהה, טורקיז, נייבי, שוקולד, שחור, אפרסק, קרם |
| 3 — Cut | `step-2` | Cut cards with real product photos as illustrations |
| 4 — Size | `step-3` | XS–XXL size buttons + size guide modal trigger |

## Design Tokens

| Token | Value | Usage |
|---|---|---|
| `--vanilla` | `#F1EADA` | Light cream backgrounds, text on dark |
| `--tobacco` | `#B59E7D` | Accents, badges, active states |
| `--mountain` | `#AAA398` | Secondary text, muted elements |
| `--sand` | `#CEC1A8` | Borders, inactive states |
| `--mahogany` | `#3D2318` | Primary dark — headers, buttons, text |
| `--mahogany2` | `#5C3825` | Hover states |
| `--dark` | `#1C1008` | Body background |
| `--light-bg` | `#FAF7F2` | App shell background |

- `--font-serif` = Cormorant Garamond (headings, prices, labels)
- `--font-sans` = Heebo (body, buttons)

## Key Conventions

- All user-facing text is Hebrew; brand name and product names are English
- Prices in ILS (₪), free shipping above ₪300; custom order fixed at ₪380
- z-index layers: screens (default) → sheet overlay (8000) → product sheet (8500) → nav overlay (9000) → nav drawer (9500) → hamburger button (10000)
- The `mailto:YOUR_EMAIL` in the nav drawer contact section is a placeholder pending the brand email address
- OG image must always be JPEG or PNG — WhatsApp does not render SVG in link previews
- When adding new product photos: copy raw file from `תמונות בגד ים/` to `imgs/` with a clean English filename, then reference `/imgs/filename.jpg` in the `products` array

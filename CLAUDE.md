# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MARILILU is a single-file Israeli swimwear brand web app — a mobile-first PWA-style page (max-width 430px, RTL/Hebrew) deployed on Vercel, connected to GitHub at `itamar1199/marililu`.

The entire app lives in **`index.html`** — one file containing all HTML, CSS, and JavaScript. There is no build step, no npm, no bundler.

## Development

Open `index.html` directly in a browser, or use any static file server:

```bash
npx serve .
# or
python -m http.server
```

Deploy by pushing to `main` — Vercel auto-deploys.

## Architecture

**Single file: `index.html`**

- **CSS** (~1000 lines): design tokens in `:root`, then per-screen component styles. Token names: `--vanilla`, `--tobacco`, `--mountain`, `--sand`, `--mahogany`, `--mahogany2`, `--dark`, `--light-bg`.
- **HTML structure**: `#app` shell (max-width 430px) containing four `.screen` sections + a fixed `#bottom-nav`. Only the `.active` screen is visible.
- **JavaScript** (~350 lines, inline `<script>`): no framework, plain DOM manipulation.

**Four screens** (toggled via `navigate(screenName)`):
1. `screen-home` — Hero, About, Designer card, Instagram grid, Contact form
2. `screen-custom` — 4-step custom order wizard (fabric → color → cut → size)
3. `screen-collection` — Filterable product grid + inspiration strip
4. `screen-cart` — Cart list, order summary, checkout form

**State:**
- `cart[]` — array of cart item objects
- `customOrder{}` — wizard selections (`fabric`, `color`, `colorHex`, `cut`, `size`)
- `currentStep` — integer 0–3 for the custom order wizard

**Backend:** Supabase (anon key in source). Only one table used: `leads` — written on contact form submit and checkout submit via `saveLead()`.

**Assets:**
- `logo.svg` — red square with "MARILILU" text (Cormorant Garamond, pink on red)
- `תמונות בגד ים/` — folder of product/hero photos (WhatsApp exports, JPEG)
- Hero background: `תמונות בגד ים/WhatsApp Image 2026-04-11 at 20.49.03.jpeg`

**Image pattern:** Many sections use `.img-empty-slot` placeholder divs waiting for real photos. Product cards in `renderProducts()` have Unsplash URLs on each product object but currently render as placeholders — swapping them to `<img>` tags is a known pending task.

## Key Conventions

- Fonts: `--font-serif` = Cormorant Garamond (headings, prices, labels), `--font-sans` = Heebo (body, buttons)
- All user-facing text is Hebrew; brand name and product names are English
- Prices in ILS (₪), free shipping above ₪300
- Custom order price is fixed at ₪380

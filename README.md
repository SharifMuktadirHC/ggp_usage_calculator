# Gogopower Power Usage Calculator

Self-contained widget (HTML + CSS + JS + Liquid in one file) for the Gogopower Shopify store.

## Install

1. Shopify Admin → Online Store → Themes → Customize
2. Add a **Custom Liquid** section/block on the page you want the calculator (e.g. a "Solar & Generator Calculator" page)
3. Open `gogopower-energy-calculator.liquid`, copy the **entire file**, paste it into the Custom Liquid block, and Save
4. Preview the page on desktop and mobile

## Configuration

All settings live in one block near the top of the file:

```liquid
{%- assign ggp_collection_handle = 'generator' -%}
{%- assign ggp_kva_namespace = 'custom' -%}
{%- assign ggp_kva_key = 'prime_power' -%}
{%- assign ggp_contact_url = 'https://www.gogopower.com.au/pages/contact' -%}
{%- assign ggp_power_factor = 0.8 -%}
{%- assign ggp_safety_margin = 1.3 -%}
```

| Setting | What it does |
|---|---|
| `ggp_collection_handle` | The collection (`/collections/<handle>`) the calculator searches for products to recommend. **Double-check this is the exact handle** — the URL you gave was `/collections/generator/products/`, so `generator` was used. |
| `ggp_kva_namespace` / `ggp_kva_key` | Metafield holding each product's kVA rating. Currently set to `custom.prime_power`. |
| `ggp_contact_url` | Where the final "Talk to Our Team" button links to. |
| `ggp_power_factor` | Used to convert total Watts → kVA (default 0.8, typical for mixed loads). |
| `ggp_safety_margin` | Headroom added on top of connected load before matching a generator (default 1.3 = 30%). |

Only products that have a value in the kVA metafield are included — products with it blank are skipped automatically, so there's no need to filter the collection manually.

**Price** is not a metafield — it's pulled straight from each product's own price via `product.price | money`, formatted using your store's money format automatically. There is no separate namespace/key to configure for it.

Note: an earlier version of this widget also showed a "brand" field sourced from a `custom.engine` metafield. That was removed — `custom.engine` turned out to be a reference-type metafield (not plain text), which Shopify returns as a structured object rather than a string, and rendered as `[object Object]` in the UI. Price replaces it.

## What's included

- **Guideline intro** — 4-step "how it works" panel shown before the calculator, with a "Start Calculator" button
- **4 appliance categories** — Household, Commercial, Industrial, Agriculture, each with ~10-16 common appliances (icon + name on one line, tap to add)
- **Editable list** — every add creates its own row (qty, watts, hours/day, computed kWh/day); the same appliance can be added multiple times with different wattages; a "+ Add custom appliance" option covers anything not in the presets
- **Live totals** — total connected load (W), estimated daily usage (kWh/day), recommended capacity (kVA)
- **Product suggestions** — pulled live from your `generator` collection; shows name, kVA and price, and links straight to the product page
- **Contact CTA** — button at the end linking to your Contact page
- **Responsive** — grid/table/cards reflow for mobile; brand colours (white, black, `#032B44`, `#3C9342`) throughout

## Notes / things to sanity-check after pasting

- Default running watts/hours per appliance are approximate industry figures — editable per row by the customer, and you can adjust the defaults in the `APPLIANCES` object in the `<script>` if you want different starting values.
- If `generator` isn't the correct collection handle, or `custom.prime_power` isn't the right metafield, the "Recommended Gogopower Generators" section will simply show no matches — update the config block and re-save.
- The sizing formula (`kVA = totalWatts × 1.3 ÷ (1000 × 0.8)`) is a simplified rule of thumb for non-technical visitors, not a substitute for a proper load study — the "Talk to Our Team" CTA is there for anyone who needs a precise recommendation.

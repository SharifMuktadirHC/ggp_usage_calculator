# Gogopower Power Usage Calculator

Two separate, complementary tools live in this repo, each shipped two ways (a theme section with settings, and a standalone Custom Liquid paste-in version):

## 1. Power Usage Calculator (appliance load list -> recommended generator)

- **`sections/ggp-power-calculator.liquid`** — a real theme section with a `{% schema %}` block. Recommended: settings are edited from the theme customizer (collection picker, contact link, etc.) instead of code, and it can be deployed via git/GitHub instead of copy-paste. See [Install as a theme section](#install-as-a-theme-section) below.
- **`gogopower-energy-calculator.liquid`** — the standalone version meant to be pasted into a "Custom Liquid" block. Kept for reference / stores that can't add theme files. See [Install via Custom Liquid (copy-paste)](#install-via-custom-liquid-copy-paste) below.

Functionally the two are identical. Lets visitors pick appliances (household/commercial/industrial/agriculture), builds a load list, and estimates daily usage + recommended generator kVA with matching products.

**Lead-gated recommendation:** Total Connected Load and Estimated Daily Usage are shown immediately. Recommended Capacity and the matching-product suggestions are hidden behind a lead-capture step - visitors click "Show My Recommendation," which loads your real **Pipedrive web form** (the same embed used elsewhere on the site: `PIPEDRIVE_FORM_URL` near the top of the `<script>` block), and once they submit it, the results reveal.

Unlocking happens two ways: (1) automatically, via a best-effort listener for Pipedrive's postMessage events (unverified against Pipedrive's exact message format - test this live), and (2) always available as a manual fallback: an "I've submitted the form — Show My Recommendation" button beneath the embed, so the feature never gets stuck even if the automatic detection doesn't fire. Leads land in Pipedrive exactly like every other form submission on the site - no separate integration needed.

**Note on the required generator size landing in Pipedrive itself:** right now only the visitor's own form fields (whatever your Pipedrive form asks for) reach Pipedrive - the calculated Total Watts/kWh/kVA aren't injected into the CRM record automatically, since that requires knowing your Pipedrive form's field keys (visible in your Pipedrive form builder) to prefill a hidden/custom field via URL parameter. If you want the calculated numbers to land inside the Pipedrive lead itself, add a field (e.g. "Calculator Result") to the Pipedrive form and share its field key.

An earlier version of this used Shopify's native `{% form 'contact' %}` instead. That was based on a wrong assumption about how this store's leads are actually collected (confirmed to be Pipedrive), and it also had a real bug: the JS never successfully intercepted the submit event, so the browser fell back to a normal full-page form submission. It's been replaced entirely by the Pipedrive embed described above.

## 2. Generator & Battery Sizing Toolkit (six standard industry calculators)

- **`sections/ggp-power-tools.liquid`** — theme section version.
- **`ggp-power-tools-calculator.liquid`** — standalone Custom Liquid paste-in version.

A separate, complementary page/section (deliberately does **not** duplicate the appliance load-list calculator above) containing six calculators commonly found on generator/battery vendor sites:

1. **kVA / kW / Amps Converter** — enter any one of kVA, kW or Amps plus power factor, phase and voltage, get the other two live.
2. **Motor Starting (Surge) Calculator** — running vs. starting kVA for a motor load, using standard DOL/Star-Delta/Soft-Starter/VFD multiplier presets. Generator sizing mistakes are usually about starting load, not running load - this is why it's separate from the converter.
3. **Fuel Consumption & Running Cost Calculator** — estimated L/hr, weekly/monthly/annual fuel cost and CO₂, from a rated kW + load % + fuel price.
4. **Cable Size & Voltage Drop Calculator** — recommends a minimum copper cable size from load current, cable length and max allowable voltage drop %, using a typical mV/A/m reference table.
5. **Battery Backup & Bank Sizing Calculator** — toggle between "how long will my battery last" and "what size bank do I need," covering voltage/DoD/inverter efficiency.
6. **Altitude & Temperature Derating Calculator** — works out the nameplate kVA you need to buy so a site at altitude/high ambient temp still delivers your required output.

The Converter, Motor Starting and Altitude/Temp Derating tools each show **live-matching Gogopower generator suggestions** underneath their result, using the exact same collection + kVA metafield lookup as the Power Usage Calculator above (see Configuration below - both tools share the same settings shape, configured independently per section/file).

**Important - these are indicative, rule-of-thumb calculators**, clearly labelled as such in the UI (fuel consumption, cable sizing and derating figures especially). They're for budgeting and first-pass sizing, not a substitute for a proper electrical/engineering sign-off - each panel that carries real safety/compliance weight (cable sizing in particular) says so directly under its result.

## Install as a theme section

This requires the file to be part of your actual theme's codebase (not just this standalone repo).

1. **Get the file into your theme**, either:
   - **One-time manual step**: Shopify Admin → Online Store → Themes → your theme's **⋯** menu → **Edit code** → in the **Sections** folder, **Add a new section**, name it `ggp-power-calculator`, paste in the contents of `sections/ggp-power-calculator.liquid`, and Save. *or*
   - **Git-managed**: if your live theme is connected to a GitHub repo (Admin → Themes → **Add theme** → **Connect from GitHub**, or ask your dev to set this up), add this file at `sections/ggp-power-calculator.liquid` in that repo. Every push then updates the theme automatically — no more copy-paste.
2. Go to the theme customizer (Admin → Online Store → Themes → **Customize**), open the page you want the calculator on, click **Add section**, and choose **GGP Power Calculator**.
3. In the section's settings panel (right sidebar), configure:
   - **Generator collection** — pick your generator collection from the dropdown (no more typing a handle by hand)
   - **kVA metafield namespace / key** — defaults to `custom` / `prime_power`
   - **Contact page link** — pick your Contact page with Shopify's link picker
   - **Power factor** / **Safety margin** — sliders, default 0.8 / 1.3
4. Save, then preview on desktop and mobile.

## Install via Custom Liquid (copy-paste)

1. Shopify Admin → Online Store → Themes → Customize
2. Add a **Custom Liquid** section/block on the page you want the calculator (e.g. a "Solar & Generator Calculator" page)
3. Open `gogopower-energy-calculator.liquid`, copy the **entire file**, paste it into the Custom Liquid block, and Save
4. Preview the page on desktop and mobile

### Configuration (Custom Liquid version only)

The section version uses the customizer's settings panel instead (see above). The Custom Liquid version's settings live in one block near the top of the file:

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

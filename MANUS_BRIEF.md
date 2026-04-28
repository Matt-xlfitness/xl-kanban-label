# Brief for Manus — XL Kanban Label

**Hi Manus.** Read this fully before writing any code.

---

## What's changed

The Kanban label is **already built and hosted live**. You no longer need to render it. Your only job is to **send data to it**.

🔗 **Live label page:** https://matt-xlfitness.github.io/xl-kanban-label/
🔗 **URL builder (for testing):** https://matt-xlfitness.github.io/xl-kanban-label/builder.html

When opened with the right query string, the label renders correctly — the design, fonts, layout, auto-fit text, QR generation and 127 × 127 mm print are all handled. **Do not rebuild any of it.**

---

## What you do

For every part on the XL Fitness site, add a **"Print Kanban Label"** button. The button opens the live label page with the part's data passed in the URL.

You have two ways to pass data:

### Option A — `?data=` URL (preferred, no backend needed)

1. Build the product JSON (see exact shape below).
2. Base64-encode the JSON.
3. Build the URL: `https://matt-xlfitness.github.io/xl-kanban-label/?data=<base64>`
4. The "Print Label" button opens that URL in a new tab.

JavaScript snippet you can drop into your codebase:

```js
function kanbanLabelUrl(product) {
  const json = JSON.stringify(product);
  const b64  = btoa(unescape(encodeURIComponent(json)));
  return `https://matt-xlfitness.github.io/xl-kanban-label/?data=${b64}`;
}

// Usage:
<a href={kanbanLabelUrl(productData)} target="_blank">Print Kanban Label</a>
```

### Option B — `?code=` + your API

If you'd rather have the label fetch from your backend:

1. Expose `GET https://your-site/api/parts/:code` returning the JSON below.
2. Tell Matt and I'll set `API_ENDPOINT` in `index.html` to your URL.
3. Buttons just link to `https://matt-xlfitness.github.io/xl-kanban-label/?code=SUPPS-0267`.

**Pick A unless you have a reason to use B.** A means zero backend wiring on your side.

---

## The exact JSON shape

```json
{
  "code": "SUPPS-0267",
  "description": "Amino Energy Sparkling - 355ml - Blueberry Lemonade",
  "rows": {
    "a3":  { "label": "TYPE",         "value": "Energy Drinks" },
    "a4":  { "label": "VARIANT",      "value": "Blueberry Lemonade" },
    "a5":  { "label": "BRAND",        "value": "Optimum Nutrition" },
    "a6":  { "label": "",             "value": "" },
    "a7":  { "label": "",             "value": "" },
    "a8":  { "label": "",             "value": "" },
    "a9":  { "label": "",             "value": "" },
    "a10": { "label": "SUPPLIER",     "value": "FitnessVending" },
    "a11": { "label": "XL PART ID",   "value": "SUPPS-0267" },
    "a12": { "label": "SUPPLIER SKU", "value": "12345678" },
    "a1":  { "label": "Q to Order",   "value": "1" },
    "a2":  { "label": "Box Size",     "value": "12" }
  },
  "image":     "https://your-cdn.com/parts/SUPPS-0267.png",
  "qrPayload": "https://xlfitness.com.au/parts/SUPPS-0267",
  "color":     "#121826"
}
```

### Field rules

| Key | Required | Notes |
|---|---|---|
| `code` | yes | Renders in the top header bar |
| `description` | yes | Renders in the second header bar |
| `rows.a3..a12` | yes (slots) | Spec rows. Both `label` AND `value` empty → row is hidden |
| `rows.a1` | yes | Bottom-left big box: Q to Order |
| `rows.a2` | yes | Bottom-left big box: Box Size |
| `image` | optional | URL; if missing, `<IMAGE>` placeholder shown |
| `qrPayload` | optional | String encoded into the QR code; defaults to `code` if missing |
| `color` | optional | Hex; defaults to navy `#121826`. Future per-product colour-coding hook |

---

## Hard rules — DO / DO NOT

✅ **DO**
- Map your site's fields to the JSON keys exactly per the table below.
- Leave `label` AND `value` as empty strings (`""`) for rows with no data — the label page hides them automatically.
- Always populate `a1` (Q to Order) and `a2` (Box Size).
- Test with the URL builder before integrating: https://matt-xlfitness.github.io/xl-kanban-label/builder.html

❌ **DO NOT**
- **Do not rebuild the label HTML/CSS.** Use the live page.
- **Do not** put placeholder strings like `"Attribute 6"`, `"Attribute 7"` in empty rows. Use `""`.
- **Do not** put Q to Order or Box Size in `a3..a12`. They go in `a1` / `a2`.
- **Do not** put prices or quantities in the `Brand` / `Supplier` rows.
- **Do not** strip print CSS or change the page size.

---

## Field mapping — your site → JSON

| Your site's field | Maps to JSON key |
|---|---|
| Part code (SUPPS-…) | `code` and `rows.a11.value` |
| Product name | `description` |
| Type / Category | `rows.a3` (label `"TYPE"`) |
| Variant / Flavour | `rows.a4` (label `"VARIANT"`) |
| Brand | `rows.a5` (label `"BRAND"`) |
| (extra attributes 6–9 if any) | `rows.a6..a9` |
| Supplier name | `rows.a10` (label `"SUPPLIER"`) |
| Supplier SKU / barcode | `rows.a12` (label `"SUPPLIER SKU"`) |
| Q to Order (per-part value) | `rows.a1` (label `"Q to Order"`) |
| Box Size (units per box) | `rows.a2` (label `"Box Size"`) |
| Product image URL | `image` |
| Part page URL on XL site | `qrPayload` |
| Card accent colour (per-category) | `color` |

---

## What the last attempt got wrong (so you don't repeat it)

Looking at the previous render of `SUPPS-0267`:

| Symptom | Cause |
|---|---|
| Title cut off ("…Blueberry Lemo") | Auto-fit script wasn't running. Won't happen if you use the live page — the script is built in. |
| "Q to Order" with value "Energy Drinks" | Field mapping wrong: `a1` got TYPE data instead. |
| "Brand" showing "$3.30" | Price was put in the Brand row. Brand is the brand name string. |
| "Attribute 6/7/8" boxes visible | Empty rows had placeholder labels. Use `""` for both `label` and `value`. |
| Image and QR overflowed the card border | Layout was redrawn. Won't happen if you use the live page. |
| Bottom rows in wrong place | Q to Order and Box Size were stuffed into `a3`/`a4` instead of `a1`/`a2`. |

Fix: stop rebuilding. Use the live page + send data via the URL.

---

## Confirmation checklist before you ship

- [ ] Each product page has a "Print Kanban Label" button
- [ ] The button opens `https://matt-xlfitness.github.io/xl-kanban-label/?data=…` in a new tab
- [ ] Empty rows are sent as `{ "label": "", "value": "" }` — never `"Attribute N"`
- [ ] `a1` and `a2` contain Q to Order and Box Size (not category data)
- [ ] Opening the link for `SUPPS-0267` shows the energy drink card with no clipped text and no overflowing boxes
- [ ] Print preview shows a 127 × 127 mm page with solid navy bars (not light grey)

If any of these fail, the bug is in your data — not the label.

---

## Questions

If the data on your site doesn't perfectly match a field, transform it client-side before encoding. Don't change the live label page. Ask Matt if you're unsure.

— Matt

# Brief for Manus — XL Kanban Label Integration

**Hi Manus.** Please read this in full before writing any code.

---

## TL;DR — what changed

You no longer need to render the label yourself. The label is **already built** as a self-contained HTML page (`product-label.html`). Your previous attempt mis-mapped the data into the wrong rows, used "Attribute 6/7/8" placeholders, and broke the print layout.

**Your new, smaller job:** expose the product data in the exact JSON format below, and link to the label page from each product. The label page does all rendering, auto-fitting, QR generation, and printing — leave that alone.

---

## What you must do

### 1. Expose product data at a JSON endpoint

For every XL Part ID on the site, there must be an endpoint that returns the product as JSON:

```
GET /api/parts/SUPPS-0267
```

Response body — **must** match this shape exactly:

```json
{
  "code": "SUPPS-0267",
  "description": "Amino Energy Sparkling - 355ml - Blueberry Lemonade",
  "rows": {
    "a3":  { "label": "TYPE",            "value": "Energy Drinks" },
    "a4":  { "label": "VARIANT",         "value": "Blueberry Lemonade" },
    "a5":  { "label": "BRAND",           "value": "Optimum Nutrition" },
    "a6":  { "label": "",                "value": "" },
    "a7":  { "label": "",                "value": "" },
    "a8":  { "label": "",                "value": "" },
    "a9":  { "label": "",                "value": "" },
    "a10": { "label": "SUPPLIER",        "value": "FitnessVending" },
    "a11": { "label": "XL PART ID",      "value": "SUPPS-0267" },
    "a12": { "label": "SUPPLIER SKU",    "value": "XXXXXXXX" },
    "a1":  { "label": "Q to Order",      "value": "1" },
    "a2":  { "label": "Box Size",        "value": "12" }
  },
  "image":     "https://your-cdn.com/parts/SUPPS-0267.png",
  "qrPayload": "https://xlfitness.com.au/parts/SUPPS-0267",
  "color":     "#121826"
}
```

### 2. Add a "Print Kanban Label" button

On each product page, add a button that opens the label page in a new tab with the part code in the query string:

```html
<a href="/labels/index.html?code=SUPPS-0267" target="_blank">
  Print Kanban Label
</a>
```

The label page (which I will update) will:
1. Read `?code=SUPPS-0267` from the URL
2. Call `/api/parts/SUPPS-0267`
3. Render the label
4. User hits print → 127 × 127 mm card

That's the whole flow. **You do not render the label. You provide the data + the link.**

---

## Exact field mapping — do not deviate

| Your site's field | Maps to JSON key | Goes into label slot |
|---|---|---|
| Part code (SUPPS-…) | `code` | Top header bar (large text) |
| Product name | `description` | Second header bar |
| Type / Category | `rows.a3.value` | Spec row 1 |
| Variant / Flavour | `rows.a4.value` | Spec row 2 |
| Brand | `rows.a5.value` | Spec row 3 |
| (extra attributes if any) | `rows.a6..a9.value` | Spec rows 4–7 |
| Supplier name | `rows.a10.value` | Spec row 8 |
| XL Part ID (same as code) | `rows.a11.value` | Spec row 9 |
| Supplier SKU | `rows.a12.value` | Spec row 10 |
| Q to Order | `rows.a1.value` | Bottom row 1 (large box) |
| Box Size | `rows.a2.value` | Bottom row 2 (large box) |
| Product image URL | `image` | Right side image area |
| URL to encode in QR | `qrPayload` | QR code on bottom right |
| Card colour (hex) | `color` | Header bar / accent colour (default `#121826`) |

The corresponding `label` field in each row must be the human-readable label shown on the card (`TYPE`, `VARIANT`, `BRAND`, `SUPPLIER`, etc.).

---

## Hard rules

✅ **DO**
- Always populate `a3`, `a4`, `a5`, `a10`, `a11`, `a12`, `a1`, `a2` if data exists.
- Leave both `label` AND `value` as empty strings (`""`) for any row that has no data — the label page automatically hides empty rows so the card stays clean.
- Provide an `image` URL when one exists; the label gracefully falls back to a placeholder if it's missing.
- Use the actual price/box-size/supplier values in the correct rows.

❌ **DO NOT**
- Do **not** put placeholder text like `"Attribute 6"`, `"Attribute 7"`, etc. in empty rows. Leave them as `""`.
- Do **not** put Q to Order or Box Size in the spec area (`a3..a12`). They belong in `a1` and `a2`, which render as the big bottom rows.
- Do **not** put prices ($39.60, $3.30) in the Brand or Box Size rows — those are wrong fields.
- Do **not** rebuild the label HTML/CSS yourself. Use the file I'm sending you.
- Do **not** strip the `print-color-adjust` CSS or change the `@page { size: 127mm 127mm }` rule — they exist to make the print come out right.

---

## What was broken in the last attempt (so you don't repeat it)

Looking at the last render of `SUPPS-0267`:

| Symptom | Cause |
|---|---|
| Title cut off ("…Blueberry Lemo") | Auto-fit script wasn't running — happens because you rebuilt the label instead of using the source file. |
| "Q to Order" with value "Energy Drinks" | Field mapping wrong — `a1` got TYPE data instead of Q to Order. |
| "Brand" showing "$3.30" | Price was placed in the Brand row. |
| "Attribute 6/7/8" boxes visible | Empty rows showed placeholder labels instead of being hidden. |
| Image and QR overflowed the card border | Layout proportions changed when rebuilt — the source file already has the correct proportions. |
| Bottom rows ("Q to Order" / "Box Size") in wrong place | They belong in two big boxes at the bottom, not in the top spec area. |

Fix: stop rebuilding. Just provide the JSON + link to the source label page.

---

## Confirmation checklist before you say it's done

- [ ] `/api/parts/SUPPS-0267` returns the JSON above (or close to it with real values)
- [ ] The product page has a "Print Kanban Label" button/link pointing to `/labels/index.html?code=…`
- [ ] Empty rows are returned as `{ "label": "", "value": "" }` — never as `"Attribute 6"`
- [ ] `a1` and `a2` contain Q to Order and Box Size (NOT type/brand data)
- [ ] When the label page is opened with `?code=SUPPS-0267`, you see the energy drink card render correctly with no clipped text and no overflowing boxes
- [ ] Print preview shows a 127 × 127 mm page with solid navy bars (not light grey)

If any of these fail, the fix is in your data layer — not in the label HTML.

---

## Files you have

- `product-label.html` — the label rendering page. Host this as a static file (e.g. at `/labels/index.html`). I will update it to call your `/api/parts/:code` endpoint instead of using the local stub.
- `INTEGRATION.md` — full technical spec (use as reference)
- This brief — the rules

If you have questions about the data shape, **ask before guessing**. Do not invent placeholders.

— Matt

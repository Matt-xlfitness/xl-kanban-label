# XL Kanban Label — Integration Brief

**For:** the AI / engineer integrating this label into the XL Fitness site (Manus).
**Reference file:** [`product-label.html`](./product-label.html)

---

## What this is

A **printable Kanban inventory card**, 127 × 127 mm, for XL Fitness parts. Each card carries the part code, description, attributes, image, QR code, and reorder quantities. Staff scan the QR to reorder when stock is low.

The reference HTML is **fully self-contained** — open it in any browser, hit *Print 127×127mm*, you get a finished label.

## What you need to build

1. A page on the XL Fitness site that:
   - Accepts an XL Part ID via URL (e.g. `/label?code=SUPPS-0356`)
   - Looks up the part's data from the existing site database/API
   - Renders the label using **the exact visual style in `product-label.html`**
   - Has a Print button → prints at 127 × 127 mm

2. Match the visual style **pixel-for-pixel**. The HTML is the source of truth.

## Data contract

Per part, you need to provide:

```json
{
  "code": "SUPPS-0356",
  "description": "Gatorade NO SUGAR - Berry",
  "rows": {
    "a3":  { "label": "TYPE",         "value": "Drinks, Hydration" },
    "a4":  { "label": "VARIANT",      "value": "No Sugar - Berry" },
    "a5":  { "label": "BRAND",        "value": "Gatorade" },
    "a6":  { "label": "",             "value": "" },
    "a7":  { "label": "",             "value": "" },
    "a8":  { "label": "",             "value": "" },
    "a9":  { "label": "",             "value": "" },
    "a10": { "label": "SUPPLIER",     "value": "FitnessVending" },
    "a11": { "label": "XL PART ID",   "value": "SUPPS-0356" },
    "a12": { "label": "SUPPLIER SKU", "value": "2875839578208" },
    "a1":  { "label": "Q to Order",   "value": "1" },
    "a2":  { "label": "Box Size",     "value": "12" }
  },
  "image":     "https://xlfitness.com.au/images/parts/SUPPS-0356.png",
  "qrPayload": "https://xlfitness.com.au/parts/SUPPS-0356",
  "color":     "#121826"
}
```

### Field rules

| Field | Required | Notes |
|---|---|---|
| `code` | yes | Goes in the top header bar |
| `description` | yes | Goes in the second header bar |
| `rows.a3..a12` | yes (slots) | Spec rows. Both `label` AND `value` empty → row hides |
| `rows.a1` | yes | Bottom: Q to Order |
| `rows.a2` | yes | Bottom: Box Size |
| `image` | optional | URL; if missing, `<IMAGE>` placeholder shown |
| `qrPayload` | optional | String encoded in QR; defaults to part code if missing |
| `color` | optional | Hex; defaults to `#121826` (navy). Future: per-category colour-coding |

## Integration point

In `product-label.html`, find the function `loadProduct(code)`:

```js
async function loadProduct(code) {
  return PRODUCTS[code] || null;   // ← stub, replace this
}
```

Replace with a real fetch:

```js
async function loadProduct(code) {
  const res = await fetch(`/api/parts/${encodeURIComponent(code)}`);
  if (!res.ok) return null;
  return await res.json();   // must return the shape above
}
```

Everything else (rendering, auto-fit, QR generation, printing) already works.

## Critical visual specs

These are pulled from Figma (`ePmTOphFK3IFYsh7X78hcs`, node `1:4`) — do not deviate:

| Spec | Value |
|---|---|
| Card size | 127 × 127 mm (square) |
| Outer border | 16 px black (visually 1.92cqw), 30 px radius |
| Navy fill | `#121826` |
| Font | **Lexend ExtraBold (800)** — Google Fonts |
| Header bars | Navy bg, white text, 10 px radius |
| Spec rows | 10 rows, navy label cell + white value cell, 3 px black border |
| Bottom rows | Q to Order + Box Size, 88 px tall |
| Image rect | 371 × 371 (Figma units), 1 px black border |
| QR rect | 275 × 275 (Figma units), 1 px black border |

Internally the layout uses `%` and `cqw` units against a 127 mm container — it scales perfectly to any container size. Don't switch to fixed pixels.

## Printing — non-obvious gotchas

1. **`print-color-adjust: exact`** is set globally. Without it, browsers strip the dark navy fills to save ink → labels print as light grey.
2. **Browser dialog setting:** Chrome/Edge/Safari ship with *Background graphics* OFF by default. The user must tick it ON. Surface this in your UI if you can ("Tip: enable Background graphics in the print dialog").
3. **Page setup:**
   - Paper size: **127 × 127 mm** (custom)
   - Margins: **None**
   - Scale: **100%** — *not* "Fit to page" (this distorts the 127 mm dimension)

## Auto-fit text

Long values (like a 13-digit SKU or `SUPPLIER SKU` label) auto-shrink to fit one line. The `fitText()` function reduces font-size 0.5 px at a time until `scrollWidth ≤ clientWidth`. Re-runs on: load, font-load, resize, and `beforeprint`. Keep this script as-is.

## Future: variable card colour

The `color` field is in the schema but not yet wired. When the colour-coding rule is finalised (by category / supplier / urgency), the integration should:

1. Read `color` from the product data
2. Apply it inline: `label.style.setProperty('--navy', color)`
3. The whole card updates — header bars, label cells, all use `var(--navy)`

---

## Questions? Mismatches?

If the API returns a different shape, transform it client-side to match the contract above before passing to `render()`. Don't change the rendering code — keep the visual output identical to the reference HTML.

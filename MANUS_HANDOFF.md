# Hi Manus — Kanban Label Handoff

Quick context change so we stop going in circles.

The Kanban label has been **fully built and deployed as a live page**. Don't rebuild it, restyle it, or copy the HTML into your codebase. **Just send data to it.**

---

## The two URLs you need

- **Live label page:** https://matt-xlfitness.github.io/xl-kanban-label/
- **Test it yourself:** https://matt-xlfitness.github.io/xl-kanban-label/builder.html

Open the builder, paste sample product JSON, hit *Build & Open*. You'll see the label render. That's what your "Print Label" button on each product page should do — open the live page with the part's data in the URL.

---

## What I need from you (3 things)

### 1. Add a "Print Kanban Label" button to every product page

It links to the live label page with the part's JSON base64-encoded into the `?data=` query string.

Drop-in helper:

```js
function kanbanLabelUrl(product) {
  const json = JSON.stringify(product);
  const b64  = btoa(unescape(encodeURIComponent(json)));
  return `https://matt-xlfitness.github.io/xl-kanban-label/?data=${b64}`;
}
```

Then in your product page template:

```html
<a href={kanbanLabelUrl(productData)} target="_blank" class="btn">
  Print Kanban Label
</a>
```

### 2. Build `productData` from your existing fields using this exact shape

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

### 3. Follow these mapping + behaviour rules exactly

**Mapping (your field → JSON key):**

| Your site's field | JSON key |
|---|---|
| Part code (SUPPS-…) | `code`, also `rows.a11.value` |
| Product name / title | `description` |
| Type / Category | `rows.a3.value` |
| Variant / Flavour | `rows.a4.value` |
| Brand | `rows.a5.value` |
| (extra attributes, if any) | `rows.a6` to `rows.a9` |
| Supplier name | `rows.a10.value` |
| Supplier SKU / barcode | `rows.a12.value` |
| Q to Order (per part) | `rows.a1.value` |
| Box Size (units per box) | `rows.a2.value` |
| Product image URL | `image` |
| Part page URL on the XL site | `qrPayload` |
| Per-product accent colour | `color` (hex; defaults to `#121826`) |

**Behaviour rules — please follow these strictly:**

- ✅ Empty rows must be sent as `{ "label": "", "value": "" }`. The label page hides them automatically. **Do not** put placeholder strings like `"Attribute 6"`.
- ✅ `a1` and `a2` are reserved for **Q to Order** and **Box Size** — they render in the big bottom boxes. Don't put TYPE / BRAND / price data in them.
- ✅ Test each product first by opening the live URL and visually checking — no clipped text, no overflowing boxes, navy bars solid, QR code present.
- ❌ Do not modify or rehost the label HTML. The styling, fonts, auto-fit, QR generation, and 127 × 127 mm print are all handled by the live page.

---

## Testing the connection

For your first integration test:

1. Pick one product on the site (use SUPPS-0267 if you still have its data loaded).
2. Build the JSON per the shape above.
3. Use the helper to build the URL.
4. Open it in a new tab.
5. **It should look exactly like a finished Kanban card** — title across the top, all rows correctly labelled and populated, image on the right, QR below it, Q to Order / Box Size in the big bottom boxes. No clipped text. No "Attribute 6" boxes.

If it doesn't, the bug is in your JSON — fix the data, don't touch the label. Compare against the URL builder's sample output as ground truth.

---

## When you're done, send me back

- The URL of one finished product page that has the working button
- A screenshot of the rendered label opened from that button

That's the whole job. Let me know if anything in the JSON shape doesn't map cleanly to your existing fields and I'll help.

— Matt

---

**Full reference (only if you need deeper detail):** https://github.com/Matt-xlfitness/xl-kanban-label/blob/main/MANUS_BRIEF.md

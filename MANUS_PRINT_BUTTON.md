# Manus — Add a Print Label Button (concise brief)

**Hi Manus.** Read all of this before writing code. It's short.

---

## What I want

For every item on the XL Fitness site, add a **"Print Label"** button.

The button opens an external print-label service in a new tab, with the item's data passed in the URL.

## ⛔ Hard rules — read this first, do not skip

The URL `https://matt-xlfitness.github.io/xl-kanban-label/...` is an **opaque external service**. Treat it like a third-party API endpoint.

- ❌ **Do NOT visit, preview, screenshot, or render the URL** in your reasoning steps.
- ❌ **Do NOT iframe, embed, or proxy** the URL inside the Manus site.
- ❌ **Do NOT recreate, restyle, redesign, or "improve"** what's at the URL.
- ❌ **Do NOT add the URL to your design system, component library, or theme.**
- ❌ **Do NOT round corners, change colours, or adjust spacing** to "match" the Manus site.
- ✅ **DO** treat the URL as a string. Just build the string and put it in `<a href="…">`. Nothing more.

The print page already has its own design, fonts, layout, colours, and print rules. **Touching any of it = breaking it.** Your only job is to construct the URL.

---

## What to do (3 steps)

### 1. Drop in this helper

```js
function makePrintLabelHref(product) {
  const json = JSON.stringify(product);
  const b64  = btoa(unescape(encodeURIComponent(json)));
  return "https://matt-xlfitness.github.io/xl-kanban-label/?data=" + b64;
}
```

### 2. On each product page, build the `product` object from the page's data

The exact shape — copy this template, fill in the values from the item:

```js
const product = {
  code:        item.xlCode,                   // e.g. "SUPPS-0356"
  description: item.name,                     // e.g. "Gatorade NO SUGAR - Berry"
  rows: {
    a3:  { label: "TYPE",         value: item.type        ?? "" },
    a4:  { label: "VARIANT",      value: item.variant     ?? "" },
    a5:  { label: "BRAND",        value: item.brand       ?? "" },
    a6:  { label: item.a6Label   ?? "", value: item.a6   ?? "" },
    a7:  { label: item.a7Label   ?? "", value: item.a7   ?? "" },
    a8:  { label: item.a8Label   ?? "", value: item.a8   ?? "" },
    a9:  { label: item.a9Label   ?? "", value: item.a9   ?? "" },
    a10: { label: "SUPPLIER",     value: item.supplier    ?? "" },
    a11: { label: "XL PART ID",   value: item.xlCode      ?? "" },
    a12: { label: "SUPPLIER SKU", value: item.supplierSku ?? "" },
    a1:  { label: "Q to Order",   value: String(item.qToOrder ?? "") },
    a2:  { label: "Box Size",     value: String(item.boxSize  ?? "") }
  },
  image:     item.imageUrl ?? "",             // optional product image URL
  qrPayload: item.publicUrl ?? "",            // URL to encode in QR (e.g. the item's page on the site)
  color:     item.color ?? "#121826"          // hex; controls the dark blue accent
};
```

Adjust the right-hand side (`item.xxx`) to match whatever your data fields are actually called.

### 3. Render the button

```jsx
<a
  href={makePrintLabelHref(product)}
  target="_blank"
  rel="noopener"
  className="btn"
>
  Print Label
</a>
```

That's the entire job.

---

## Hard rules on the JSON data itself

- ✅ Empty A-slots: send `{ "label": "", "value": "" }` — both as empty strings. The print service hides the row automatically.
- ❌ Do NOT use placeholder strings like `"Attribute 6"`, `"N/A"`, or `"-"` for empty rows. They will render as visible empty boxes and look broken.
- ✅ `a1` is **Q to Order**. `a2` is **Box Size**. They render as the two big rows at the bottom.
- ❌ Do NOT put TYPE / BRAND / VARIANT data in `a1` or `a2`. They go in `a3`–`a5`.
- ✅ `a11` value should equal `code` (the XL Part ID is shown both in the top header and in row a11).
- ✅ `qrPayload` should be the URL of the item's page on the Manus site so scanning the QR opens that item.
- ✅ `color` is hex like `"#7a1f3d"`. Defaults to dark navy if missing or invalid.

---

## How to test

1. Pick one item (e.g. `SUPPS-0356`).
2. Add the button.
3. Click it. A new tab opens.
4. Confirm visually:
   - Top bar shows the code
   - Second bar shows the description
   - All filled rows show the right label + value
   - Empty rows are not rendered (no visible empty boxes)
   - Image and QR appear on the right
   - The dark blue accent matches the item's `color`
   - Q to Order and Box Size are in the **two big bottom rows** (not in the spec area)

If anything is wrong, **the bug is in your `product` object, not in the print page.** Fix the data, do not touch the URL or attempt to "fix" the print design.

---

## Done = this

- Every product page on the XL Fitness site has a working "Print Label" button.
- Clicking it opens the external print page with that item's data correctly populated.
- You did not visit, preview, restyle, iframe, or modify the print URL.

Send back: one screenshot of the working button on a product page + the URL of that product page. Done.

— Matt

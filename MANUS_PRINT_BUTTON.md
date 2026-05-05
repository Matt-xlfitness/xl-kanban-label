# Manus — Add a Print Label Button (concise brief)

**Hi Manus.** Read all of this before writing code. It's short and exact.

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
- ✅ **DO** treat the URL as a string. Just build the string and put it in `<a href="…">`.

The print page already has its own design, fonts, layout, colours, and print rules. **Touching any of it = breaking it.** Your only job is to construct the URL.

---

## ⚠️ The bug we keep hitting — read this carefully

Your form has fields numbered `1`–`9` plus three tagged "A10/A11/A12" plus two tagged "A1/A2". **The 1–9 numbers are display positions on the form, NOT A-codes.** You must route every field to its correct A-slot by the field's NAME, not by its display position.

Specifically:
- **`Q to order`** is field number `1` on your form, but it goes in **`a1`** (the bottom big row), NOT `a3`.
- **`Box Size`** is field number `2` on your form, but it goes in **`a2`** (the bottom big row), NOT `a4`.
- **`Item Type`** is field number `3` on your form, but it goes in **`a3`** with the canonical label `"TYPE"` (uppercase), NOT `"Item Type"`.

If you map fields by their position number you will get every label wrong. Use the table below.

---

## The mapping table — single source of truth

| Manus form field | A-slot (JSON key) | Canonical label on card |
|---|---|---|
| `Q to order` (badge `A1 = Q to Order`) | **`a1`** | `Q to Order` |
| `Box Size` (badge `A2 = Box Size`) | **`a2`** | `Box Size` |
| `Item Type` | `a3` | **`TYPE`** |
| `Flavour / Variant` | `a4` | **`VARIANT`** |
| `Brand` | `a5` | **`BRAND`** |
| `Attribute 6` | `a6` | (custom label entered by user — see note below) |
| `Attribute 7` | `a7` | (custom label entered by user) |
| `Attribute 8` | `a8` | (custom label entered by user) |
| `Attribute 9` | `a9` | (custom label entered by user) |
| `A10 — Supplier` (badge) | `a10` | `SUPPLIER` |
| `A11 — XL Part ID` (badge) | `a11` | `XL PART ID` |
| `A12 — Supplier SKU` (badge) | `a12` | `SUPPLIER SKU` |

### Note on Attributes 6–9

These four slots are for **custom item-specific attributes**. Each needs **two inputs on Manus's form**: a **Label** (e.g. `MATERIAL`) and a **Value** (e.g. `Stainless Steel`). If your form currently has only a value input for these slots, please add a label input alongside it. If both label AND value are blank, the row hides automatically on the card — that is the desired behaviour for items with fewer than 9 attributes.

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

### 2. On each product page, build the `product` object from the form data

Use **this exact template**. Read each field by its name from the form, route to its A-slot per the table above, hard-code the canonical label.

```js
const product = {
  code:        item.xlPartId,           // SUPPS-0254
  description: item.name,               // CBUM Thavage PUMP RTD 355ml Juicy Pumps
  rows: {
    // Top spec rows — fixed canonical headings
    a3:  { label: "TYPE",         value: item.itemType        ?? "" },
    a4:  { label: "VARIANT",      value: item.flavourVariant  ?? "" },
    a5:  { label: "BRAND",        value: item.brand           ?? "" },

    // Custom attribute rows — both label AND value come from the form
    a6:  { label: item.attr6Label ?? "", value: item.attr6Value ?? "" },
    a7:  { label: item.attr7Label ?? "", value: item.attr7Value ?? "" },
    a8:  { label: item.attr8Label ?? "", value: item.attr8Value ?? "" },
    a9:  { label: item.attr9Label ?? "", value: item.attr9Value ?? "" },

    // Fixed standard rows
    a10: { label: "SUPPLIER",     value: item.supplier        ?? "" },
    a11: { label: "XL PART ID",   value: item.xlPartId        ?? "" },
    a12: { label: "SUPPLIER SKU", value: item.supplierSku     ?? "" },

    // Bottom big rows — Q to Order and Box Size go HERE, not in a3/a4
    a1:  { label: "Q to Order",   value: String(item.qToOrder ?? "") },
    a2:  { label: "Box Size",     value: String(item.boxSize  ?? "") }
  },
  image:     item.imageUrl   ?? "",     // optional product image URL
  qrPayload: item.publicUrl  ?? "",     // URL to encode in QR (item's page on the XL site)
  color:     item.color      ?? "#121826"
};
```

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

## Ground-truth target — SUPPS-0254

For SUPPS-0254 (CBUM Thavage Juicy Pumps), the JSON inside `?data=` MUST decode to **exactly this**:

```json
{
  "code": "SUPPS-0254",
  "description": "CBUM Thavage PUMP RTD 355ml Juicy Pumps",
  "rows": {
    "a3":  { "label": "TYPE",         "value": "Drinks, Energy" },
    "a4":  { "label": "VARIANT",      "value": "Juicy Pumps" },
    "a5":  { "label": "BRAND",        "value": "CBUM Thavage" },
    "a6":  { "label": "",             "value": "" },
    "a7":  { "label": "",             "value": "" },
    "a8":  { "label": "",             "value": "" },
    "a9":  { "label": "",             "value": "" },
    "a10": { "label": "SUPPLIER",     "value": "FitnessVending" },
    "a11": { "label": "XL PART ID",   "value": "SUPPS-0254" },
    "a12": { "label": "SUPPLIER SKU", "value": "3481611960416" },
    "a1":  { "label": "Q to Order",   "value": "1" },
    "a2":  { "label": "Box Size",     "value": "12" }
  },
  "image":     "<image URL>",
  "qrPayload": "<XL site item URL>",
  "color":     "#121826"
}
```

If your generated JSON for SUPPS-0254 doesn't match this shape exactly, the button is wrong. Fix the data, do not touch the print URL.

---

## Hard rules on the data itself

- ✅ Empty A-slots: send `{ "label": "", "value": "" }` — both empty strings. The print page auto-hides the row.
- ❌ Do NOT use placeholder strings like `"Attribute 6"`, `"N/A"`, `"-"`, or `"unset"` for empty slots — they will render as visible empty boxes.
- ✅ `a1` is **always** Q to Order. `a2` is **always** Box Size. They render as the two big rows at the bottom.
- ❌ Do NOT put TYPE / VARIANT / BRAND / Q to Order / Box Size in the wrong A-slot, even if the form lays them out in a different order visually.
- ✅ `a11.value` should equal `code` (XL Part ID is shown twice — top header and row a11).
- ✅ `qrPayload` should be the URL of the item's page on the Manus site so scanning the QR opens that item.
- ✅ `color` is hex like `"#7a1f3d"`. Defaults to `#121826` if missing or invalid.

---

## How to test

1. Pick **SUPPS-0254** as your first test.
2. Add the button.
3. Click it. A new tab opens.
4. Confirm visually:
   - Top header: `SUPPS-0254`
   - Description bar: `CBUM Thavage PUMP RTD 355ml Juicy Pumps`
   - **First spec row**: `TYPE` / `Drinks, Energy` (NOT `Q to order` / `1`)
   - **Second spec row**: `VARIANT` / `Juicy Pumps`
   - **Third spec row**: `BRAND` / `CBUM Thavage`
   - Empty rows (a6–a9): no visible boxes, just a gap
   - Then `SUPPLIER` / `FitnessVending`
   - Then `XL PART ID` / `SUPPS-0254`
   - Then `SUPPLIER SKU` / `3481611960416`
   - **Bottom big row 1**: `Q to Order` / `1`
   - **Bottom big row 2**: `Box Size` / `12`
   - Image and QR on the right
5. Print preview should show a 127 × 127 mm page with solid navy bars.

If anything's wrong: **the bug is in your `product` object, not in the print page.** Fix the data.

---

## Done = this

- Every product page has a working "Print Label" button.
- Clicking it opens the external print page with that item's data correctly populated.
- SUPPS-0254 specifically renders matching the ground-truth JSON above.
- You did not visit, preview, restyle, iframe, or modify the print URL.

Send back: a screenshot of the rendered SUPPS-0254 label opened from the button + the URL of the SUPPS-0254 product page.

— Matt

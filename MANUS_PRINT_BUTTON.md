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

A3–A9 are **PER-CATEGORY**, not fixed across the whole site.

You have category schemas (SUPPS, BLT, PAPP, MERCH, CONS, etc.) — each one defines its own labels for slots A3–A9 AND its own card colour. **Item PAPP-0010 must use the PAPP category's labels and colour, NOT the SUPPS labels.**

Currently you're using SUPPS labels (`Item Type`, `Flavour / Variant`, `Brand`) for items in every category. That's the bug.

The rule:

- **A1, A2, A10, A11, A12** — labels are FIXED across all categories.
- **A3–A9** — labels are LOOKED UP from the item's category schema.
- **`color`** — LOOKED UP from the item's category (e.g. SUPPS = navy, BLT = blue, PAPP = pink).
- The CODE PREFIX (e.g. `PAPP` in `PAPP-0010`) tells you which category an item belongs to.

If you hardcode A3–A9 labels you will get every non-SUPPS item wrong.

---

## The mapping table — single source of truth

### Standard slots (same for every category)

| A-slot | Label on card | Source |
|---|---|---|
| `a1` | `Q to Order` | Item's `Q to order` field |
| `a2` | `Box Size` | Item's `Box Size` field |
| `a10` | `SUPPLIER` | Item's `Supplier` field |
| `a11` | `XL PART ID` | Item's code (e.g. `PAPP-0010`) |
| `a12` | `SUPPLIER SKU` | Item's `Supplier SKU` field |

### Custom slots (per category)

| A-slot | Label on card | Value on card |
|---|---|---|
| `a3` | `category.a3Label` (e.g. `Item Type` for SUPPS, `Size (Diameter)` for BLT, `Size` for PAPP) | Item's value for that field |
| `a4` | `category.a4Label` (e.g. `Flavour / Variant` for SUPPS, `Length` for BLT, `Type` for PAPP) | Item's value |
| `a5` | `category.a5Label` (e.g. `Brand` for SUPPS, `Grade` for BLT, blank for PAPP) | Item's value |
| `a6` | `category.a6Label` (e.g. `Material` for BLT, blank for SUPPS) | Item's value |
| `a7` | `category.a7Label` (e.g. `Finish / Coating` for BLT) | Item's value |
| `a8` | `category.a8Label` (e.g. `Head Type` for BLT) | Item's value |
| `a9` | `category.a9Label` (e.g. `Thread Pitch` for BLT) | Item's value |

If a category doesn't define a label for a slot (e.g. PAPP has no A5–A9), send both label AND value as `""` — the row hides automatically on the card.

### Card colour (per category)

The `color` hex on each printed card comes from the item's category, not from the item itself. Make sure each category in your schema has a `color` field (hex string like `#121826`, `#1f6feb`, `#f7c8d9`). If a category doesn't have one set, default to `#121826` (navy).

If your category schema doesn't yet expose a colour picker per category, add one as part of this fix. One colour per category, applied to every item in that category.

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

### 2. On each product page, build the `product` object using the item AND its category

Use **this exact template**. The labels for A3–A9 and the `color` come from the **category**. Values come from the item.

```js
function buildProduct(item) {
  // 1. Find the category for this item.
  //    The code prefix (SUPPS, BLT, PAPP, MERCH, CONS, ...) is the lookup key.
  const categoryCode = item.code.split("-")[0];          // e.g. "PAPP" from "PAPP-0010"
  const cat = getCategoryByCode(categoryCode);            // your existing category lookup

  return {
    code:        item.code,                               // SUPPS-0254 / BLT-0033 / PAPP-0010
    description: item.name,
    rows: {
      // ── Standard slots — same labels for every category ─────────────
      a1:  { label: "Q to Order",   value: String(item.qToOrder    ?? "") },
      a2:  { label: "Box Size",     value: String(item.boxSize     ?? "") },
      a10: { label: "SUPPLIER",     value: item.supplier           ?? ""  },
      a11: { label: "XL PART ID",   value: item.code               ?? ""  },
      a12: { label: "SUPPLIER SKU", value: item.supplierSku        ?? ""  },

      // ── Custom slots — label from CATEGORY, value from ITEM ─────────
      a3:  { label: cat.a3Label ?? "", value: item.a3 ?? "" },
      a4:  { label: cat.a4Label ?? "", value: item.a4 ?? "" },
      a5:  { label: cat.a5Label ?? "", value: item.a5 ?? "" },
      a6:  { label: cat.a6Label ?? "", value: item.a6 ?? "" },
      a7:  { label: cat.a7Label ?? "", value: item.a7 ?? "" },
      a8:  { label: cat.a8Label ?? "", value: item.a8 ?? "" },
      a9:  { label: cat.a9Label ?? "", value: item.a9 ?? "" }
    },
    image:     item.imageUrl  ?? "",
    qrPayload: item.publicUrl ?? "",
    color:     cat.color      ?? "#121826"               // ← from CATEGORY
  };
}
```

**Critical:** the `cat.aNLabel` strings come from your category schema, NOT from a hardcoded list. PAPP returns "Size" / "Type"; BLT returns "Size (Diameter)" / "Length" / "Grade" / "Material" / etc.; SUPPS returns "Item Type" / "Flavour / Variant" / "Brand". Whatever the category defines is what gets sent.

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

## Worked examples — three categories

### Example 1: SUPPS-0254 (Supplements)

```json
{
  "code": "SUPPS-0254",
  "description": "CBUM Thavage PUMP RTD 355ml Juicy Pumps",
  "rows": {
    "a3":  { "label": "Item Type",         "value": "Drinks, Energy" },
    "a4":  { "label": "Flavour / Variant", "value": "Juicy Pumps" },
    "a5":  { "label": "Brand",             "value": "CBUM Thavage" },
    "a6":  { "label": "",                  "value": "" },
    "a7":  { "label": "",                  "value": "" },
    "a8":  { "label": "",                  "value": "" },
    "a9":  { "label": "",                  "value": "" },
    "a10": { "label": "SUPPLIER",          "value": "Fitness Vending" },
    "a11": { "label": "XL PART ID",        "value": "SUPPS-0254" },
    "a12": { "label": "SUPPLIER SKU",      "value": "3481611960416" },
    "a1":  { "label": "Q to Order",        "value": "1" },
    "a2":  { "label": "Box Size",          "value": "12" }
  },
  "image":     "<image URL>",
  "qrPayload": "<XL site item URL>",
  "color":     "#121826"
}
```

### Example 2: BLT-0033 (Bolts) — different labels, different colour

```json
{
  "code": "BLT-0033",
  "description": "<bolt name>",
  "rows": {
    "a3":  { "label": "Size (Diameter)", "value": "M10" },
    "a4":  { "label": "Length",          "value": "50mm" },
    "a5":  { "label": "Grade",           "value": "8.8" },
    "a6":  { "label": "Material",        "value": "Stainless 316" },
    "a7":  { "label": "Finish / Coating","value": "Zinc Plated" },
    "a8":  { "label": "Head Type",       "value": "Hex" },
    "a9":  { "label": "Thread Pitch",    "value": "1.5mm" },
    "a10": { "label": "SUPPLIER",        "value": "<supplier>" },
    "a11": { "label": "XL PART ID",      "value": "BLT-0033" },
    "a12": { "label": "SUPPLIER SKU",    "value": "<sku>" },
    "a1":  { "label": "Q to Order",      "value": "1" },
    "a2":  { "label": "Box Size",        "value": "100" }
  },
  "image":     "<image URL>",
  "qrPayload": "<XL site item URL>",
  "color":     "#1f6feb"
}
```

### Example 3: PAPP-0010 (Paper & Printing) — only 2 custom slots used, pink colour

```json
{
  "code": "PAPP-0010",
  "description": "<paper item name>",
  "rows": {
    "a3":  { "label": "Size",         "value": "A4" },
    "a4":  { "label": "Type",         "value": "Gloss 200gsm" },
    "a5":  { "label": "",             "value": "" },
    "a6":  { "label": "",             "value": "" },
    "a7":  { "label": "",             "value": "" },
    "a8":  { "label": "",             "value": "" },
    "a9":  { "label": "",             "value": "" },
    "a10": { "label": "SUPPLIER",     "value": "<supplier>" },
    "a11": { "label": "XL PART ID",   "value": "PAPP-0010" },
    "a12": { "label": "SUPPLIER SKU", "value": "<sku>" },
    "a1":  { "label": "Q to Order",   "value": "1" },
    "a2":  { "label": "Box Size",     "value": "500" }
  },
  "image":     "<image URL>",
  "qrPayload": "<XL site item URL>",
  "color":     "#f7c8d9"
}
```

Notice how each category produces a different `color` and different A3–A9 labels — but the standard slots (`a1`, `a2`, `a10`–`a12`) are identical across categories. That's the pattern.

If your generated JSON for any item doesn't follow this pattern (right labels for the right category, right colour), the bug is in your category lookup. Fix the data, do not touch the print URL.

---

## Hard rules on the data itself

- ✅ Empty A-slots: send `{ "label": "", "value": "" }` — both empty strings. The print page auto-hides the row.
- ❌ Do NOT use placeholder strings like `"Attribute 6"`, `"N/A"`, `"-"`, or `"unset"` for empty slots — they will render as visible empty boxes.
- ✅ A3–A9 labels come from the **item's category schema**, not from a hardcoded list.
- ❌ Do NOT use SUPPS labels (`Item Type`, `Flavour / Variant`, `Brand`) for non-SUPPS items. PAPP items use PAPP labels, BLT items use BLT labels, etc.
- ✅ `a1` is **always** Q to Order. `a2` is **always** Box Size. They render as the two big rows at the bottom.
- ❌ Do NOT put Q to Order / Box Size in any slot other than `a1`/`a2`, regardless of where the form displays them.
- ✅ `a11.value` should equal `code` (XL Part ID is shown twice — top header and row a11).
- ✅ `qrPayload` should be the URL of the item's page on the Manus site so scanning the QR opens that item.
- ✅ `color` comes from the item's **category**, not from a hardcoded value or per-item override.
- ✅ `color` is hex like `"#7a1f3d"`. Defaults to `#121826` if missing or invalid.

---

## How to test (must pass for all THREE categories)

Per-category schemas are the new behaviour, so you MUST test more than one category. Pick one item from each:

| Item | Category | Expected A3–A9 labels | Expected colour |
|---|---|---|---|
| **SUPPS-0254** | Supplements | A3=`Item Type`, A4=`Flavour / Variant`, A5=`Brand`, A6–A9 empty | Navy `#121826` |
| **BLT-0033** (or any BLT item) | Bolts | A3=`Size (Diameter)`, A4=`Length`, A5=`Grade`, A6=`Material`, A7=`Finish / Coating`, A8=`Head Type`, A9=`Thread Pitch` | Blue (BLT category colour) |
| **PAPP-0010** (or any PAPP item) | Paper & Printing | A3=`Size`, A4=`Type`, A5–A9 empty | Pink (PAPP category colour) |

For each item:

1. Click the Print Label button.
2. Confirm visually:
   - Top header: the item's code
   - Card colour: matches the **category** (NOT always navy)
   - A3–A9 labels: match the **category schema** (NOT always SUPPS labels)
   - A1 (bottom big row): `Q to Order` with the item's quantity
   - A2 (bottom big row): `Box Size` with the item's box size
   - Empty rows (where the category has no label): hidden — no visible boxes
   - SUPPLIER / XL PART ID / SUPPLIER SKU populated correctly

If anything's wrong: **the bug is in your `buildProduct` function**, specifically in the category lookup. Fix the data, do not touch the print page.

---

## Done = this

- Every product page has a working "Print Label" button.
- Clicking it opens the external print page with that item's data correctly populated.
- **All three test items** (SUPPS, BLT, PAPP) render with the right per-category labels and colours.
- You did not visit, preview, restyle, iframe, or modify the print URL.

Send back: screenshots of the rendered labels for **SUPPS-0254, one BLT item, and one PAPP item** + the URL of each product page.

---

# Phase 2 — Print Queue (multiple cards per page)

Once the single-card button is working on every product page, add a **Print Queue** that batch-prints multiple selected items in one print job.

## Live URL for batch printing

```
https://matt-xlfitness.github.io/xl-kanban-label/sheet.html
```

Same opaque-URL rules apply — do not visit, embed, or restyle.

## How it works

Same `?data=` mechanism, but the JSON is now an **ARRAY** of products instead of a single object. The sheet page lays them out **2 cards per A4 page** (the maximum that fits at 127mm) and paginates automatically.

```js
// Build the sheet URL from an array of selected products
function makePrintSheetHref(productsArray) {
  const json = JSON.stringify(productsArray);            // ← array, not object
  const b64  = btoa(unescape(encodeURIComponent(json)));
  return "https://matt-xlfitness.github.io/xl-kanban-label/sheet.html?data=" + b64;
}
```

Each product in the array uses the **exact same shape** documented above for single-card mode — no changes per-item. The only difference is you wrap them in `[ … ]`.

## Print Queue UI on the Manus site

1. **Add a checkbox or "Add to Print Queue" button** on each item — selected items go into a session-scoped list (`localStorage`, Vuex, Redux, whatever).
2. **A floating "Print Queue (N)" indicator** showing how many items are queued.
3. **A "Print Selected" button** that:
   - Builds the array of `product` objects (each formatted per the Phase 1 template)
   - Calls `makePrintSheetHref(arr)`
   - Opens the resulting URL in a new tab
4. **The user prints from that tab** — one print job, multiple A4 pages, 2 cards per page.

## Example

If 5 items are selected, the sheet page renders:
- Page 1: card 1, card 2
- Page 2: card 3, card 4
- Page 3: card 5 (alone, no trailing blank page)

User prints once → 3 sheets of A4 → cuts → 5 finished Kanban cards.

## Hard rules for the array

- ✅ Order in the array = order on the printed sheet. Sort however you want before encoding.
- ✅ Empty rows still hide per-card (same as single-card mode).
- ✅ Per-card colour still applied per-card via `color` field.
- ❌ Do NOT include duplicate items in the array unless the user explicitly wants two copies of the same card.
- ❌ Do NOT change the per-card JSON shape — it's identical to single-card mode.

## Test it

Open this URL to see the sheet render with 2 sample products (built-in test data — no real codes needed):

```
https://matt-xlfitness.github.io/xl-kanban-label/sheet.html?codes=SUPPS-0254,SUPPS-0356
```

— Matt

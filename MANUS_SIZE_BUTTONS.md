# Manus — Add Label-Size Buttons to the Print Queue

**Hi Manus.** Quick brief — read all of it before writing code.

This is an **incremental change** on top of `MANUS_PRINT_BUTTON.md` Phase 2 (the Print Queue / batch sheet flow). The data shape, encoding helper, and URL base are **unchanged**. Only the UI and one URL parameter change.

---

## What I want

Add **4 buttons** alongside the existing Print Queue:

- **Large**
- **Medium**
- **Small**
- **All**

The user can select any combination (Large = on, Medium = off, Small = on, etc.). The **All** button is a shortcut that toggles all three at once.

When the user clicks **Print Selected**, the queued items get printed at every selected size — in **one** print job, on **A4 paper**, with as few pages as possible.

Example: 3 items selected + Large + Medium = 3 large cards (2 per A4 = 2 pages) followed by 3 medium cards (5 per A4 = 1 page) = **3 A4 pages total** in one print dialog.

---

## ⛔ Hard rules — same as before

The URL `https://matt-xlfitness.github.io/xl-kanban-label/sheet.html` is an **opaque external service**. Treat it like a third-party API endpoint.

- ❌ Do NOT visit, preview, screenshot, or render the URL.
- ❌ Do NOT iframe, embed, or proxy the URL inside the Manus site.
- ❌ Do NOT recreate, restyle, redesign, or "improve" what's at the URL.
- ❌ Do NOT change paper size, margins, layout, or any print rule yourself — A4 is handled by the print page.
- ✅ DO treat the URL as a string. Build it, put it in `<a href="…">`, open in a new tab.

The print page handles the layout, page breaks, and per-size CSS. Your job is only to (a) construct the URL with the right `?data=` and `&sizes=` and (b) render the buttons.

---

## The new URL contract

Same base URL, same `?data=` array, **plus** one new query param:

```
https://matt-xlfitness.github.io/xl-kanban-label/sheet.html?data=<base64 JSON array>&sizes=<csv>
```

| `sizes=` value | Meaning |
|---|---|
| omitted | Large only (existing behaviour — backwards compatible) |
| `large` | Large only |
| `medium` | Medium only |
| `small` | Small only |
| `large,medium` | Large then Medium |
| `large,medium,small` | All three (or use `sizes=all` shortcut) |
| `all` | Alias for `large,medium,small` |

Order in the CSV = order on the printed sheet. Group by size (all Large, then all Medium, then all Small).

---

## Per-item `sizes` field (per-card override) — NEW

The Print Queue UI has **per-item** size toggles (LRG / MED / SML / ALL on every queued card). Different items often need different sizes:

- Bolt bins (BLT) → Small only
- Supplement drinks (SUPPS) → Large only
- Stationery (PAPP) → Medium + Large

The print page now respects a **per-item `sizes` array** inside each product object. This **overrides the global `&sizes=`** for that specific card.

### Add `"sizes"` to each product object

```json
{
  "code": "BLT-1999",
  "description": "Hex Head 1/4\" x 1/2\" (BSF Fine)",
  "rows": { "...": "..." },
  "image": "https://...",
  "qrPayload": "https://kanflow.manus.space/items/1999",
  "color": "#0ea5e9",
  "sizes": ["small"]
}
```

### Resolution rule (the print page applies this per card)

| Per-item `sizes` field | Behaviour for that card |
|---|---|
| `["large"]`, `["small","medium"]`, etc. | Use exactly those sizes for this card only |
| `["all"]` | Expands to all 3 sizes for this card only |
| Omitted, `null`, or `[]` | Fall back to the global `&sizes=` |

Same alias rules as global: `"lrg"` / `"med"` / `"sml"` are accepted.

### Worked example

Queue with 3 items, each with different per-item sizes:

```js
[
  { "code": "SUPPS-0254", "...": "...", "sizes": ["large"] },
  { "code": "BLT-1999",   "...": "...", "sizes": ["small"] },
  { "code": "PAPP-0010",  "...": "...", "sizes": ["medium", "large"] }
]
```

Result: 4 cards total
- 2 Large (SUPPS-0254 + PAPP-0010)
- 1 Medium (PAPP-0010)
- 1 Small (BLT-1999)

Printed in size order: 1 A4 page of large + 1 A4 page of medium + 1 A4 page of small = 3 A4 pages, all in **one print job**.

### Backwards compat

If you don't include `"sizes"` per item, the global `&sizes=` keeps working exactly as documented above. No regression for existing flows.

---

## Buttons mapping

| Button label | UI behaviour | Becomes in URL |
|---|---|---|
| **Large** | Toggle | `large` |
| **Medium** | Toggle | `medium` |
| **Small** | Toggle | `small` |
| **All** | Toggles all three on (or off if all already on) | `large,medium,small` (joined CSV of whichever toggles are on) |

The **All** button is a convenience shortcut — internally it just sets the same three toggles. The URL only contains the actual sizes selected.

---

## Code — extend the existing helper

```js
// Existing helper (unchanged signature, one extra arg)
function makePrintSheetHref(productsArray, sizes = ["large"]) {
  const json = JSON.stringify(productsArray);
  const b64  = btoa(unescape(encodeURIComponent(json)));
  const sizesCsv = sizes.join(",");
  return `https://matt-xlfitness.github.io/xl-kanban-label/sheet.html?data=${b64}&sizes=${sizesCsv}`;
}
```

```js
// In your print-queue component, add the per-item sizes when you build each product
function buildLabelProduct(item) {
  return {
    code: item.code,
    description: item.name,
    rows: { /* a1–a12 per MANUS_PRINT_BUTTON.md */ },
    image: item.imageUrl ?? "",
    qrPayload: item.publicUrl ?? "",
    color: item.category.color ?? "#121826",
    sizes: item.activeSizes      // ← NEW: ["large", "small"] from the per-card toggles
  };
}

const queuedItems   = queue.map(buildLabelProduct);
const fallbackSizes = ["large"];               // global default if a product omits its sizes

// On "Print Selected":
const url = makePrintSheetHref(queuedItems, fallbackSizes);
window.open(url, "_blank", "noopener");
```

The per-item product shape is **the existing shape from `MANUS_PRINT_BUTTON.md` plus the new `sizes` array**. All other fields (rows a1–a12, image, qrPayload, color) are unchanged. The print page picks which fields to render based on the chosen size for each card.

---

## UI rules

- **Default state** when the queue is opened: Large = on, Medium = off, Small = off (matches today's behaviour).
- **Multi-select**: any of the 3 size toggles can be on at the same time.
- **Disable "Print Selected"** when:
  - the queue is empty (0 items), OR
  - no size is toggled on (0 sizes).
- **Show a count** like *"Print 3 items × 2 sizes = 6 cards"* near the Print button so the user knows what they'll get.
- **All button**:
  - Click while any size is OFF → turn all 3 ON.
  - Click while all 3 are ON → turn all 3 OFF (deselect-all shortcut).

---

## Test plan

Set up a queue with 3 items from different categories (one SUPPS, one BLT, one PAPP) so per-category schemas + colours all get exercised.

| Test | Expected |
|---|---|
| Queue 3 items, **Large** only, click Print | Opens sheet.html in new tab. 3 large cards, 2 per A4 page → 2 pages. |
| Queue 3 items, **Medium** only | 3 medium cards, multiple per A4 → 1 page. |
| Queue 3 items, **Small** only | 3 small cards, multiple per A4 → 1 page. |
| Queue 3 items, **All** | 3 large + 3 medium + 3 small, grouped by size, fewest pages possible. |
| Queue 3 items, **0 sizes selected** | Print button disabled. |
| Queue 0 items, any sizes | Print button disabled. |
| Existing single-item Print Label button | Still works exactly as before — no regression. |
| Per-item: SUPPS=`["large"]`, BLT=`["small"]`, PAPP=`["medium","large"]` | 2 large + 1 medium + 1 small = 4 cards total, grouped by size on 3 A4 pages. |
| Per-item omitted on all 3 items, global `&sizes=medium` | All 3 print at medium only (fallback to global). |
| Per-item: 1 item with `sizes=["small"]`, 2 items with no sizes field, global `&sizes=large` | 1 small + 2 large = 3 cards total. |

Confirm in the print dialog: paper = **A4**, scale = **100%**, margins = **None**, background graphics = **ON**.

---

## Done = this

- 4 buttons (Large / Medium / Small / All) on each queued card (per-item, not global).
- Each product object in the `?data=` array includes a `"sizes": [...]` array reflecting that card's toggles.
- Multi-select works per item; **All** acts as toggle-all-three; count line shows total cards.
- "Print Selected" opens `sheet.html?data=…&sizes=…` in a new tab — `&sizes=` becomes the fallback for any item missing per-item sizes.
- 0-item or all-cards-have-empty-sizes states disable the Print button cleanly.
- Single-item "Print Label" button unchanged.
- You did not visit, preview, restyle, iframe, or modify the print page.

Send back: a screenshot of the queue UI with all 4 buttons + the URL string your code generates for a 3-item × all-sizes test. I'll verify the URL on my end before you ship.

— Matt

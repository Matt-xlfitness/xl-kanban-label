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
// In your print-queue component
const queuedItems   = [...];                  // already-formatted product objects
const selectedSizes = ["large", "medium"];    // from your size-button state

// On "Print Selected":
const url = makePrintSheetHref(queuedItems, selectedSizes);
window.open(url, "_blank", "noopener");
```

The per-item `product` object shape is **unchanged** — exactly the one documented in `MANUS_PRINT_BUTTON.md` (rows a1–a12, image, qrPayload, color, etc.). Don't modify it per size — the print page picks which fields to render based on the size.

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

Confirm in the print dialog: paper = **A4**, scale = **100%**, margins = **None**, background graphics = **ON**.

---

## Done = this

- 4 buttons (Large / Medium / Small / All) on the Print Queue UI.
- Multi-select works, **All** acts as toggle-all, count line shows items × sizes.
- "Print Selected" opens `sheet.html?data=…&sizes=…` in a new tab.
- 0-item or 0-size states disable the Print button cleanly.
- Single-item "Print Label" button unchanged.
- You did not visit, preview, restyle, iframe, or modify the print page.

Send back: a screenshot of the queue UI with all 4 buttons + the URL string your code generates for a 3-item × all-sizes test. I'll verify the URL on my end before you ship.

— Matt

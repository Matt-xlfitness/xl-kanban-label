# XL Kanban Label

A printable 127 × 127 mm Kanban inventory card for XL Fitness parts.

## Pages

- **`index.html`** — single label, 1 card per 127×127mm page. Open with `?code=SUPPS-0254` or `?data=<base64 JSON>`.
- **`sheet.html`** — multi-card print sheet, 2 cards per A4 page (auto-paginated). Open with `?data=<base64 JSON ARRAY>` or `?codes=A,B,C`.
- **`builder.html`** — interactive URL builder for the single-label page.

## Briefs

- **`MANUS_PRINT_BUTTON.md`** — concise brief for Manus (single + batch print integration).
- **`MANUS_BRIEF.md`** — full integration reference.
- **`MANUS_HANDOFF.md`** — paste-ready handoff message.
- **`INTEGRATION.md`** — full technical spec.

## Quick tests

| URL | What it shows |
|---|---|
| `https://matt-xlfitness.github.io/xl-kanban-label/?code=SUPPS-0254` | Single-card ground truth |
| `https://matt-xlfitness.github.io/xl-kanban-label/?code=SUPPS-0356` | Single-card ground truth (Gatorade) |
| `https://matt-xlfitness.github.io/xl-kanban-label/sheet.html?codes=SUPPS-0254,SUPPS-0356` | Sheet with 2 sample cards |
| `https://matt-xlfitness.github.io/xl-kanban-label/builder.html` | Build a `?data=` URL from JSON |

## How external systems pass data

### Single label (`index.html`)
- **`?data=<base64 JSON>`** — caller encodes one product object. No backend.
- **`?code=<XL_PART_ID>`** — fetched from `API_ENDPOINT` if configured.

### Print sheet (`sheet.html`)
- **`?data=<base64 JSON ARRAY>`** — caller encodes an array of product objects. Renders 2 cards per A4 page, auto-paginated.
- **`?codes=A,B,C`** — comma-separated list, looked up in built-in samples (testing only).

Source design: Figma file `ePmTOphFK3IFYsh7X78hcs` node `1:4`.

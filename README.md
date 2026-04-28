# XL Kanban Label

A printable 127 × 127 mm Kanban inventory card for XL Fitness parts.

- **`index.html`** — the live label page. Open with `?code=SUPPS-0356` or `?data=<base64 JSON>`.
- **`builder.html`** — paste product JSON, get a shareable URL.
- **`INTEGRATION.md`** — full technical spec.
- **`MANUS_BRIEF.md`** — instructions for the integrating AI / engineer.

## Quick test

```
https://<github-pages-url>/?code=SUPPS-0356        ← uses local sample data
https://<github-pages-url>/builder.html             ← interactive URL builder
```

## How external systems pass data

Either:

1. **`?data=<base64-encoded JSON>`** — no backend required. Build the URL and link to it.
2. **`?code=<XL_PART_ID>`** — set `API_ENDPOINT` in `index.html` to your part-catalog API.

Source design: Figma file `ePmTOphFK3IFYsh7X78hcs` node `1:4`.

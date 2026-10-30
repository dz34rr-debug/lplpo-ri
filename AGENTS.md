# AGENTS.md

LPLPO Depo: a TanStack Start app that builds a monthly LPLPO (pharmacy stock usage report) from Excel exports
(SIMRS "Medify" reports). UI language is Indonesian — keep all user-facing copy in Indonesian.

## Formula (source of truth)

Mirrors the user's `LPLPO RI` sheet:
`SISA = stok awal + RET + H + IN − OUT − SALE`, `WORKSHEET = SUMIF(name, AKHIR)`, `SELISIH = SISA − WORKSHEET`.
All lookups are by **item name** (normalised: trim, collapse spaces, uppercase). The master item list and its order come
from the previous month's SO (or a saved period). For SO, the first match per name is used (VLOOKUP semantics); every
other source is summed per name (pivot-table semantics).

## Architecture

| Path | Purpose |
| ---- | ------- |
| `src/lib/lplpo.ts` | Pure logic: source definitions (header names, sheet-name patterns), header detection, parsing, `computeLplpo`, `summarize`, number formatting. No browser/server deps. |
| `src/lib/excel.ts` | Browser-only SheetJS wrapper (dynamically imported): read a single-source file, read a combined workbook by sheet name, export the LPLPO workbook. |
| `src/lib/periods.functions.ts` | Server functions (list/get/save/delete saved periods) using Drizzle. `savePeriod` replaces an existing depo+month. |
| `db/schema.ts`, `db/index.ts` | Drizzle schema (`periods`, `period_items`) and client (`drizzle-orm/netlify-db`). |
| `netlify/database/migrations/` | Generated migrations — never edit applied ones; regenerate with `npx drizzle-kit generate --name …`. |
| `src/components/lplpo-table.tsx` | Shared `SummaryCards` and `LplpoTable` (search, filters, sort, pagination). |
| `src/components/app-shell.tsx` | Header/nav/footer wrapping every page (mounted in `__root.tsx`). |
| `src/routes/index.tsx` | Main "Hitung LPLPO" screen: period, source uploads, results, export, save. |
| `src/routes/riwayat/` | Saved-period list and detail (`$id`). |

## Conventions and decisions

- Excel parsing happens entirely in the browser; only computed rows are sent to the server when saving.
- Combined-workbook sheet detection is by sheet name (`H`/`RET` and `IN`/`O` share column layouts). Per-source uploads detect by headers.
- The exported workbook mirrors the user's combined file: `LPLPO RI` uses live VLOOKUP/SUMIF formulas into per-source sheets
  (`SO <bulan lalu>`, RET, H, IN, O, SALE, Worksheet) whose headers the app can re-read. `SO <bulan ini>` (linked to WORKSHEET)
  is placed before `SO <bulan lalu>` so single-file SO upload takes the first and combined upload takes the last.
- Number parsing (`toNumber`) accepts en ("1,234.5") and id ("26923,05") formats.
- Theme variables live in `src/styles.css` (pharmacy green palette, Fraunces headings, IBM Plex Sans/Mono). Use `.num` for numeric cells.
- Use `drizzle-orm@beta` / `drizzle-kit@beta` (required for the Netlify adapter).

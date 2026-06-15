# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**misc-projects**: Suite of 12+ internal logistics tools built with vanilla HTML/CSS/JS. No build tools, no frameworks. Single-file deployment model.

**Tools in repo:**
- RUTEOPM, RUTEOHD, RUTEO24HRS — route planning and assignment
- GENERARRESUMEN — batch summary generation
- PREOLAREGIONES — region assignment (PREOLA)
- ADHERENCIACSV — geosort filtering  
- ACTUALIZAROTIF — OTIF metric updates
- REPORTERUTAS — route progress (React+Tailwind, not in design system)
- AUDITORIARUTAS — route auditing
- CALCULOLEADTIME — leadtime calculations
- ASIGNACIONGEO — unshipped ISO tracking
- MONITOREORUTAS — map-based route monitoring (new)

**Landing:** `index.html` — grid menu linking to each tool.

---

## Design System

**Read:** `DESIGN_SYSTEM.md` (comprehensive, 1500 lines).

### Key Principles
- **Warm off-white** base (`#FAFAF9`), **near-black primary** (`#18181B`), **minimal shadows**.
- **No build tools** — all CSS inlined in `<style>` blocks.
- **Plus Jakarta Sans** + monospace stack for UI and data.
- **Data-first hierarchy** — typography and borders, not color saturation.
- **Max 3 semantic colors** visible per screen simultaneously.

### Design Tokens (CSS variables)
```css
/* Canvas */
--bg: #FAFAF9; --bgCard: #FFFFFF; --bgCardAlt: rgba(0,0,0,0.026);
/* Text */
--text: #18181B; --textSub: #52525B; --textFaint: #A1A1AA;
/* Accent (reserved for interactive states only) */
--blue: #2563EB; --blueHover: #1D4ED8;
/* Semantic (chips/badges only, never section backgrounds) */
--red: #DC2626; --green: #16A34A; --orange: #EA580C; --yellow: #CA8A04; --purple: #7C3AED;
/* Pastel fills (for chips, pills, badges) */
--pasteBlue, --pasteRed, --pasteGreen, --pasteOrange, --pasteGray, etc.
```

### Core Component Patterns

**Button precedence:**
- `.btn-primary` → near-black `#18181B`, never blue
- `.btn-secondary`, `.btn-danger`, `.btn-green` → colored pastels with tinted borders
- All buttons have subtle transform on hover/active: `translateY(-1px)` / `scale(0.98)`

**Spreadsheet tables (`.dash-table`):**
- Sticky headers (column letters, names)
- Monospace data with tabular-nums
- Row selection via `.row-active` (blue tint, no solid background)
- Input cells transparent until focused

**Tabs:**
- `.tab-btn.active` → near-black underline (not blue background)
- `.tab-badge` → pastel gray by default, colored variants for urgent state

**Modals & overlays:**
- 22% black backdrop + 4px blur
- Scale-up + fade animation (`--t-slow` timing)
- Max 520px width, shadow-xl

**Toast notifications:**
- Fixed bottom-center, slide-up animation
- Dark background (`#18181B`), optional color variants (ok/error/warn)

**Pills & badges:**
- Badges: small, numeric, pastel fills (9px font, monospace)
- Pills: state/region labels, bevel corners (`--r-xs`), never fully rounded

---

## Architecture & File Structure

```
index.html               # Landing menu grid (4 columns, 12 items)
DESIGN_SYSTEM.md        # Exhaustive style reference (read before editing)
comunas_data.js         # Static region/commune data
docs/                   # Placeholder for documentation

*.html (tools)          # Each tool is a complete standalone file
                        # Inline <style>, inline JS, no external deps
                        # Pattern: <head> + <style> + <body> + <script>
```

**No separate CSS, no JS bundler, no node_modules.** All resources are either CDN (Google Fonts, XLSX, LZ-String) or inline.

---

## Development Workflow

### Creating or Modifying a Tool

1. **Copy structure from existing tool** (e.g., RUTEOPM.html)
   - Paste `:root` design tokens (section 1 of DESIGN_SYSTEM.md)
   - Include Google Fonts link in `<head>`
   - Use `<body class="app-layout">` for full-screen tools

2. **Layout shells** (choose one pattern):
   - `.header` → `.main-content` → `.footer` (if needed)
   - `.tab-pane.active` for multi-tab tools
   - `.main-padded` for scrollable content areas

3. **Data handling:**
   - Use `XLSX.read()` for file imports
   - Use `LZ-String` for localStorage compression
   - Nameclspace JS to avoid collisions (e.g., `window.MyTool = { ... }`)

4. **Apply design tokens consistently:**
   - Use CSS variables, never hardcoded hex
   - Buttons: `.btn` + `.btn-primary` / `.btn-secondary` / color variant
   - Tables: `.dash-table` (includes sticky headers, row selection)
   - Modals: `.modal-overlay` + `.modal-content`

5. **Anti-AI-tell checklist** (from DESIGN_SYSTEM.md section 21):
   - [ ] Font is Plus Jakarta Sans (not Segoe alone)
   - [ ] Primary button is near-black, not blue
   - [ ] Badges/pills use `--paste*` pastels, not saturated colors
   - [ ] Active tabs have near-black underline, not blue background
   - [ ] Stat values default to near-black (color only with semantic reason)
   - [ ] Shadows: `shadow-xs` or `shadow-sm` max (no `shadow-lg` on cards)
   - [ ] Borders: neutral `rgba(0,0,0,0.06–0.09)`, never blue-tinted
   - [ ] Max 3 semantic colors visible per screen
   - [ ] Modals animate with scale + translateY; dropdowns with fadeInDown
   - [ ] Accordion guides: border-bottom only, no colored box

### Git Workflow

- Branch naming: feature branches use descriptive names (e.g., `claude/monitoreo-rutas-map`)
- Commit messages: brief, present tense
- **Always test in a real browser** before pushing (no headless testing for UI tools)

### Testing & Verification

**No unit tests, no build step.** Manual verification required:
- Open tool in browser (double-click `.html` file or serve locally)
- Test file upload/import flows (XLSX parsing)
- Test tab switching, modal open/close
- Test responsive behavior (@media queries in DESIGN_SYSTEM.md)
- Verify no console errors

---

## Common Tasks

### Add a New Tool to the Landing Menu
1. Create the tool `.html` file
2. Edit `index.html`: add new `<a>` link with incrementing `.num` (e.g., 13)
3. Commit both files
4. Push to master branch

### Update Design System Application
- **Old tools:** migrate component styles from DESIGN_SYSTEM.md section 22 (tool-by-tool guide)
- **Copy `:root` variables first**, then update buttons, badges, tables per section 5–12
- Run the section 21 checklist before marking complete

### Shared Data (comunas_data.js)
- Static Chilean regions and communes
- Import into tools as `<script src="comunas_data.js"></script>`
- Reference: `window.ComunasData` (or similar namespace in file)

---

## Key Implementation Details

### Inline Styles Only
All CSS is inline in `<style>` blocks. No external stylesheets except CDN fonts.  
**Rationale:** Single-file deployment, no build complexity, works offline after first load.

### JS Pattern
Tools typically use vanilla JS with:
- Event delegation (one listener on parent, multiple targets)
- `localStorage` for state persistence (often compressed with LZ-String)
- XLSX library for spreadsheet import/export
- No jQuery, no frameworks (except REPORTERUTAS which uses React)

### Accessibility Baseline
- Semantic HTML (`<button>`, `<input>`, `<table>`)
- Focus states on all interactive elements (blue outline)
- Form labels with `.field-label` class
- No color-only information (always paired with text or icon)

---

## Performance & Constraints

- **File size:** each tool is 20–250 KB (uncompressed HTML)
- **Browser support:** modern Chrome/Firefox/Safari (ES6, CSS Grid, CSS variables)
- **No service workers, no caching strategy** — tools served as-is
- **Network:** XLSX and LZ-String loaded from CDN; falls back gracefully if unavailable

---

## Useful References in This Repo

- **DESIGN_SYSTEM.md** — complete component library, color palette, animation specs, checklist
- **docs/superpowers** — placeholder (may contain internal process docs)
- **index.html** — minimal example of grid layout and link structure

---

## Questions or Uncertainties?

- **"Does this color match the system?"** → check `DESIGN_SYSTEM.md` section 1 `:root` variables
- **"Should I add a new button variant?"** → reuse existing `.btn-*` classes (primary, secondary, danger, green); avoid creating new ones
- **"How do I add a new tab?"** → copy `.tab-btn` + `.tab-pane` pattern from an existing tool; mirror `.tab-badge` for counts
- **"Is this tool responsive?"** → must support 640px+ (mobile breakpoint at 640px); test with browser devtools

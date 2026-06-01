# Design System — Logistics Tools

Sistema de diseño unificado para las 8 herramientas internas de logística.
Filosofía: **warm off-white · tipografía editorial · minimalismo utilitario · datos primero**.

No requiere build tools ni dependencias adicionales. Aplica copiando al `<style>` inline.

---

## 0. Tipografía — CDN (opcional pero recomendado)

Añade este `<link>` antes del `<style>` en cada herramienta para obtener la tipografía correcta:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
```

Si no hay acceso a Google Fonts (entorno sin internet), el stack de fallback funciona con `Helvetica Neue` en macOS o `Segoe UI Variable` en Windows moderno. La jerarquía visual no se pierde.

---

## 1. Design Tokens

```css
:root {
  /* ─── Canvas ──────────────────────────────────────────────────── */
  --bg:        #FAFAF9;   /* warm off-white, no cool-gray */
  --bgCard:    #FFFFFF;
  --bgCardAlt: rgba(0, 0, 0, 0.026);
  --bgHover:   rgba(0, 0, 0, 0.042);

  /* ─── Texto ───────────────────────────────────────────────────── */
  --text:      #18181B;   /* zinc-900, no pure black */
  --textSub:   #52525B;   /* zinc-600 */
  --textFaint: #A1A1AA;   /* zinc-400 */

  /* ─── Borders ─────────────────────────────────────────────────── */
  --border:      rgba(0, 0, 0, 0.09);
  --borderSoft:  rgba(0, 0, 0, 0.06);
  --borderFocus: #2563EB;

  /* ─── Accent funcional (solo para estados interactivos) ──────── */
  /* El azul se reserva para: tab activo, focus ring, links, selección en tabla */
  --blue:      #2563EB;
  --blueHover: #1D4ED8;
  --accentBg:  rgba(37, 99, 235, 0.06);

  /* ─── Semántica — NUNCA como background de sección grande ─────── */
  --red:    #DC2626;
  --green:  #16A34A;
  --orange: #EA580C;
  --yellow: #CA8A04;
  --purple: #7C3AED;

  /* ─── Pastels semánticos (para chips, badges, pills) ─────────── */
  /* Fills: muy desaturados, nunca vividos */
  --pasteBlue:   #EFF6FF;  /* text: --blue */
  --pasteRed:    #FEF2F2;  /* text: --red */
  --pasteGreen:  #F0FDF4;  /* text: --green */
  --pasteOrange: #FFF7ED;  /* text: --orange */
  --pasteYellow: #FEFCE8;  /* text: --yellow */
  --pastePurple: #F5F3FF;  /* text: --purple */
  --pasteGray:   #F4F4F5;  /* text: --textSub */

  /* ─── Tipografía ──────────────────────────────────────────────── */
  --font-base: 'Plus Jakarta Sans', 'Helvetica Neue', system-ui, -apple-system, sans-serif;
  --font-mono: 'Geist Mono', 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;

  /* ─── Radii — escala compacta (herramientas, no marketing) ───── */
  --r-xs: 2px;
  --r-sm: 4px;   /* inputs, badges */
  --r-md: 6px;   /* botones, cards pequeñas */
  --r-lg: 8px;   /* cards, modals */
  --r-xl: 10px;  /* modals grandes */
  /* Aliases de compatibilidad */
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
  --radius-xl: 10px;

  /* ─── Shadows — prácticamente invisibles ─────────────────────── */
  /* La jerarquía se construye con borders y color, no con sombras */
  --shadow-xs: 0 1px 2px rgba(0, 0, 0, 0.04);
  --shadow-sm: 0 1px 4px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 2px 8px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 8px 24px rgba(0, 0, 0, 0.07);
  --shadow-xl: 0 16px 40px rgba(0, 0, 0, 0.08);

  /* ─── Transitions ─────────────────────────────────────────────── */
  --t-fast:   0.12s ease;
  --t-normal: 0.20s ease;
  --t-slow:   0.28s cubic-bezier(0.16, 1, 0.3, 1);
  /* Aliases */
  --transition-fast:   0.12s ease;
  --transition-normal: 0.20s ease;
}
```

---

## 2. Base Reset & Body

```css
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body { height: 100%; }

body {
  font-family: var(--font-base);
  font-size: 12px;
  line-height: 1.55;
  background: var(--bg);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Apps de pantalla completa con header fijo */
body.app-layout {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  height: 100vh;
}
```

---

## 3. Scrollbar

```css
::-webkit-scrollbar       { width: 5px; height: 5px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.14);
  border-radius: 3px;
}
::-webkit-scrollbar-thumb:hover { background: rgba(0, 0, 0, 0.26); }
```

---

## 4. Tipografía

```css
h1 { font-size: 17px; font-weight: 700; color: var(--text); letter-spacing: -0.01em; }
h2 { font-size: 14px; font-weight: 700; color: var(--text); letter-spacing: -0.01em; }
h3 { font-size: 12px; font-weight: 600; color: var(--text); }

/* Label de sección — reemplaza el eyebrow genérico */
.section-label {
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.09em;
  color: var(--textFaint);
}

/* Etiqueta sobre input */
.field-label {
  display: block;
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: var(--textFaint);
  margin-bottom: 5px;
}

/* Datos numéricos en tabla y métricas */
.num { font-family: var(--font-mono); font-variant-numeric: tabular-nums; }

/* Helpers */
.text-xs   { font-size: 10px; color: var(--textFaint); }
.text-sm   { font-size: 11px; color: var(--textSub);   }
.text-mono { font-family: var(--font-mono); }
```

---

## 5. Botones

El botón primario usa **near-black**, no azul. El azul se reserva para estados interactivos (focus, selección), nunca como relleno de botón principal.

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  padding: 5px 12px;
  border: 1px solid transparent;
  border-radius: var(--r-md);
  font-family: var(--font-base);
  font-size: 11px;
  font-weight: 500;
  line-height: 1;
  cursor: pointer;
  outline: none;
  white-space: nowrap;
  user-select: none;
  transition:
    background   var(--t-fast),
    border-color var(--t-fast),
    color        var(--t-fast),
    transform    0.10s ease;
}

.btn:hover:not(:disabled)  { transform: translateY(-1px); }
.btn:active:not(:disabled) { transform: translateY(0) scale(0.98); }
.btn:disabled              { opacity: 0.32; cursor: not-allowed; }
.btn:focus-visible         { outline: 2px solid var(--borderFocus); outline-offset: 2px; }

/* Primary — near-black, no azul */
.btn-primary {
  background: #18181B;
  color: #FFFFFF;
  border-color: rgba(0, 0, 0, 0.06);
}
.btn-primary:hover:not(:disabled) { background: #27272A; }

/* Secondary — superficie neutral */
.btn-secondary {
  background: var(--bgCardAlt);
  color: var(--text);
  border-color: var(--border);
}
.btn-secondary:hover:not(:disabled) { background: var(--bgHover); }

/* Danger */
.btn-danger {
  background: var(--pasteRed);
  color: var(--red);
  border-color: rgba(220, 38, 38, 0.18);
}
.btn-danger:hover:not(:disabled) { background: rgba(220, 38, 38, 0.12); }

/* Confirm / success */
.btn-green {
  background: var(--pasteGreen);
  color: var(--green);
  border-color: rgba(22, 163, 74, 0.18);
}
.btn-green:hover:not(:disabled) { background: rgba(22, 163, 74, 0.12); }

/* Variante azul — solo donde el context claramente lo justifica */
.btn-blue {
  background: var(--blue);
  color: #fff;
  border-color: rgba(0, 0, 0, 0.10);
}
.btn-blue:hover:not(:disabled) { background: var(--blueHover); }

/* Tamaños */
.btn-sm { padding: 3px 8px; font-size: 10px; }
.btn-lg { padding: 8px 16px; font-size: 12px; font-weight: 600; }

/* Full-width (Ejecutar, Generar) */
.btn-block {
  width: 100%;
  padding: 9px;
  font-size: 12px;
  font-weight: 600;
  background: #18181B;
  color: #fff;
  border-radius: var(--r-md);
  border: 1px solid rgba(0, 0, 0, 0.08);
}
.btn-block:hover:not(:disabled) { background: #27272A; }
```

---

## 6. Header & Navegación

```css
.header {
  background: var(--bgCard);
  border-bottom: 1px solid var(--border);
  box-shadow: var(--shadow-xs);
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  position: sticky;
  top: 0;
  z-index: 100;
}

.header-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  height: 44px;
  border-bottom: 1px solid var(--borderSoft);
  gap: 12px;
}

/* Nombre de la herramienta */
.header-brand {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.01em;
  white-space: nowrap;
}

/* Sub-label (versión, módulo) */
.header-brand .brand-tag {
  font-size: 10px;
  font-weight: 500;
  color: var(--textFaint);
  font-family: var(--font-mono);
  letter-spacing: 0.02em;
}

.logo-text {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.09em;
  color: var(--textSub);
  text-transform: uppercase;
}

/* Tabs */
.tabs-container {
  display: flex;
  padding: 0 8px;
}

.tabs-wrapper {
  display: flex;
  align-items: flex-end;
}

.tab-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 7px 13px;
  border: none;
  border-bottom: 2px solid transparent;
  border-radius: var(--r-sm) var(--r-sm) 0 0;
  background: transparent;
  color: var(--textFaint);
  font-family: var(--font-base);
  font-size: 11px;
  font-weight: 500;
  cursor: pointer;
  white-space: nowrap;
  margin-bottom: -1px;
  transition: color var(--t-fast), background var(--t-fast), border-color var(--t-fast);
}

.tab-btn:hover {
  color: var(--textSub);
  background: var(--bgHover);
}

.tab-btn.active {
  color: var(--text);
  font-weight: 600;
  border-bottom-color: var(--text);  /* near-black underline, no azul */
  background: transparent;
}

/* Badge de conteo en tab — sin relleno de color, solo borde */
.tab-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 18px;
  height: 16px;
  padding: 0 4px;
  border-radius: 3px;
  font-size: 9px;
  font-weight: 700;
  font-family: var(--font-mono);
  background: var(--pasteGray);
  color: var(--textSub);
  border: 1px solid var(--border);
}

/* Badge con estado urgente */
.tab-badge.alert {
  background: var(--pasteRed);
  color: var(--red);
  border-color: rgba(220, 38, 38, 0.15);
}
```

---

## 7. Layout

```css
.main-content {
  flex: 1;
  overflow: hidden;
  position: relative;
}

.tab-pane {
  display: none;
  width: 100%;
  height: 100%;
  overflow-y: auto;
}

.tab-pane.active {
  display: flex;
  flex-direction: column;
}

.main-padded {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}

.container    { width: 100%; max-width: 900px;  margin: 0 auto; }
.container-sm { width: 100%; max-width: 560px;  margin: 0 auto; }
.container-lg { width: 100%; max-width: 1100px; margin: 0 auto; }

.grid-2 { display: grid; grid-template-columns: 1fr 1fr;       gap: 16px; }
.grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr;   gap: 12px; }
.grid-4 { display: grid; grid-template-columns: repeat(4,1fr); gap: 12px; }

@media (max-width: 640px) { .grid-2, .grid-3, .grid-4 { grid-template-columns: 1fr; } }
@media (max-width: 900px) { .grid-3, .grid-4 { grid-template-columns: 1fr 1fr; } }

.layout-sidebar {
  display: grid;
  grid-template-columns: 260px 1fr;
  gap: 20px;
  padding: 20px;
  min-height: calc(100vh - 52px);
}

@media (max-width: 900px) { .layout-sidebar { grid-template-columns: 1fr; } }
```

---

## 8. Cards & Panels

```css
.card {
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-lg);
  padding: 18px;
  /* Sin shadow — la jerarquía la da el border en --bg */
}

.card-header {
  padding-bottom: 12px;
  margin-bottom: 14px;
  border-bottom: 1px solid var(--borderSoft);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

/* Panel de parámetros colapsable */
.param-section {
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-lg);
  overflow: hidden;
  margin-bottom: 10px;
}

.param-section-header {
  padding: 10px 14px;
  border-bottom: 1px solid var(--borderSoft);
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--bgCardAlt);
}

.param-section-body { padding: 14px; }

/* Fila de lista clickeable */
.list-row {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 9px 14px;
  border: none;
  border-bottom: 1px solid var(--borderSoft);
  background: transparent;
  color: inherit;
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  text-align: left;
  transition: background var(--t-fast);
}

.list-row:hover    { background: var(--bgHover); }
.list-row:last-child { border-bottom: none; }
.list-row.selected { background: var(--accentBg); }

/* Panel de logs / output */
.log-panel {
  background: #18181B;
  color: #A1A1AA;
  font-family: var(--font-mono);
  font-size: 11px;
  border-radius: var(--r-lg);
  padding: 14px;
  min-height: 120px;
  max-height: 260px;
  overflow-y: auto;
  line-height: 1.7;
}

.log-time  { color: #52525B; margin-right: 8px; }
.log-ok    { color: #86EFAC; }
.log-warn  { color: #FDE68A; }
.log-error { color: #FCA5A5; }
```

---

## 9. Inputs & Forms

```css
.input {
  width: 100%;
  padding: 6px 10px;
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-sm);
  color: var(--text);
  font-family: var(--font-base);
  font-size: 11px;
  line-height: 1.4;
  outline: none;
  transition: border-color var(--t-fast), box-shadow var(--t-fast);
}

.input:focus {
  border-color: var(--borderFocus);
  box-shadow: 0 0 0 2.5px rgba(37, 99, 235, 0.10);
}

.input:disabled    { opacity: 0.45; cursor: not-allowed; }
.input::placeholder { color: var(--textFaint); }
.input-mono        { font-family: var(--font-mono); font-size: 12px; }

.textarea { resize: vertical; min-height: 80px; line-height: 1.5; }

/* Stepper */
.stepper { display: flex; align-items: center; gap: 4px; }

.stepper-btn {
  width: 28px;
  height: 28px;
  border: 1px solid var(--border);
  border-radius: var(--r-sm);
  background: var(--bgCard);
  color: var(--text);
  font-size: 15px;
  font-weight: 600;
  font-family: var(--font-mono);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background var(--t-fast);
}
.stepper-btn:hover { background: var(--bgHover); }

.stepper-input {
  width: 48px;
  text-align: center;
  font-size: 13px;
  font-weight: 700;
  font-family: var(--font-mono);
}

/* Zona de carga de archivo */
.upload-zone {
  border: 1px dashed var(--border);
  border-radius: var(--r-lg);
  padding: 26px 16px;
  text-align: center;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  background: var(--bgCardAlt);
  transition: background var(--t-normal), border-color var(--t-normal);
}

.upload-zone:hover,
.upload-zone.drag {
  background: var(--accentBg);
  border-color: rgba(37, 99, 235, 0.35);
}

.upload-zone.loaded {
  border-style: solid;
  border-color: rgba(22, 163, 74, 0.30);
  background: var(--pasteGreen);
}

.upload-icon     { font-size: 26px; line-height: 1; color: var(--textFaint); }
.upload-filename { font-size: 11px; font-weight: 600; color: var(--textSub); word-break: break-all; max-width: 220px; }
.upload-ok       { font-size: 10px; font-weight: 700; color: var(--green); letter-spacing: 0.05em; text-transform: uppercase; }
```

---

## 10. Toolbar

```css
.toolbar {
  display: flex;
  align-items: center;
  padding: 5px 10px;
  border-bottom: 1px solid var(--borderSoft);
  gap: 6px;
  background: var(--bg);
  flex-wrap: wrap;
  flex-shrink: 0;
}

.toolbar-sep {
  width: 1px;
  height: 16px;
  background: var(--border);
  margin: 0 2px;
  flex-shrink: 0;
}

.search-wrap { position: relative; flex: 1; max-width: 220px; }
.search-wrap svg {
  position: absolute;
  left: 7px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  color: var(--textFaint);
}
.search-wrap .input { padding-left: 26px; padding-top: 4px; padding-bottom: 4px; }

.stats-bar {
  display: flex;
  align-items: center;
  padding: 3px 10px;
  border-bottom: 1px solid var(--borderSoft);
  gap: 5px;
  flex-wrap: wrap;
  background: var(--bgCardAlt);
  flex-shrink: 0;
}
```

---

## 11. Badges & Pills — reglas anti-AI-tell

**Qué evitar:** chips con relleno vivo (azul saturado, verde vivo), pills anidadas, docenas de colores distintos en la misma pantalla.

**Regla:** máximo 3 colores semánticos visibles a la vez por herramienta. El resto en gris neutro.

```css
/* ── Chip de métrica (stats-bar) ─── */
.stat-chip {
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
  padding: 2px 8px;
  border: 1px solid var(--border);
  border-radius: var(--r-xs);
  background: var(--bgCard);
}

.stat-chip-val {
  font-family: var(--font-mono);
  font-size: 11px;
  font-weight: 700;
  color: var(--text);            /* near-black, no azul por defecto */
}

.stat-chip-val.blue   { color: var(--blue);   }
.stat-chip-val.green  { color: var(--green);  }
.stat-chip-val.red    { color: var(--red);    }
.stat-chip-val.orange { color: var(--orange); }

.stat-chip-lbl {
  font-size: 9px;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--textFaint);
}

/* ── Stat card (métrica grande) ─── */
.stat-card {
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-md);
  padding: 12px 14px;
}

.stat-card-val {
  font-family: var(--font-mono);
  font-size: 22px;
  font-weight: 800;
  color: var(--text);
  letter-spacing: -0.02em;
  line-height: 1.1;
}

.stat-card-lbl {
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.07em;
  color: var(--textFaint);
  margin-top: 4px;
}

/* ── Badge (conteo inline, tabs) ─── */
/* Pastels desaturados — no chips de color vivo */
.badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 18px;
  height: 17px;
  padding: 0 5px;
  border-radius: 3px;           /* cuadrado, no pill redondeado */
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 700;
  background: var(--pasteGray);
  color: var(--textSub);
  border: 1px solid var(--border);
}

.badge.blue   { background: var(--pasteBlue);   color: var(--blue);   border-color: rgba(37,  99, 235, 0.15); }
.badge.red    { background: var(--pasteRed);     color: var(--red);    border-color: rgba(220,  38,  38, 0.15); }
.badge.green  { background: var(--pasteGreen);   color: var(--green);  border-color: rgba(22,  163,  74, 0.15); }
.badge.orange { background: var(--pasteOrange);  color: var(--orange); border-color: rgba(234,  88,  12, 0.15); }

/* ── Pill (estado, categoría, región) ─── */
/* Forma: borde recto no pill redondeado — diferencia clara vs. badge */
.pill {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  padding: 1px 6px;
  border-radius: var(--r-xs);
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 600;
  border: 1px solid;
  white-space: nowrap;
  letter-spacing: 0.02em;
}

.pill-gray   { background: var(--pasteGray);   color: var(--textSub); border-color: var(--border); }
.pill-blue   { background: var(--pasteBlue);   color: var(--blue);   border-color: rgba(37,  99, 235, 0.20); }
.pill-green  { background: var(--pasteGreen);  color: var(--green);  border-color: rgba(22,  163,  74, 0.20); }
.pill-red    { background: var(--pasteRed);    color: var(--red);    border-color: rgba(220,  38,  38, 0.20); }
.pill-orange { background: var(--pasteOrange); color: var(--orange); border-color: rgba(234,  88,  12, 0.20); }
.pill-yellow { background: var(--pasteYellow); color: var(--yellow); border-color: rgba(202, 138,   4, 0.20); }
.pill-purple { background: var(--pastePurple); color: var(--purple); border-color: rgba(124,  58, 237, 0.20); }

/* Toggle de columna visible */
.col-toggle {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 3px 9px;
  border: 1px solid var(--border);
  border-radius: var(--r-xs);
  font-size: 10px;
  font-weight: 500;
  color: var(--textFaint);
  cursor: pointer;
  user-select: none;
  transition: all var(--t-fast);
}

.col-toggle.active {
  background: var(--pasteBlue);
  border-color: rgba(37, 99, 235, 0.22);
  color: var(--blue);
}
```

---

## 12. Tabla Spreadsheet

```css
.spreadsheet-wrap {
  flex: 1;
  overflow: auto;
  position: relative;
}

.dash-table {
  border-collapse: separate;
  border-spacing: 0;
  width: 100%;
  font-size: 11px;
  font-family: var(--font-mono);
}

.dash-table th,
.dash-table td {
  border-right: 1px solid var(--borderSoft);
  border-bottom: 1px solid var(--borderSoft);
  padding: 0;
  position: relative;
}

/* Fila de letras */
.dash-table thead tr.col-letters-row th {
  background: #F4F4F5;
  color: var(--textFaint);
  font-size: 9px;
  font-weight: 600;
  text-align: center;
  padding: 2px 4px;
  position: sticky;
  top: 0;
  z-index: 22;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.dash-table thead tr.col-letters-row th.col-letter-active {
  background: var(--pasteBlue);
  color: var(--blue);
  font-weight: 700;
}

/* Fila de nombres */
.dash-table thead tr.col-names-row {
  position: sticky;
  top: 18px;
  z-index: 21;
}

.dash-table thead tr.col-names-row th {
  background: #FAFAFA;
  padding: 5px 7px;
  text-align: left;
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.02em;
  white-space: nowrap;
  color: var(--textSub);
  font-family: var(--font-base);
}

.dash-table thead tr.col-names-row th.col-name-active {
  background: var(--pasteBlue);
  color: var(--blue);
}

/* Esquinas sticky */
.row-num-corner-top {
  background: #F4F4F5 !important;
  z-index: 25 !important;
  position: sticky;
  left: 0; top: 0;
  width: 30px;
}

.row-num-corner-bot {
  background: #FAFAFA !important;
  z-index: 25 !important;
  position: sticky;
  left: 0; top: 18px;
  width: 30px;
}

/* Número de fila */
.row-num {
  background: #FAFAFA;
  text-align: center;
  color: var(--textFaint);
  padding: 3px 4px;
  width: 30px;
  position: sticky;
  left: 0;
  z-index: 5;
  font-size: 9px;
  cursor: pointer;
  user-select: none;
  transition: background var(--t-fast), color var(--t-fast);
}

.row-num:hover {
  background: var(--pasteBlue);
  color: var(--blue);
}

/* Body rows */
.dash-table tbody tr:hover { background: rgba(0, 0, 0, 0.016); }

.dash-table tr.row-active td { background: rgba(37, 99, 235, 0.03); }

.dash-table tr.row-active td.row-num {
  background: var(--pasteBlue) !important;
  color: var(--blue);
  font-weight: 700;
}

/* Input en celda */
.dash-table td input {
  width: 100%;
  height: 100%;
  border: none;
  background: transparent;
  color: var(--text);
  padding: 3px 7px;
  outline: none;
  font-family: inherit;
  font-size: inherit;
}

.dash-table td input:focus {
  background: var(--bgCard);
  box-shadow: inset 0 0 0 1.5px var(--borderFocus);
}

/* Duplicados */
.dup-row  { background: rgba(220, 38, 38, 0.03) !important; }
.dup-cell { border-left: 2px solid var(--red) !important; }

/* Acciones de fila */
.row-actions {
  opacity: 0;
  transition: opacity var(--t-fast);
  display: flex;
  align-items: center;
  gap: 1px;
  padding: 2px;
}

.dash-table tr:hover .row-actions { opacity: 1; }

.act-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 2px 5px;
  border-radius: var(--r-xs);
  font-size: 10px;
  color: var(--textSub);
  transition: background var(--t-fast);
}

.act-btn:hover { background: var(--bgHover); }

/* Barra de fórmulas */
.formula-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 10px;
  border-bottom: 1px solid var(--borderSoft);
  background: var(--bgCardAlt);
  flex-shrink: 0;
}

.formula-bar-fx {
  font-family: var(--font-mono);
  font-size: 10px;
  color: var(--textFaint);
  font-style: italic;
  padding: 0 4px;
}

.formula-bar-input {
  flex: 1;
  background: transparent;
  border: 1px solid transparent;
  color: var(--text);
  padding: 3px 7px;
  border-radius: var(--r-xs);
  font-family: var(--font-mono);
  font-size: 11px;
  outline: none;
  min-width: 0;
  transition: background var(--t-fast), border-color var(--t-fast);
}

.formula-bar-input:focus {
  background: var(--bgCard);
  border-color: var(--borderFocus);
}

.cell-name-box {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 70px;
  height: 22px;
  padding: 0 8px;
  border-radius: var(--r-xs);
  border: 1px solid var(--border);
  background: var(--bgCard);
  color: var(--textSub);
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.04em;
}

/* Sort / filter en header */
.th-inner { display: flex; align-items: center; justify-content: space-between; gap: 3px; }

.filter-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 1px 3px;
  color: var(--textFaint);
  font-size: 9px;
  line-height: 1;
  transition: color var(--t-fast);
}

.filter-btn:hover, .filter-btn.active { color: var(--text); }

/* Dropdown de filtros */
.filter-dropdown {
  position: absolute;
  top: calc(100% + 2px);
  right: 0;
  min-width: 190px;
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-lg);
  box-shadow: var(--shadow-md);
  z-index: 300;
  overflow: hidden;
  animation: fadeInDown 0.13s ease;
}

.filter-dd-header {
  padding: 7px;
  border-bottom: 1px solid var(--borderSoft);
  display: flex;
  flex-direction: column;
  gap: 4px;
}
```

---

## 13. Modal

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.22);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 500;
  opacity: 0;
  pointer-events: none;
  transition: opacity var(--t-normal);
}

.modal-overlay.active {
  opacity: 1;
  pointer-events: auto;
}

.modal-content {
  background: var(--bgCard);
  border: 1px solid var(--border);
  border-radius: var(--r-xl);
  width: 100%;
  max-width: 520px;
  padding: 22px;
  position: relative;
  transform: scale(0.97) translateY(6px);
  transition: transform var(--t-slow);
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: var(--shadow-xl);
}

.modal-overlay.active .modal-content {
  transform: scale(1) translateY(0);
}

.modal-title {
  font-size: 14px;
  font-weight: 700;
  letter-spacing: -0.01em;
  margin-bottom: 16px;
}

.modal-close {
  position: absolute;
  top: 12px;
  right: 12px;
  background: none;
  border: none;
  color: var(--textFaint);
  cursor: pointer;
  font-size: 16px;
  line-height: 1;
  padding: 4px;
  border-radius: var(--r-xs);
  transition: color var(--t-fast), background var(--t-fast);
}

.modal-close:hover { color: var(--text); background: var(--bgHover); }
```

---

## 14. Toast

```css
.toast {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%) translateY(80px);
  background: #18181B;
  color: #F4F4F5;
  border: 1px solid rgba(255, 255, 255, 0.08);
  padding: 8px 16px;
  border-radius: var(--r-md);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.01em;
  box-shadow: var(--shadow-lg);
  z-index: 2000;
  white-space: nowrap;
  pointer-events: none;
  transition: transform var(--t-slow);
}

.toast.show          { transform: translateX(-50%) translateY(0); }
.toast.toast-ok      { background: #14532D; border-color: rgba(255,255,255,0.08); }
.toast.toast-error   { background: #7F1D1D; border-color: rgba(255,255,255,0.08); }
.toast.toast-warn    { background: #78350F; border-color: rgba(255,255,255,0.08); }
```

```js
function showToast(msg, type = '', ms = 2400) {
  const t = document.querySelector('.toast');
  t.textContent = msg;
  t.className = 'toast show' + (type ? ' toast-' + type : '');
  clearTimeout(t._tid);
  t._tid = setTimeout(() => t.className = 'toast', ms);
}
```

---

## 15. Guide / Accordion

```css
.guide {
  border-bottom: 1px solid var(--border);  /* solo línea inferior, no caja */
  margin-bottom: 0;
}

.guide:first-child { border-top: 1px solid var(--border); }

.guide-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  cursor: pointer;
  user-select: none;
  transition: color var(--t-fast);
}

.guide-head:hover .guide-title { color: var(--text); }

.guide-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
  font-weight: 600;
  color: var(--textSub);
  transition: color var(--t-fast);
}

/* Toggle +/- en monospace, limpio */
.guide-toggle {
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 400;
  color: var(--textFaint);
  line-height: 1;
  transition: color var(--t-fast);
}

.guide:not(.open) .guide-toggle::before { content: "+"; }
.guide.open .guide-toggle::before       { content: "−"; }

.guide-body {
  display: none;
  padding: 0 0 12px 0;
}

.guide.open .guide-body {
  display: block;
  animation: fadeIn 0.16s ease;
}

.guide-step {
  display: flex;
  gap: 10px;
  align-items: flex-start;
  font-size: 11px;
  line-height: 1.6;
  color: var(--textSub);
  margin-bottom: 7px;
}

.guide-step:last-child { margin-bottom: 0; }

/* Número de paso — tipografía, no caja */
.guide-step-num {
  flex-shrink: 0;
  font-family: var(--font-mono);
  font-size: 10px;
  font-weight: 700;
  color: var(--textFaint);
  margin-top: 2px;
  min-width: 18px;
}
```

---

## 16. Empty State

```css
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  min-height: 200px;
  gap: 10px;
  padding: 40px;
  text-align: center;
}

.empty-icon {
  width: 40px;
  height: 40px;
  border-radius: var(--r-md);
  background: var(--bgCardAlt);
  border: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--textFaint);
}

.empty-title { font-size: 13px; font-weight: 600; color: var(--textSub); }
.empty-body  { font-size: 11px; color: var(--textFaint); max-width: 260px; }
```

---

## 17. Loading & Progress

```css
/* Spinner */
.spinner {
  display: inline-block;
  width: 14px;
  height: 14px;
  border: 2px solid var(--borderSoft);
  border-top-color: var(--text);   /* near-black, no azul */
  border-radius: 50%;
  animation: spin 0.65s linear infinite;
}

.spinner-sm { width: 10px; height: 10px; border-width: 1.5px; }
.spinner-lg { width: 20px; height: 20px; border-width: 2.5px; }

/* Barra de progreso */
.progress-bar {
  width: 100%;
  height: 2px;                     /* ultra-fina */
  background: var(--borderSoft);
  border-radius: 1px;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  background: var(--text);         /* near-black por defecto */
  border-radius: 1px;
  transition: width 0.3s ease;
}

.progress-bar-fill.blue   { background: var(--blue); }
.progress-bar-fill.green  { background: var(--green); }
.progress-bar-fill.orange { background: var(--orange); }

/* Skeleton shimmer */
.skeleton {
  background: linear-gradient(90deg,
    var(--bgCardAlt) 25%,
    rgba(0,0,0,0.05) 50%,
    var(--bgCardAlt) 75%
  );
  background-size: 200% 100%;
  border-radius: var(--r-xs);
  animation: shimmer 1.5s ease infinite;
}
```

---

## 18. Paneles auxiliares (replace-bar, col-panel)

```css
.replace-bar {
  display: none;
  align-items: center;
  padding: 5px 10px;
  border-bottom: 1px solid var(--borderSoft);
  gap: 8px;
  background: var(--pasteBlue);
  flex-shrink: 0;
}
.replace-bar.visible { display: flex; }

.col-panel {
  display: none;
  flex-wrap: wrap;
  gap: 5px;
  padding: 6px 10px;
  border-bottom: 1px solid var(--borderSoft);
  background: var(--bgCard);
  flex-shrink: 0;
}
.col-panel.visible { display: flex; }
```

---

## 19. Animaciones

```css
@keyframes fadeIn     { from { opacity: 0; }                      to { opacity: 1; } }
@keyframes fadeInDown { from { opacity: 0; transform: translateY(-6px); } to { opacity: 1; transform: translateY(0); } }
@keyframes fadeInUp   { from { opacity: 0; transform: translateY(6px);  } to { opacity: 1; transform: translateY(0); } }
@keyframes popIn      { 0% { opacity: 0; transform: scale(0.95); } 100% { opacity: 1; transform: scale(1); } }
@keyframes spin       { to { transform: rotate(360deg); } }
@keyframes shimmer    { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }

/* Clases de entrada */
.anim-fade-in      { animation: fadeIn      0.18s ease; }
.anim-fade-in-down { animation: fadeInDown  0.16s ease; }
.anim-fade-in-up   { animation: fadeInUp    0.16s ease; }
.anim-pop-in       { animation: popIn       0.22s cubic-bezier(0.16, 1, 0.3, 1); }
```

---

## 20. Utilidades

```css
/* Visibilidad */
.hidden  { display: none !important; }
.visible { display: flex !important; }

/* Flex helpers */
.flex             { display: flex; }
.flex-col         { display: flex; flex-direction: column; }
.items-center     { align-items: center; }
.justify-between  { justify-content: space-between; }
.flex-1           { flex: 1; }

/* Gaps */
.gap-xs { gap: 4px; }
.gap-sm { gap: 6px; }
.gap-md { gap: 10px; }
.gap-lg { gap: 14px; }
.gap-xl { gap: 20px; }

/* Padding */
.p-sm { padding: 10px 12px; }
.p-md { padding: 14px 16px; }
.p-lg { padding: 18px 20px; }

/* Separador */
.divider { width: 100%; height: 1px; background: var(--borderSoft); margin: 10px 0; }

.truncate { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.w-full   { width: 100%; }
```

---

## 21. Anti-AI-tell — checklist para cada herramienta

Antes de dar por terminada la maquetación de cualquier tool, verifica:

- [ ] **Tipografía:** usa `Plus Jakarta Sans` (o el stack de fallback) — no Segoe UI como única opción.
- [ ] **Botón primario:** `#18181B` near-black, no azul brillante.
- [ ] **Badges/pills:** fills pastel desaturados (`--paste*`), nunca colores vivos saturados.
- [ ] **Tab activo:** subrayado `--text` (near-black), no fondo azul sólido.
- [ ] **Stat chips:** el valor en mono near-black por defecto; solo usa color cuando hay semántica real.
- [ ] **Sombras:** `shadow-xs` o `shadow-sm` como máximo — sin `shadow-lg` en cards.
- [ ] **Borders:** `rgba(0,0,0,0.06–0.09)` neutro — nunca azul-tintado.
- [ ] **Máx. 3 colores semánticos visibles** simultáneamente por pantalla.
- [ ] **Animaciones:** modal con `scale + translateY`, dropdown con `fadeInDown`, toast con `--t-slow`. Nada más.
- [ ] **Guide accordion:** solo `border-bottom`, no caja con fondo coloreado.

---

## 22. Guía de aplicación por herramienta

| Herramienta       | CDN font | Componentes clave |
|-------------------|----------|-------------------|
| RUTEOPM           | sí       | btn, header+tabs, toolbar, stats-bar, stat-chip, spreadsheet, formula-bar, modal, toast, replace-bar, col-panel |
| RUTEOHD           | sí       | btn, header+tabs, card, upload-zone, input, grid-2, modal, toast |
| RUTEO24HRS        | sí       | btn, header+tabs, card, upload-zone, input, stat-card, modal, toast |
| GENERARRESUMEN    | sí       | btn, header+tabs, card, upload-zone, input, textarea, dash-table, toast, empty-state |
| PREOLAREGIONES    | sí       | btn, header+tabs, layout-sidebar, card, upload-zone, input, stepper, stat-card, list-row, pill, modal, toast, guide |
| ADHERENCIACSV     | sí       | btn, header+tabs, card, upload-zone, input, dash-table, stat-chip, toast |
| ACTUALIZAROTIF    | sí       | btn, card, upload-zone, stat-card, toast |
| REPORTERUTAS      | —        | usa React+Tailwind CDN — sistema no aplica directamente |

### Pasos para migrar una herramienta existente

1. Añade el `<link>` de Google Fonts (sección 0) antes del `<style>`.
2. Reemplaza el bloque `:root` completo con los tokens de la sección 1.
3. Actualiza el reset y body (sección 2).
4. Copia scrollbar (3) y tipografía (4).
5. Reemplaza los estilos de los componentes que usa, copiando desde este documento.
6. Cambia `.btn-primary` de azul a near-black donde sea la acción principal.
7. Pasa todas las clases de `.tab-badge` y `.pill-*` a los nuevos pastels.
8. Ejecuta el checklist de la sección 21.

---

## 23. Escalas de referencia

### Espaciado
```
4px · 6px · 8px · 10px · 12px · 14px · 16px · 18px · 20px · 24px
```

### Font-size
```
9px  — letras de columna, etiquetas XS
10px — field-label, texto auxiliar, badge
11px — UI base, botones, celdas de tabla
12px — body de inputs, texto corriente
13px — subheadings
14px — heading de herramienta
17px — h1 de pantalla de configuración
22px — valor grande en stat-card
```

### Z-index
```
1–4   contenido normal
5–25  sticky (row-num, headers de tabla)
100   header de página
300   dropdowns
500   modals y overlays
2000  toast
```

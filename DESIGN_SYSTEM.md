# Design System - IKEA Logistics Tools

## Overview

Sistema de diseño para herramientas internas de gestión logística (RUTEOPM, GENERARRESUMEN, RUTEO24HRS, etc.). Interfaz densa tipo spreadsheet, orientada a productividad y datos en tiempo real.

**Principios:**
- Funcionalidad sobre decoración
- Consistencia en componentes
- Accesibilidad y contraste
- Soporte light/dark mode
- Densidad de datos optimizada

---

## 1. Tokens CSS

### Light Mode (default)

```css
:root {
  /* Backgrounds */
  --bg: #f1f5f9;
  --bgCard: #ffffff;
  --bgCardAlt: rgba(0, 0, 0, 0.025);
  
  /* Text */
  --text: #1a2e42;
  --textSub: #3d6080;
  --textFaint: #5c7c98;
  
  /* Borders & Focus */
  --borderSoft: rgba(58, 114, 172, 0.18);
  --borderFocus: #2563eb;
  
  /* Colors */
  --blue: #2563eb;
  --red: #b91c1c;
  --green: #15803d;
  --orange: #ea580c;
  --purple: #7c3aed;
  --accentBg: rgba(37, 99, 235, 0.08);
  
  /* Typography */
  --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  --font-base: "Segoe UI", system-ui, -apple-system, sans-serif;
  
  /* Sizing */
  --radius-sm: 3px;
  --radius-md: 4px;
  --radius-lg: 6px;
  --radius-xl: 8px;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.03), 0 1px 3px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.12), 0 8px 10px rgba(0, 0, 0, 0.06);
  
  /* Transitions */
  --transition-fast: 0.12s ease;
  --transition-normal: 0.18s ease;
}
```

### Dark Mode

Para agregar soporte dark mode, envuelve los tokens en:

```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #0d1b29;
    --bgCard: #152030;
    --bgCardAlt: rgba(255, 255, 255, 0.018);
    --text: #b8cede;
    --textSub: #8ab0cc;
    --textFaint: #5c7c98;
    --borderSoft: rgba(55, 95, 148, 0.22);
    --borderFocus: #4a84c8;
    --blue: #3a72ac;
    --red: #9a3535;
    --green: #2a6548;
    --orange: #c8703e;
    --purple: #6d5a99;
    --accentBg: rgba(58, 114, 172, 0.11);
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3), 0 1px 3px rgba(0, 0, 0, 0.2);
    --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.4), 0 2px 4px rgba(0, 0, 0, 0.2);
    --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.5), 0 4px 6px rgba(0, 0, 0, 0.2);
    --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.6), 0 8px 10px rgba(0, 0, 0, 0.3);
  }
}
```

---

## 2. Base Styles

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
}

body {
  font-family: var(--font-base);
  background: var(--bg);
  color: var(--text);
  font-size: 12px;
  line-height: 1.5;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
```

---

## 3. Typography

### Escalas de tamaño

```css
/* Display */
h1 { font-size: 18px; font-weight: 700; }
h2 { font-size: 16px; font-weight: 700; }
h3 { font-size: 14px; font-weight: 700; }

/* Body */
body, p { font-size: 12px; color: var(--text); }
.text-sm { font-size: 11px; color: var(--textSub); }
.text-xs { font-size: 10px; color: var(--textFaint); }

/* Monospace */
code, .code { 
  font-family: var(--font-mono);
  font-size: 11px;
}
```

---

## 4. Componentes

### 4.1 Buttons

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  padding: 6px 12px;
  border-radius: var(--radius-md);
  font-weight: 500;
  font-size: 11px;
  cursor: pointer;
  border: 1px solid transparent;
  outline: none;
  white-space: nowrap;
  transition: background var(--transition-fast), 
              border-color var(--transition-fast),
              transform 0.1s ease;
}

.btn:hover:not(:disabled) {
  transform: translateY(-1px);
}

.btn:active:not(:disabled) {
  transform: translateY(1px);
}

.btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

/* Variants */
.btn-primary {
  background: var(--blue);
  color: #fff;
  border-color: rgba(0, 0, 0, 0.15);
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-secondary {
  background: rgba(0, 0, 0, 0.04);
  color: var(--text);
  border-color: var(--borderSoft);
}

.btn-secondary:hover:not(:disabled) {
  background: rgba(0, 0, 0, 0.08);
}

.btn-danger {
  background: rgba(220, 38, 38, 0.1);
  color: #dc2626;
  border-color: rgba(220, 38, 38, 0.25);
}

.btn-danger:hover:not(:disabled) {
  background: rgba(220, 38, 38, 0.18);
}

.btn-green {
  background: rgba(22, 163, 74, 0.1);
  color: #16a34a;
  border-color: rgba(22, 163, 74, 0.25);
}

.btn-green:hover:not(:disabled) {
  background: rgba(22, 163, 74, 0.18);
}

.btn-purple {
  background: rgba(124, 58, 237, 0.1);
  color: var(--purple);
  border-color: rgba(124, 58, 237, 0.25);
}

.btn-purple:hover:not(:disabled) {
  background: rgba(124, 58, 237, 0.18);
}
```

### 4.2 Header & Navigation

```css
.header {
  background: var(--bgCard);
  border-bottom: 1px solid var(--borderSoft);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  z-index: 100;
}

.header-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 14px;
  border-bottom: 1px solid var(--borderSoft);
}

.logo-text {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: var(--textSub);
  text-transform: uppercase;
}

.tabs-container {
  display: flex;
  padding: 0 6px;
}

.tab-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 8px 13px;
  border-radius: 4px 4px 0 0;
  border: none;
  border-bottom: 2px solid transparent;
  background: transparent;
  color: var(--textFaint);
  font-size: 11px;
  font-weight: 500;
  cursor: pointer;
  transition: all var(--transition-fast);
  margin-bottom: -1px;
}

.tab-btn:hover {
  color: var(--textSub);
  background: rgba(0, 0, 0, 0.03);
}

.tab-btn.active {
  color: var(--blue);
  font-weight: 600;
  border-bottom-color: var(--blue);
  background: var(--accentBg);
}

.tab-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 1px 5px;
  border-radius: 2px;
  background: rgba(37, 99, 235, 0.15);
  color: var(--blue);
}

.tab-badge-red {
  background: rgba(220, 38, 38, 0.15);
  color: #dc2626;
}
```

### 4.3 Cards & Panels

```css
.card {
  background: var(--bgCard);
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-lg);
  padding: 18px;
  box-shadow: var(--shadow-sm);
  transition: all var(--transition-normal);
}

.card:hover {
  box-shadow: var(--shadow-md);
  border-color: rgba(37, 99, 235, 0.3);
}

.module-card {
  cursor: pointer;
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-lg);
  padding: 18px;
  background: var(--bgCard);
  display: flex;
  flex-direction: column;
  gap: 10px;
  box-shadow: var(--shadow-sm);
  transition: all var(--transition-normal);
}

.module-card:hover {
  border-color: rgba(37, 99, 235, 0.4);
  background: var(--bgCard);
  box-shadow: var(--shadow-md);
}

.param-section {
  background: var(--bgCard);
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-lg);
  overflow: hidden;
  margin-bottom: 10px;
  box-shadow: var(--shadow-sm);
}

.param-section-header {
  padding: 12px 15px;
  border-bottom: 1px solid var(--borderSoft);
  display: flex;
  align-items: flex-start;
  gap: 10px;
  background: var(--bgCardAlt);
}
```

### 4.4 Inputs & Forms

```css
.input {
  width: 100%;
  padding: 6px 10px;
  background: var(--bgCard);
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-md);
  color: var(--text);
  font-size: 11px;
  outline: none;
  transition: border-color var(--transition-fast);
  box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.02);
}

.input:focus {
  border-color: var(--borderFocus);
  box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.02),
              0 0 0 2px rgba(37, 99, 235, 0.1);
}

.input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.textarea {
  resize: vertical;
  min-height: 80px;
}

.file-drop {
  border: 1px dashed var(--borderSoft);
  border-radius: var(--radius-lg);
  padding: 16px;
  text-align: center;
  cursor: pointer;
  transition: all var(--transition-normal);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 7px;
}

.file-drop:hover {
  background: rgba(37, 99, 235, 0.03);
  border-color: rgba(37, 99, 235, 0.4);
}
```

### 4.5 Modal

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 500;
  opacity: 0;
  pointer-events: none;
  transition: opacity var(--transition-normal);
}

.modal-overlay.active {
  opacity: 1;
  pointer-events: auto;
}

.modal-content {
  background: var(--bgCard);
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-xl);
  width: 100%;
  max-width: 520px;
  padding: 22px;
  position: relative;
  transform: scale(0.97);
  transition: transform var(--transition-normal);
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: var(--shadow-xl);
}

.modal-overlay.active .modal-content {
  transform: scale(1);
}

.modal-close {
  position: absolute;
  top: 11px;
  right: 11px;
  background: none;
  border: none;
  color: var(--textFaint);
  cursor: pointer;
  font-size: 18px;
  padding: 3px;
  transition: color var(--transition-fast);
}

.modal-close:hover {
  color: var(--text);
}
```

### 4.6 Table

```css
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

.dash-table thead tr.col-letters-row th {
  background: #e2e8f0;
  color: var(--textSub);
  font-size: 9px;
  font-weight: 600;
  text-align: center;
  padding: 4px 4px;
  position: sticky;
  top: 0;
  z-index: 22;
  font-family: var(--font-mono);
  letter-spacing: 0.04em;
  text-transform: uppercase;
}

.dash-table thead tr.col-letters-row th.col-letter-active {
  background: rgba(37, 99, 235, 0.15);
  color: var(--blue);
  font-weight: 700;
}

.dash-table thead tr.col-names-row {
  position: sticky;
  top: 20px;
  z-index: 21;
}

.dash-table thead tr.col-names-row th {
  background: #f1f5f9;
  padding: 6px 6px;
  text-align: left;
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.03em;
  white-space: nowrap;
  color: var(--textSub);
}

.dash-table thead tr.col-names-row th.col-name-active {
  background: #e0e7ff;
  color: var(--blue);
}

.dash-table tbody tr:hover {
  background: rgba(0, 0, 0, 0.02);
}

.dash-table tr.row-active td {
  background: rgba(37, 99, 235, 0.04);
}

.dash-table tr.row-active td.row-num {
  background: rgba(37, 99, 235, 0.15) !important;
  color: var(--blue);
  font-weight: 700;
}

.dash-table td input {
  width: 100%;
  height: 100%;
  border: none;
  background: transparent;
  color: var(--text);
  padding: 3px 6px;
  outline: none;
  font-family: inherit;
  font-size: inherit;
}

.dash-table td input:focus {
  background: var(--bgCard);
  box-shadow: inset 0 0 0 1.5px var(--borderFocus);
}

.row-num {
  background: #f8fafc;
  text-align: center;
  color: var(--textFaint);
  padding: 3px 4px;
  width: 32px;
  position: sticky;
  left: 0;
  z-index: 5;
  font-size: 9px;
  cursor: pointer;
  user-select: none;
}

.row-num:hover {
  background: rgba(37, 99, 235, 0.1);
  color: var(--blue);
}

.dup-row {
  background: rgba(220, 38, 38, 0.05) !important;
}

.dup-cell {
  border-left: 2px solid #dc2626 !important;
}
```

### 4.7 Toast

```css
.toast {
  position: fixed;
  bottom: 18px;
  left: 50%;
  transform: translateX(-50%) translateY(80px);
  background: var(--bgCard);
  color: var(--text);
  border: 1px solid var(--borderSoft);
  padding: 8px 18px;
  border-radius: var(--radius-md);
  font-weight: 600;
  font-size: 11px;
  box-shadow: var(--shadow-lg);
  transition: transform var(--transition-normal);
  z-index: 2000;
  white-space: nowrap;
}

.toast.show {
  transform: translateX(-50%) translateY(0);
}
```

### 4.8 Guide / Accordion

```css
.guide {
  border: 1px solid var(--borderSoft);
  border-radius: var(--radius-lg);
  background: var(--bgCardAlt);
  margin-bottom: 16px;
  overflow: hidden;
}

.guide-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 14px;
  cursor: pointer;
  border-bottom: 1px solid transparent;
  transition: background var(--transition-fast);
  user-select: none;
}

.guide-head:hover {
  background: rgba(0, 0, 0, 0.03);
}

.guide.open .guide-head {
  border-bottom-color: var(--borderSoft);
}

.guide-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 10px;
  font-weight: 700;
  color: var(--textSub);
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.guide-title::before {
  content: "i";
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: rgba(37, 99, 235, 0.15);
  color: var(--blue);
  font-size: 9px;
  font-weight: 700;
  font-family: var(--font-mono);
}

.guide-toggle {
  font-family: var(--font-mono);
  color: var(--textFaint);
  font-size: 14px;
  font-weight: 600;
  line-height: 1;
}

.guide:not(.open) .guide-toggle::before {
  content: "+";
}

.guide.open .guide-toggle::before {
  content: "−";
}

.guide-body {
  display: none;
  padding: 12px 16px 14px;
}

.guide.open .guide-body {
  display: block;
}

.guide-step {
  display: flex;
  gap: 10px;
  align-items: flex-start;
  font-size: 11px;
  line-height: 1.55;
  color: var(--textSub);
}

.guide-step-num {
  flex-shrink: 0;
  width: 18px;
  height: 18px;
  border-radius: var(--radius-sm);
  background: rgba(37, 99, 235, 0.1);
  border: 1px solid rgba(37, 99, 235, 0.25);
  color: var(--blue);
  font-size: 10px;
  font-weight: 700;
  font-family: var(--font-mono);
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  margin-top: 1px;
}
```

### 4.9 Stats & Chips

```css
.stat-chip {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 2px 8px;
  border-radius: var(--radius-sm);
  border: 1px solid var(--borderSoft);
  background: var(--bgCard);
}

.stat-chip-val {
  color: var(--blue);
  font-weight: 700;
  font-family: var(--font-mono);
  font-size: 11px;
}

.stat-chip-lbl {
  font-size: 9px;
  text-transform: uppercase;
  color: var(--textFaint);
  letter-spacing: 0.04em;
}
```

### 4.10 Empty State

```css
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  gap: 12px;
  padding: 40px;
  text-align: center;
}

.empty-icon {
  width: 44px;
  height: 44px;
  border-radius: var(--radius-md);
  background: var(--accentBg);
  border: 1px solid rgba(37, 99, 235, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.6;
  color: var(--blue);
}
```

### 4.11 Scrollbar

```css
.custom-scrollbar::-webkit-scrollbar {
  width: 5px;
  height: 5px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.15);
  border-radius: 3px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.25);
}
```

---

## 5. Utility Classes

```css
.hidden {
  display: none !important;
}

.visible {
  display: flex !important;
}

.flex {
  display: flex;
}

.grid {
  display: grid;
}

.grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 18px;
}

.gap-sm { gap: 5px; }
.gap-md { gap: 8px; }
.gap-lg { gap: 12px; }
.gap-xl { gap: 18px; }
```

---

## 6. Cómo aplicar a otros archivos

### Paso 1: Copiar tokens CSS
Reemplaza la sección `:root` de tu archivo con los tokens de arriba.

### Paso 2: Copiar estilos base
Copia todos los estilos de "Base Styles" al inicio del `<style>`.

### Paso 3: Copiar componentes
Copia solo las clases que uses en tu HTML (no necesitas todas).

### Paso 4: Dark mode (opcional)
Si quieres agregar dark mode, envuelve los tokens en la media query:
```css
@media (prefers-color-scheme: dark) { /* dark tokens */ }
```

O usa `data-theme="dark"` en el `<html>`:
```html
<html data-theme="dark">
```

Y luego:
```css
:root[data-theme="dark"] {
  /* dark tokens */
}
```

---

## 7. Variables usadas en el proyecto

### Espaciado estándar
- Small: `5px`, `6px`
- Medium: `10px`, `12px`
- Large: `14px`, `16px`, `18px`
- Extra Large: `20px`, `22px`

### Font sizes
- XS: `9px`
- SM: `10px`
- BASE: `11px`, `12px`
- MD: `13px`, `14px`
- LG: `16px`, `18px`

### Z-index scale
- Base: 1
- Sticky elements: 5-25
- Modals: 500
- Toast: 2000

---

## 8. Notas de implementación

1. **Preservar HTML:** No cambies la estructura HTML, solo CSS
2. **Selectors:** Usa las clases documentadas, evita IDs
3. **JavaScript:** Los estilos dinámicos (hover, active, etc.) son gestionados por JS
4. **Breakpoints:** Este design system es mobile-responsive; preserva los `display: flex/grid` existentes
5. **Accesibilidad:** Mantén los colores con suficiente contraste (WCAG AA)

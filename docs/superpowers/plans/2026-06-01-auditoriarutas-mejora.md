# Mejora de uso AUDITORIARUTAS — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convertir `AUDITORIARUTAS.html` en una herramienta de auditoría principalmente visual, con parámetros configurables, severidad automática de conflictos (🟡/🔴), tabla de resumen en blanco y negro con filtros tipo Excel y exportación selectiva, navegación fluida y guía integrada.

**Architecture:** Un único archivo HTML autocontenido (JS vanilla, CSS inline). Se conserva el motor geo (`pointInPolygon`, `comunaFromCoords`, `haversine`, `comunas_data.js`), Leaflet y SheetJS. Los cambios son: estado de parámetros nuevo (persistido en `localStorage` `aud_*_v3`), reescritura de varias funciones de render, y CSS adicional. Sin build, sin dependencias nuevas.

**Tech Stack:** HTML + CSS inline + JavaScript vanilla; Leaflet 1.9.4, SheetJS 0.18.5, lz-string 1.5.0 (todas ya vía CDN pinneada); `comunas_data.js` local.

**Spec:** `docs/superpowers/specs/2026-06-01-auditoriarutas-mejora-design.md`

---

## Notas de verificación (sin test runner)

No hay framework de tests. Cada tarea se verifica **abriendo `AUDITORIARUTAS.html` en el navegador** (doble clic o `start AUDITORIARUTAS.html`) y comprobando comportamiento. Necesitarás un archivo de plan de prueba (XLSX/CSV con columnas Título, Vehículo, Dirección y, para geo, Latitud/Longitud). Mantén la consola del navegador abierta (F12) para detectar errores JS. Cada tarea termina con un commit.

**Importante (preferencia del usuario):** NO añadir trailers `Co-Authored-By: Claude` ni "Generated with Claude Code" a los commits.

## File Structure

- **Modify:** `misc-projects/AUDITORIARUTAS.html` — único archivo de la herramienta. Todos los cambios viven aquí (HTML en `<body>`, CSS en `<style>`, JS en `<script>`).

No se crean archivos nuevos. El archivo es grande (~1560 líneas) pero es el patrón del repo (un tool = un archivo); no se divide.

---

## Task 1: Estado de parámetros + persistencia v3

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (bloque STATE ~líneas 519-541; PERSISTENCE ~546-572)

- [ ] **Step 1: Añadir el objeto `params` al estado**

En el bloque STATE, tras `let excludedVehicles = new Set()` (línea ~523), añade:

```js
// Parámetros configurables (persistidos en aud_params_v3)
let params = {
  excludedVehicles: ['VPV01','VPV02','VPV03','VPV04'],
  projectVehicles:  ['VPR01','VPR02'],
  borderKm: 0.1,
  borderWeight: 2.2,
  borderOpacity: 0.75,
}
function vehSet(arr){ return new Set((arr||[]).map(v => (v||'').trim().toUpperCase())) }
function isExcludedVeh(veh){ return vehSet(params.excludedVehicles).has((veh||'').toUpperCase()) }
function isProjectVeh(veh){ return vehSet(params.projectVehicles).has((veh||'').toUpperCase()) }
```

- [ ] **Step 2: Persistir y cargar `params`**

En `saveState()` (línea ~546), añade dentro del `try`:

```js
localStorage.setItem('aud_params_v3', JSON.stringify(params))
```

En `loadState()` (línea ~560), añade dentro del `try` (antes del `catch`):

```js
const p = localStorage.getItem('aud_params_v3')
if (p) { try { params = Object.assign(params, JSON.parse(p)) } catch(e){} }
```

- [ ] **Step 3: Reemplazar el uso de `excludedVehicles` (Set poblado por clic) por `params`**

Elimina la línea `let excludedVehicles = new Set()` (la lógica ahora usa `isExcludedVeh`). Busca todos los usos de `excludedVehicles.has(...)` y reemplázalos por `isExcludedVeh(...)`:
- En `getVisibleConflicts()` (~823): `if (isExcludedVeh(c.veh)) return false`
- En `getSummaryRows()` (~831): `if (isExcludedVeh(c.veh)) continue`
- En `updateBadges()` (~920): `conflicts.filter(c => !isExcludedVeh(c.veh) && !resolvedConflicts.has(c.iso))`
- Elimina `excludedVehicles` de `saveState`/`loadState` y de `handleClear`/`processFile` (los reset de `excludedVehicles=new Set()` se eliminan).

- [ ] **Step 4: Verificar en navegador**

Abre el archivo. En consola (F12) ejecuta `params` → debe mostrar el objeto con los defaults. Recarga la página → `params` persiste. Sin errores en consola.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): add configurable params state with v3 persistence"
```

---

## Task 2: Panel ⚙ Parámetros en la pestaña Cargar Plan

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (HTML `tab-plan` ~252-288; JS render nuevo)

- [ ] **Step 1: Añadir el HTML del panel**

Dentro de `#tab-plan`, después del `div#plan-conflicts-info` (línea ~287), inserta:

```html
<details id="params-panel" style="border:1px solid var(--border);border-radius:var(--r-lg);background:var(--bgCard)">
  <summary style="cursor:pointer;padding:12px 16px;font-size:12px;font-weight:700;user-select:none">⚙ Parámetros de auditoría</summary>
  <div style="padding:0 16px 16px;display:flex;flex-direction:column;gap:16px">
    <div>
      <p class="section-label" style="margin-bottom:6px">Vehículos excluidos de auditoría geo</p>
      <div id="param-excluded-chips" class="filter-pills" style="margin-bottom:6px"></div>
      <input class="input" id="param-excluded-input" placeholder="Agregar código y Enter (ej. VPV05)" style="max-width:280px">
    </div>
    <div>
      <p class="section-label" style="margin-bottom:6px">Vehículos de proyecto (van a la pestaña Proyectos)</p>
      <div id="param-project-chips" class="filter-pills" style="margin-bottom:6px"></div>
      <input class="input" id="param-project-input" placeholder="Agregar código y Enter (ej. VPR03)" style="max-width:280px">
    </div>
    <div style="display:flex;gap:20px;flex-wrap:wrap">
      <label style="font-size:11px;color:var(--textSub)">Umbral borde 🟡/🔴 (km)
        <input class="input" id="param-border-km" type="number" step="0.1" min="0" style="width:90px;margin-top:4px">
      </label>
      <label style="font-size:11px;color:var(--textSub)">Grosor borde comuna
        <input class="input" id="param-border-weight" type="number" step="0.1" min="0.5" style="width:90px;margin-top:4px">
      </label>
    </div>
  </div>
</details>
```

- [ ] **Step 2: Añadir el render y los handlers del panel**

En el bloque ROUTE PLAN TAB (que vamos a vaciar en Task 5) o junto a HELPERS, añade:

```js
function renderParamsPanel() {
  const exWrap = document.getElementById('param-excluded-chips')
  const prWrap = document.getElementById('param-project-chips')
  if (!exWrap || !prWrap) return
  const chip = (code, kind) =>
    `<span class="filter-pill active" style="cursor:default">${escH(code)}
      <span onclick="removeParamVeh('${kind}','${escH(code)}')" style="cursor:pointer;margin-left:4px;font-weight:700">✕</span>
    </span>`
  exWrap.innerHTML = params.excludedVehicles.map(c => chip(c,'excluded')).join('') || '<span style="font-size:10px;color:var(--textFaint)">Ninguno</span>'
  prWrap.innerHTML = params.projectVehicles.map(c => chip(c,'project')).join('') || '<span style="font-size:10px;color:var(--textFaint)">Ninguno</span>'
  const km = document.getElementById('param-border-km'); if (km) km.value = params.borderKm
  const w = document.getElementById('param-border-weight'); if (w) w.value = params.borderWeight
}
function addParamVeh(kind, code) {
  code = (code||'').trim().toUpperCase()
  if (!code) return
  const arr = kind==='excluded' ? params.excludedVehicles : params.projectVehicles
  if (!arr.includes(code)) arr.push(code)
  saveState(); renderParamsPanel(); reanalyze()
}
function removeParamVeh(kind, code) {
  const key = kind==='excluded' ? 'excludedVehicles' : 'projectVehicles'
  params[key] = params[key].filter(c => c !== code)
  saveState(); renderParamsPanel(); reanalyze()
}
function reanalyze() {
  if (routeData.length && routeData.some(r=>r.lat!==null)) runGeoAnalysis()
  else updateAllUI()
}
function wireParamsPanel() {
  const exIn = document.getElementById('param-excluded-input')
  const prIn = document.getElementById('param-project-input')
  if (exIn) exIn.addEventListener('keydown', e => { if (e.key==='Enter'){ addParamVeh('excluded', exIn.value); exIn.value='' } })
  if (prIn) prIn.addEventListener('keydown', e => { if (e.key==='Enter'){ addParamVeh('project', prIn.value); prIn.value='' } })
  const km = document.getElementById('param-border-km')
  if (km) km.addEventListener('change', () => { params.borderKm = parseFloat(km.value)||0.1; saveState(); reanalyze() })
  const w = document.getElementById('param-border-weight')
  if (w) w.addEventListener('change', () => { params.borderWeight = parseFloat(w.value)||2.2; saveState(); redrawComunaBorders() })
}
```

`redrawComunaBorders` se define en Task 6 (déjalo referenciado; si aún no existe, créalo como `function redrawComunaBorders(){}` temporal y complétalo en Task 6).

- [ ] **Step 3: Llamar a `renderParamsPanel()` y `wireParamsPanel()` en init**

En `init()` (línea ~1521), tras `updateAllUI()`, añade:

```js
renderParamsPanel()
wireParamsPanel()
```

Y añade `renderParamsPanel()` dentro de `updateAllUI()`.

- [ ] **Step 4: Verificar en navegador**

Abre el archivo → pestaña Cargar Plan → abre "⚙ Parámetros". Debes ver los chips `VPV01..04` y `VPR01/VPR02`, el umbral `0.1`. Agrega `VPV05` con Enter → aparece como chip y persiste al recargar. Quita un chip con ✕. Sin errores en consola.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): add editable parameters panel (excluded/project vehicles, border threshold)"
```

---

## Task 3: Proyectos por vehículo (VPR01/VPR02)

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (`getProyectosData` ~801-804; `runGeoAnalysis` ~707-740)

- [ ] **Step 1: Reescribir `getProyectosData`**

Reemplaza (líneas ~801-804):

```js
function getProyectosData() {
  return routeData.filter(r => isProjectVeh(r.veh))
}
```

- [ ] **Step 2: Excluir vehículos de proyecto de los conflictos geo**

En `runGeoAnalysis()` (línea ~716), cambia el inicio del bucle:

```js
  for (const r of pts) {
    if (isExcludedVeh(r.veh) || isProjectVeh(r.veh)) continue
    const kDir   = normK(r.comuna)
    // ... resto igual
```

- [ ] **Step 3: Verificar en navegador**

Carga un plan que tenga filas con Vehículo `VPR01`/`VPR02` (o agrégalas a tu archivo de prueba). Pestaña Proyectos → deben listarse esas ISOs. Pestaña Auditoría Geo → esas ISOs NO deben generar conflictos. Recuento de Proyectos coincide con las filas VPR01/VPR02.

- [ ] **Step 4: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): detect projects by vehicle code (VPR01/VPR02) instead of driver name"
```

---

## Task 4: Severidad automática 🟡/🔴 (distancia al borde)

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (GEO UTILS ~501-507; `runGeoAnalysis` ~727-734)

- [ ] **Step 1: Añadir distancia punto→polígono**

En GEO UTILS, tras `haversine` (línea ~507), añade:

```js
// Distancia mínima (km) de un punto al borde de un polígono [[lat,lng],...]
function distPointToSegmentKm(p, a, b) {
  // Proyección local: escalar lng por cos(lat) para aproximar metros
  const latRef = (a[0]+b[0])/2 * Math.PI/180
  const kx = 111.32 * Math.cos(latRef)   // km por grado lng
  const ky = 110.57                       // km por grado lat
  const px=(p[1])*kx, py=(p[0])*ky
  const ax=a[1]*kx, ay=a[0]*ky, bx=b[1]*kx, by=b[0]*ky
  const dx=bx-ax, dy=by-ay
  const len2 = dx*dx+dy*dy
  let t = len2 ? ((px-ax)*dx+(py-ay)*dy)/len2 : 0
  t = Math.max(0, Math.min(1, t))
  const cx=ax+t*dx, cy=ay+t*dy
  return Math.hypot(px-cx, py-cy)
}
function distPointToPolygonKm(lat, lng, poly) {
  let min = Infinity
  for (let i=0, j=poly.length-1; i<poly.length; j=i++) {
    const d = distPointToSegmentKm([lat,lng], poly[j], poly[i])
    if (d < min) min = d
  }
  return min
}
```

- [ ] **Step 2: Clasificar severidad en `runGeoAnalysis`**

En el `conflicts.push({...})` (líneas ~727-734), calcula la distancia al borde de la comuna declarada y añade `severity` + `distBordeKm`:

```js
    const cdDir = D[kDir]
    let distBordeKm = Infinity
    if (cdDir && cdDir.p) distBordeKm = distPointToPolygonKm(r.lat, r.lng, cdDir.p)
    const severity = distBordeKm <= (params.borderKm||0.1) ? 'amarilla' : 'roja'
    conflicts.push({
      iso: r.iso, veh: r.veh, dir: r.dir,
      comunaDireccion: r.comuna,
      comunaReal: getDisplayName(kCoord),
      comunaRealKey: kCoord,
      lat: r.lat, lng: r.lng, parada: r.parada,
      severity, distBordeKm: Math.round(distBordeKm*1000)/1000,
      status: 'pendiente',
    })
```

- [ ] **Step 3: Colorear markers por severidad**

En `updateMapMarkers` (línea ~1165), reemplaza el cálculo de `col`:

```js
    const conf = isConf ? confs.find(x=>x.iso===r.iso) : null
    const sevColor = conf && conf.severity==='amarilla' ? '#CA8A04' : '#DC2626'
    const col = isF ? '#FFDA1A' : isConf ? (isRes ? '#16A34A' : sevColor) : getVehColor(r.veh)
```

Y en la clase del pulso (línea ~1167) deja el pulso rojo solo para rojas:

```js
    const cls = isF ? 'mk-focus' : (isConf && !isRes && (!conf || conf.severity==='roja') ? 'mk-conflict' : '')
```

- [ ] **Step 4: Verificar en navegador**

Carga un plan con GPS. En consola: `conflicts.map(c=>[c.iso,c.severity,c.distBordeKm])` → cada conflicto tiene `severity` y distancia. Markers: rojos los lejos, ámbar los a ≤0.1 km del borde. Cambia el umbral en Parámetros a `2` → más conflictos pasan a amarillo y el mapa se actualiza.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): classify geo conflicts by distance to border (yellow/red severity)"
```

---

## Task 5: Eliminar pestañas Route Plan y Exportar

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (tabs ~239,243; panes ~290-308, 408-456; JS render/switch/update)

- [ ] **Step 1: Quitar los botones de pestaña**

Elimina la línea del botón `data-tab="tab-vehiculos"` (línea ~239) y la de `data-tab="tab-export"` (línea ~243).

- [ ] **Step 2: Eliminar los paneles**

Borra el bloque completo `<!-- TAB: ROUTE PLAN -->` (`#tab-vehiculos`, ~290-308) y el bloque `<!-- TAB: EXPORT -->` (`#tab-export`, ~408-456). El contenido de exportación se reubicará en Resumen (Task 9); no lo pierdas: ya está cubierto por las funciones `exportFullPlan`/`exportSession` que se conservan.

- [ ] **Step 3: Limpiar el JS muerto**

- Elimina `renderRoutePlan` (~968-1042), `expandAllVehicles`/`collapseAllVehicles` (~961-966), `toggleExclude` (~1049-1054).
- En `switchTab` (~865) elimina ramas a `tab-vehiculos`/`tab-export` (no existían como `if`, pero quita `if (id === 'tab-export') updateExportTab()` ~874 — `updateExportTab` se moverá a Resumen).
- En `updateAllUI` (~899) elimina la llamada a `renderRoutePlan()`.
- En `updateBadges` (~919) elimina `setBadge('badge-vehiculos', ...)`.
- Conserva `getVehColor`, `copyISO` (se usan en el mapa/otros).

- [ ] **Step 4: Verificar en navegador**

La barra de pestañas muestra 4: Cargar Plan, Wrong Commune (se renombra en Task 8), Proyectos, Summary. No hay errores en consola al cargar un plan ni al cambiar de pestaña.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "refactor(auditoria): remove Route Plan and Export tabs"
```

---

## Task 6: Bordes de comuna más marcados + redibujo

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (`drawComunaPolygons` ~1059-1075; init maps)

- [ ] **Step 1: Usar parámetros de borde y guardar las capas**

Reemplaza `drawComunaPolygons` (~1059-1075):

```js
let comunaBorderLayers = { geo: [], proy: [] }
function drawComunaPolygons(map, which) {
  const D = window.RM_COMUNAS_DATA
  if (!D || !window.L) return
  const L = window.L
  comunaBorderLayers[which] = []
  Object.keys(D).forEach(k => {
    const d = D[k]; if (!d || !d.p) return
    const poly = L.polygon(d.p, { color:'#334155', weight:params.borderWeight, opacity:params.borderOpacity, fillColor:'#334155', fillOpacity:0.05, interactive:false }).addTo(map)
    comunaBorderLayers[which].push(poly)
    if (d.c) {
      const icon = L.divIcon({ className:'', html:`<div class="commune-lbl">${escH(d.n||k)}</div>`, iconSize:[180,24], iconAnchor:[90,12] })
      L.marker(d.c, { icon, interactive:false, zIndexOffset:-100 }).addTo(map)
    }
  })
}
function redrawComunaBorders() {
  ['geo','proy'].forEach(which => {
    const map = which==='geo' ? geoMap : proyMap
    if (!map) return
    comunaBorderLayers[which].forEach(l => map.removeLayer(l))
    drawComunaPolygons(map, which)
  })
}
```

- [ ] **Step 2: Pasar `which` en las llamadas**

En `initGeoMap` (~1083) cambia `drawComunaPolygons(geoMap)` → `drawComunaPolygons(geoMap,'geo')`. En `initProyMap` (~1096) → `drawComunaPolygons(proyMap,'proy')`.

- [ ] **Step 3: Verificar en navegador**

Abre Auditoría Geo → los bordes de comuna se ven claramente más gruesos/oscuros. En Parámetros sube "Grosor borde" a 4 → al volver al mapa los bordes engrosan (redibujo). Las etiquetas de comuna siguen visibles.

- [ ] **Step 4: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): stronger, configurable commune border lines"
```

---

## Task 7: CSS — tablas en blanco y negro + componentes nuevos (taste-skill)

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (`<style>` ~123-128 tablas; añadir bloques nuevos)

- [ ] **Step 1: Reescribir el estilo de `.dash-table` a monocromo**

Reemplaza el bloque Tables (~123-128):

```css
/* Tables — monocromo (blanco y negro) */
.dash-table{border-collapse:separate;border-spacing:0;width:100%;font-size:11px;font-family:var(--font-mono)}
.dash-table th,.dash-table td{border-right:1px solid #E7E7E7;border-bottom:1px solid #E7E7E7;padding:0;position:relative}
.dash-table thead tr th{background:#111;color:#fff;padding:8px 10px;text-align:left;font-size:10px;font-weight:600;letter-spacing:0.03em;white-space:nowrap;font-family:inherit;position:sticky;top:0;z-index:10}
.dash-table tbody tr{transition:background var(--t-fast)}
.dash-table tbody tr:hover td{background:#F4F4F4}
.dash-table td{padding:6px 10px;vertical-align:middle;color:#222}
.dash-table tbody tr.is-selected td{background:#ECECEC}
.sev-mark{font-family:var(--font-mono);font-weight:700}
.sev-roja::before{content:"● Lejos"}
.sev-amarilla::before{content:"○ Borde"}
.st-mark{font-weight:700;font-size:10px}
```

- [ ] **Step 2: Añadir CSS de checkbox, popover de filtro y barra de selección**

Antes de `/* Utilities */` (~205) añade:

```css
/* Filtro de columna (popover) */
.col-filter-btn{background:none;border:none;cursor:pointer;color:#bbb;font-size:10px;margin-left:4px;padding:0 2px}
.col-filter-btn.active{color:#fff}
.col-popover{position:absolute;top:100%;left:0;z-index:50;background:#fff;border:1px solid var(--border);border-radius:var(--r-md);box-shadow:var(--shadow-lg);width:220px;max-height:280px;display:flex;flex-direction:column;padding:8px;gap:6px}
.col-popover .cp-list{overflow-y:auto;display:flex;flex-direction:column;gap:2px}
.col-popover label{display:flex;align-items:center;gap:6px;font-size:11px;color:#222;font-family:inherit;cursor:pointer;padding:2px 4px;border-radius:3px}
.col-popover label:hover{background:#F4F4F4}
.cp-actions{display:flex;gap:6px;justify-content:space-between;border-top:1px solid var(--borderSoft);padding-top:6px}
.row-check{width:14px;height:14px;cursor:pointer;accent-color:#111}
.sel-bar{display:flex;align-items:center;gap:8px;padding:6px 12px;background:#111;color:#fff;font-size:11px}
.sel-bar.hidden{display:none}
@keyframes rowIn{from{opacity:0;transform:translateY(-2px)}to{opacity:1;transform:translateY(0)}}
.dash-table tbody tr{animation:rowIn 0.12s ease}
```

- [ ] **Step 3: Verificar en navegador**

Abre Summary con datos cargados (la tabla actual seguirá con su markup viejo hasta Task 8, pero el encabezado debe verse negro con texto blanco y sin colores en celdas). Sin errores.

- [ ] **Step 4: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "style(auditoria): black & white tables + filter/selection component CSS"
```

---

## Task 8: Resumen — filtros (severidad, vehículo, columna tipo Excel) + selección + exportación selectiva

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (toolbar `#summary-toolbar` ~384-402; `getSummaryRows` ~828-844; `renderSummary` ~1331-1405; export ~1470-1516; estado de filtros)

- [ ] **Step 1: Añadir estado de filtros y selección**

Junto a `let summaryFilter = 'all'` (~529) añade:

```js
let severityFilter = 'all'      // all | roja | amarilla
let vehFilter = 'all'           // 'all' o nombre de vehículo
let summaryColFilters = {}      // { colKey: Set(valoresPermitidos) }
let selectedRows = new Set()    // claves rowKey seleccionadas
let openColPopover = null
function rowKey(r){ return r.tipo + '|' + r.iso }
```

- [ ] **Step 2: Incluir proyectos y severidad en `getSummaryRows`**

Reemplaza `getSummaryRows` (~828-844):

```js
function getSummaryRows() {
  const rows = []
  for (const c of conflicts) {
    if (isExcludedVeh(c.veh)) continue
    rows.push({
      iso:c.iso, veh:c.veh, dir:c.dir,
      obs:'Comuna Incorrecta',
      detalle:`${c.comunaDireccion} → ${c.comunaReal}`,
      severity:c.severity||'roja', tipo:'geo',
      status:getConflictStatus(c.iso),
    })
  }
  for (const r of getProyectosData()) {
    rows.push({
      iso:r.iso, veh:r.veh, dir:r.dir,
      obs:'Proyecto', detalle:r.conductor||'',
      severity:'', tipo:'proy',
      status:getProyStatus(r.iso),
    })
  }
  return rows.sort((a,b)=>{ const o={alerta:0,pendiente:1,resuelto:2}; return (o[a.status]||0)-(o[b.status]||0) })
}
```

- [ ] **Step 3: Reemplazar la toolbar de Summary**

Reemplaza el contenido de `#summary-toolbar` (~384-402) por:

```html
<span style="font-size:11px;color:var(--textSub)" id="summary-info">0 alertas</span>
<div class="toolbar-sep"></div>
<div class="filter-pills" id="summary-status-pills">
  <button class="filter-pill active" onclick="setSummaryFilter('all',this)">Todos</button>
  <button class="filter-pill" onclick="setSummaryFilter('pendiente',this)">⏳ Pendiente</button>
  <button class="filter-pill" onclick="setSummaryFilter('alerta',this)">⚠ Alerta</button>
  <button class="filter-pill" onclick="setSummaryFilter('resuelto',this)">✓ Resuelto</button>
</div>
<div class="toolbar-sep"></div>
<div class="filter-pills" id="summary-sev-pills">
  <button class="filter-pill active" onclick="setSeverityFilter('all',this)">Severidad: todas</button>
  <button class="filter-pill" onclick="setSeverityFilter('roja',this)">🔴 Rojas</button>
  <button class="filter-pill" onclick="setSeverityFilter('amarilla',this)">🟡 Amarillas</button>
</div>
<div class="toolbar-sep"></div>
<select class="input" id="summary-veh" onchange="setVehFilter(this.value)" style="max-width:160px"><option value="all">Todos los vehículos</option></select>
<button class="btn btn-secondary btn-sm" onclick="clearAllFilters()">Limpiar filtros</button>
<div style="position:relative">
  <input class="input" id="summary-search" placeholder="Buscar…" oninput="renderSummary()" style="max-width:160px">
</div>
<div style="margin-left:auto;display:flex;gap:6px">
  <button class="btn btn-blue btn-sm" onclick="copySummaryToClipboard()">📋 Copiar</button>
  <button class="btn btn-green btn-sm" onclick="exportSummaryXLSX()">↓ Exportar</button>
</div>
```

- [ ] **Step 4: Añadir la barra de selección bajo la toolbar**

Justo después de `</div>` de cierre de `#summary-toolbar` y antes de `#summary-wrap` (~403), inserta:

```html
<div class="sel-bar hidden" id="sel-bar">
  <span id="sel-count">0 seleccionadas</span>
  <button class="btn btn-sm" style="background:#fff;color:#111" onclick="clearSelection()">Quitar selección</button>
  <span style="margin-left:auto;font-size:10px;opacity:0.8">La exportación/copia usa la selección; si no hay, usa lo filtrado.</span>
</div>
```

- [ ] **Step 5: Añadir setters de filtro**

Junto a `setSummaryFilter` (~889) añade:

```js
function setSeverityFilter(f, btn){ severityFilter=f; document.querySelectorAll('#summary-sev-pills .filter-pill').forEach(b=>b.classList.remove('active')); if(btn)btn.classList.add('active'); renderSummary() }
function setVehFilter(v){ vehFilter=v; renderSummary() }
function clearAllFilters(){
  summaryFilter='all'; severityFilter='all'; vehFilter='all'; summaryColFilters={}
  const s=document.getElementById('summary-search'); if(s)s.value=''
  document.querySelectorAll('#summary-status-pills .filter-pill').forEach((b,i)=>b.classList.toggle('active',i===0))
  document.querySelectorAll('#summary-sev-pills .filter-pill').forEach((b,i)=>b.classList.toggle('active',i===0))
  const v=document.getElementById('summary-veh'); if(v)v.value='all'
  renderSummary()
}
function clearSelection(){ selectedRows.clear(); renderSummary() }
```

- [ ] **Step 6: Añadir la función central de filas filtradas**

Antes de `renderSummary` añade:

```js
function getFilteredSummaryRows() {
  const q = (document.getElementById('summary-search')?.value||'').toLowerCase()
  return getSummaryRows().filter(r => {
    if (summaryFilter!=='all' && r.status!==summaryFilter) return false
    if (severityFilter!=='all' && r.severity!==severityFilter) return false
    if (vehFilter!=='all' && r.veh!==vehFilter) return false
    for (const col of Object.keys(summaryColFilters)) {
      const allowed = summaryColFilters[col]
      if (allowed && allowed.size && !allowed.has(String(r[col]??''))) return false
    }
    if (q) return r.iso.toLowerCase().includes(q)||r.veh.toLowerCase().includes(q)||(r.dir||'').toLowerCase().includes(q)
    return true
  })
}
```

- [ ] **Step 7: Reescribir `renderSummary` (tabla B&N, checkbox, filtro por columna)**

Reemplaza `renderSummary` (~1331-1405) por:

```js
function renderSummary() {
  const wrap = document.getElementById('summary-wrap')
  if (!wrap) return
  // poblar select de vehículos
  const vehSel = document.getElementById('summary-veh')
  if (vehSel) {
    const vehs = [...new Set(getSummaryRows().map(r=>r.veh))].sort((a,b)=>a.localeCompare(b,'es'))
    const cur = vehFilter
    vehSel.innerHTML = '<option value="all">Todos los vehículos</option>' + vehs.map(v=>`<option value="${escH(v)}">${escH(v)}</option>`).join('')
    vehSel.value = cur
  }
  const all = getSummaryRows()
  const visible = getFilteredSummaryRows()
  const pend = all.filter(r=>r.status==='pendiente').length
  document.getElementById('summary-info').innerHTML = `${all.length} alertas · <span style="font-weight:700">${pend} pendientes</span>`
  // barra de selección
  const selBar = document.getElementById('sel-bar')
  if (selBar) { selBar.classList.toggle('hidden', selectedRows.size===0); const sc=document.getElementById('sel-count'); if(sc) sc.textContent=`${selectedRows.size} seleccionadas` }

  if (!all.length) { wrap.innerHTML='<div class="empty-state"><div class="empty-icon">📊</div><div class="empty-title">Ejecuta el análisis geo para ver el resumen</div></div>'; return }
  if (!visible.length) { wrap.innerHTML='<div class="empty-state"><div class="empty-title">Sin resultados con los filtros actuales</div></div>'; return }

  const COLS = [
    {key:'iso',label:'ISO'},{key:'veh',label:'Vehículo'},{key:'dir',label:'Dirección'},
    {key:'obs',label:'Tipo'},{key:'detalle',label:'Detalle'},{key:'severity',label:'Severidad'},{key:'status',label:'Estado'},
  ]
  const visKeys = visible.map(rowKey)
  const allSelected = visKeys.length && visKeys.every(k=>selectedRows.has(k))

  wrap.innerHTML = `<table class="dash-table" style="table-layout:auto;width:100%">
    <thead><tr>
      <th style="width:30px;text-align:center"><input type="checkbox" class="row-check" ${allSelected?'checked':''} onclick="toggleSelectAll(this.checked)"></th>
      <th style="width:30px;text-align:center">#</th>
      ${COLS.map(c=>{
        const active = summaryColFilters[c.key] && summaryColFilters[c.key].size
        return `<th>${c.label}<button class="col-filter-btn ${active?'active':''}" onclick="toggleColPopover(event,'${c.key}')">▼</button></th>`
      }).join('')}
    </tr></thead>
    <tbody>
    ${visible.map((r,i)=>{
      const k = rowKey(r)
      const sel = selectedRows.has(k)
      const stTxt = r.status==='resuelto'?'✓ Resuelto':r.status==='alerta'?'⚠ Alerta':'⏳ Pendiente'
      const sevTxt = r.severity==='roja'?'● Lejos':r.severity==='amarilla'?'○ Borde':'—'
      return `<tr class="${sel?'is-selected':''}">
        <td style="text-align:center"><input type="checkbox" class="row-check" ${sel?'checked':''} onclick="toggleRowSel('${escH(k)}')"></td>
        <td style="text-align:center;color:#999">${i+1}</td>
        <td><span style="font-weight:700;cursor:pointer" onclick="navigator.clipboard.writeText('${escH(r.iso)}')" title="Copiar">${escH(r.iso)}</span></td>
        <td>${escH(r.veh)}</td>
        <td title="${escH(r.dir)}">${escH(r.dir||'—')}</td>
        <td>${escH(r.obs)}</td>
        <td title="${escH(r.detalle)}">${escH(r.detalle||'—')}</td>
        <td class="sev-mark">${sevTxt}</td>
        <td>
          <span class="st-mark">${stTxt}</span>
          <span style="margin-left:8px;display:inline-flex;gap:3px">
            <button class="btn btn-sm btn-secondary" onclick="summaryResolve('${escH(r.iso)}','${r.tipo}')">${r.status==='resuelto'?'↩':'✓'}</button>
            <button class="btn btn-sm btn-secondary" onclick="summaryFlag('${escH(r.iso)}','${r.tipo}')">${r.status==='alerta'?'✕':'⚠'}</button>
          </span>
        </td>
      </tr>`
    }).join('')}
    </tbody>
  </table>`
}
```

- [ ] **Step 8: Añadir selección y popover de columna**

Añade estas funciones cerca de `renderSummary`:

```js
function toggleRowSel(k){ if(selectedRows.has(k))selectedRows.delete(k); else selectedRows.add(k); renderSummary() }
function toggleSelectAll(check){ const vis=getFilteredSummaryRows().map(rowKey); if(check)vis.forEach(k=>selectedRows.add(k)); else vis.forEach(k=>selectedRows.delete(k)); renderSummary() }
function toggleColPopover(ev, col){
  ev.stopPropagation()
  if (openColPopover) { openColPopover.remove(); openColPopover=null }
  const th = ev.target.closest('th')
  const values = [...new Set(getSummaryRows().map(r=>String(r[col]??'')))].sort((a,b)=>a.localeCompare(b,'es'))
  const sel = summaryColFilters[col] || new Set(values)
  const pop = document.createElement('div')
  pop.className='col-popover'
  pop.innerHTML = `<input class="input" placeholder="Buscar…" oninput="filterPopoverList(this)">
    <div class="cp-list">${values.map(v=>`<label><input type="checkbox" class="row-check cp-cb" value="${escH(v)}" ${sel.has(v)?'checked':''}>${escH(v||'(vacío)')}</label>`).join('')}</div>
    <div class="cp-actions">
      <button class="btn btn-sm btn-secondary" onclick="applyColFilter('${col}',this)">Aplicar</button>
      <button class="btn btn-sm btn-secondary" onclick="resetColFilter('${col}')">Quitar</button>
    </div>`
  th.appendChild(pop)
  pop.addEventListener('click', e=>e.stopPropagation())
  openColPopover = pop
}
function filterPopoverList(input){
  const q=input.value.toLowerCase()
  input.parentElement.querySelectorAll('.cp-list label').forEach(l=>{ l.style.display = l.textContent.toLowerCase().includes(q)?'flex':'none' })
}
function applyColFilter(col, btn){
  const pop = btn.closest('.col-popover')
  const checked = [...pop.querySelectorAll('.cp-cb:checked')].map(c=>c.value)
  const total = pop.querySelectorAll('.cp-cb').length
  if (checked.length===total) delete summaryColFilters[col]
  else summaryColFilters[col] = new Set(checked)
  if (openColPopover){ openColPopover.remove(); openColPopover=null }
  renderSummary()
}
function resetColFilter(col){ delete summaryColFilters[col]; if(openColPopover){openColPopover.remove();openColPopover=null} renderSummary() }
document.addEventListener('click', ()=>{ if(openColPopover){ openColPopover.remove(); openColPopover=null } })
```

- [ ] **Step 9: Exportación/copia respetando la selección**

Reemplaza `exportSummaryXLSX` (~1470) y `copySummaryToClipboard` (~1499) para usar la selección:

```js
function rowsForExport(){
  const vis = getFilteredSummaryRows()
  if (selectedRows.size) return vis.filter(r=>selectedRows.has(rowKey(r)))
  return vis
}
function exportSummaryXLSX() {
  const rows = rowsForExport()
  if (!rows.length) { showToast('Sin filas para exportar','warn'); return }
  const W = getW(); if (!W) return
  const wb = W.utils.book_new()
  const ws = W.utils.aoa_to_sheet([
    ['ISO','Vehículo','Dirección','Tipo','Detalle','Severidad','Estado'],
    ...rows.map(r=>[r.iso,r.veh,r.dir,r.obs,r.detalle, r.severity==='roja'?'Lejos':r.severity==='amarilla'?'Borde':'', r.status])
  ])
  ws['!cols']=[{wch:14},{wch:22},{wch:50},{wch:18},{wch:40},{wch:10},{wch:12}]
  ws['!freeze']={xSplit:0,ySplit:1}
  W.utils.book_append_sheet(wb, ws, 'Alertas')
  W.writeFile(wb, `resumen_alertas_${ts()}.xlsx`)
  showToast(`✓ Exportadas ${rows.length} filas`)
}
function copySummaryToClipboard() {
  const rows = rowsForExport()
  if (!rows.length) { showToast('Sin filas','warn'); return }
  const hS='border:1px solid #ccc;padding:5px 8px;font-weight:700;background:#111;color:#fff;font-size:11px;'
  const tS='border:1px solid #ccc;padding:4px 8px;font-size:11px;'
  const head=['ISO','Vehículo','Dirección','Tipo','Detalle','Severidad','Estado']
  let rich=`<table style="border-collapse:collapse"><tr>${head.map(h=>`<th style="${hS}">${h}</th>`).join('')}</tr>`
  let plain=head.join('\t')+'\n'
  rows.forEach(r=>{
    const sev=r.severity==='roja'?'Lejos':r.severity==='amarilla'?'Borde':''
    const e=r.status==='resuelto'?'Resuelto':r.status==='alerta'?'Alerta':'Pendiente'
    const cells=[r.iso,r.veh,r.dir||'',r.obs,r.detalle,sev,e]
    rich+=`<tr>${cells.map(v=>`<td style="${tS}">${escH(String(v))}</td>`).join('')}</tr>`
    plain+=cells.join('\t')+'\n'
  })
  rich+='</table>'
  try { navigator.clipboard.write([new ClipboardItem({'text/html':new Blob([rich],{type:'text/html'}),'text/plain':new Blob([plain],{type:'text/plain'})})]); showToast(`✓ Copiadas ${rows.length} filas`) }
  catch { navigator.clipboard.writeText(plain); showToast('✓ Copiado (texto)') }
}
```

- [ ] **Step 10: Mover exportación de Plan completo + Sesión a Resumen**

Al final del pane `#tab-resumen`, tras `#summary-wrap`, añade un sub-bloque:

```html
<div style="display:flex;gap:10px;align-items:center;padding:10px 12px;border-top:1px solid var(--borderSoft);background:var(--bgCardAlt)">
  <span class="section-label">Datos / Sesión</span>
  <button class="btn btn-secondary btn-sm" onclick="exportFullPlan()">↓ Plan completo (XLSX)</button>
  <button class="btn btn-secondary btn-sm" onclick="exportSession()">↓ Sesión (JSON)</button>
</div>
```

Elimina `updateExportTab` (~1432-1437) y su llamada en `updateAllUI`/`switchTab` (ya no hay pestaña export). `exportFullPlan` y `exportSession` se conservan tal cual.

- [ ] **Step 11: Verificar en navegador**

Con datos cargados, en Summary: (a) los pills de estado y severidad filtran; (b) el select de vehículo filtra; (c) "Limpiar filtros" resetea; (d) clic en ▼ de una columna abre popover con valores, aplicar filtra, quitar lo limpia; (e) seleccionar filas muestra la barra negra con el conteo; (f) "Exportar" genera XLSX solo con seleccionadas (o filtradas si no hay selección) e incluye columna Severidad; (g) "Copiar" respeta lo mismo; (h) Plan completo y Sesión descargan. Sin errores en consola.

- [ ] **Step 12: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): B&W summary with severity/vehicle/column filters, row selection and selective export"
```

---

## Task 9: Renombrar pestaña geo + pistas contextuales

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (tab label ~240; hints en cada pane)

- [ ] **Step 1: Renombrar "Wrong Commune" a "Auditoría Geo"**

En el botón `data-tab="tab-geo"` (~240) cambia el texto visible a `📍 Auditoría Geo`.

- [ ] **Step 2: Añadir una pista contextual por pestaña**

En `#tab-plan` (arriba), `#tab-geo` (sidebar header), `#tab-proyectos` y `#tab-resumen` añade una línea breve, p. ej. al inicio del sidebar de geo:

```html
<div style="padding:6px 12px;font-size:10px;color:var(--textFaint);border-bottom:1px solid var(--borderSoft)">Haz clic en un marcador o ítem para enfocarlo. 🔴 lejos · 🟡 cerca del borde. Usa ← → para navegar.</div>
```

(Equivalentes de una línea en las otras pestañas; texto en `references` del spec, sección 8.)

- [ ] **Step 3: Verificar en navegador**

La pestaña dice "Auditoría Geo". Cada pestaña muestra su pista. Sin errores.

- [ ] **Step 4: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): rename geo tab and add contextual hints"
```

---

## Task 10: Navegación al auditar (anterior/siguiente, atajos, progreso)

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (sidebar header geo ~314-323; JS nav + keydown)

- [ ] **Step 1: Añadir controles e indicador de progreso al sidebar geo**

En `.map-sidebar-header` de `#tab-geo` (~314), tras el contador, añade:

```html
<button class="btn btn-secondary btn-sm" onclick="navConflict(-1)" title="Anterior (←)">‹</button>
<button class="btn btn-secondary btn-sm" onclick="navConflict(1)" title="Siguiente (→)">›</button>
<button class="btn btn-secondary btn-sm" onclick="navNextPendiente()" title="Siguiente pendiente">⏭ Pend.</button>
<span id="geo-progress" class="stat-chip-lbl" style="margin-left:6px"></span>
```

- [ ] **Step 2: Implementar navegación y progreso**

Añade junto a `focusConflict` (~1269):

```js
function navConflict(dir){
  const list = getVisibleConflicts()
  if (!list.length) return
  let idx = list.findIndex(c=>c.iso===geoFocusISO)
  idx = idx<0 ? 0 : (idx+dir+list.length)%list.length
  focusConflict(list[idx].iso)
}
function navNextPendiente(){
  const list = getVisibleConflicts()
  const start = Math.max(0, list.findIndex(c=>c.iso===geoFocusISO))
  for (let n=1;n<=list.length;n++){
    const c = list[(start+n)%list.length]
    if (getConflictStatus(c.iso)==='pendiente'){ focusConflict(c.iso); return }
  }
  showToast('No quedan pendientes visibles','warn')
}
function updateGeoProgress(){
  const el=document.getElementById('geo-progress'); if(!el) return
  const total=conflicts.filter(c=>!isExcludedVeh(c.veh)).length
  const done=conflicts.filter(c=>!isExcludedVeh(c.veh) && getConflictStatus(c.iso)!=='pendiente').length
  el.textContent = total ? `${done} / ${total} revisados` : ''
}
```

Llama `updateGeoProgress()` dentro de `renderConflictList()` (al final, ~1266) y en `updateAllUI`.

- [ ] **Step 3: Añadir atajos de teclado**

En `init()` (~1521), tras el wiring, añade:

```js
document.addEventListener('keydown', e => {
  if (/^(INPUT|TEXTAREA|SELECT)$/.test((e.target.tagName||'')) ) return
  if (!document.getElementById('tab-geo').classList.contains('active')) return
  if (e.key==='ArrowRight'){ e.preventDefault(); navConflict(1) }
  else if (e.key==='ArrowLeft'){ e.preventDefault(); navConflict(-1) }
  else if (e.key==='r'||e.key==='R'){ if(geoFocusISO) toggleResolveConflict(geoFocusISO) }
  else if (e.key==='a'||e.key==='A'){ if(geoFocusISO) toggleFlagConflict(geoFocusISO) }
  else if (e.key==='Escape'){ clearGeoFocus() }
})
```

- [ ] **Step 4: Verificar en navegador**

En Auditoría Geo con conflictos: ‹ › saltan entre conflictos volando el mapa; "⏭ Pend." salta al siguiente pendiente; el chip muestra "N / M revisados" y sube al resolver. Con foco activo: `R` resuelve, `A` marca alerta, `Esc` quita foco. Escribir en el buscador NO dispara atajos.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): conflict navigation, keyboard shortcuts and review progress"
```

---

## Task 11: Guía integrada "¿Cómo auditar?"

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (header ~231-234; modal nuevo; JS + flag)

- [ ] **Step 1: Añadir botón en el header**

En `.header-top` (~231), antes del botón "Limpiar", añade:

```html
<button class="btn btn-secondary btn-sm" onclick="openGuide()">❔ ¿Cómo auditar?</button>
```

- [ ] **Step 2: Añadir el modal**

Antes de `<div class="toast" id="toast">` (~460) añade:

```html
<div class="modal-overlay" id="guide-modal" onclick="if(event.target===this)closeGuide()">
  <div class="modal-content">
    <h2 style="font-size:15px;font-weight:700;margin-bottom:12px">Cómo auditar rutas</h2>
    <ol style="font-size:12px;line-height:1.7;padding-left:18px;display:flex;flex-direction:column;gap:6px">
      <li><b>Carga el plan</b> (XLSX/CSV) en la pestaña Cargar Plan. Revisa los Parámetros (vehículos excluidos y de proyecto).</li>
      <li>Ve a <b>Auditoría Geo</b>: el mapa muestra cada entrega. <b>🔴 = comuna equivocada lejos</b>, <b>🟡 = cerca del borde</b> (probable caso límite).</li>
      <li>Navega con <b>‹ ›</b> o <b>← →</b>. Para cada conflicto decide: <b>✓ Resolver</b> (sin problema) o <b>⚠ Alerta</b> (error real). Atajos: R / A / Esc.</li>
      <li>Revisa <b>Proyectos</b> (vehículos VPR01/VPR02) igual que los geo.</li>
      <li>En <b>Summary</b> filtra (estado, severidad, vehículo o por columna), <b>selecciona las filas</b> que quieras y <b>Exporta</b> solo esas.</li>
    </ol>
    <div style="margin-top:14px;text-align:right"><button class="btn btn-primary btn-sm" onclick="closeGuide()">Entendido</button></div>
  </div>
</div>
```

- [ ] **Step 3: Lógica del modal + auto-mostrar la primera vez**

Añade en el bloque de funciones:

```js
function openGuide(){ document.getElementById('guide-modal').classList.add('active') }
function closeGuide(){ document.getElementById('guide-modal').classList.remove('active'); localStorage.setItem('aud_guide_seen_v3','1') }
```

En `init()` (~1521) al final añade:

```js
if (!localStorage.getItem('aud_guide_seen_v3')) setTimeout(openGuide, 500)
```

- [ ] **Step 4: Verificar en navegador**

Borra `localStorage` (`localStorage.clear()` en consola) y recarga → la guía aparece sola. Ciérrala → no reaparece al recargar. El botón "❔ ¿Cómo auditar?" la reabre. El clic fuera del modal lo cierra.

- [ ] **Step 5: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "feat(auditoria): in-app audit guide modal with first-run auto-show"
```

---

## Task 12: Pulido de dinamismo y revisión final taste-skill

**Files:**
- Modify: `misc-projects/AUDITORIARUTAS.html` (`<style>` y detalles)

- [ ] **Step 1: Revisar transiciones y estados**

Confirma que existen transiciones suaves en: filas de tabla (animación `rowIn` ya añadida), pills de filtro (`.filter-pill` ya tiene `transition`), botones (`.btn`). Ajusta cualquier estado hover/focus que se vea brusco. Verifica contraste del modo B&N (texto `#222` sobre blanco, encabezado `#111`).

- [ ] **Step 2: Revisión de accesibilidad básica**

Verifica que los checkboxes y botones tengan área de clic suficiente y que el foco de teclado sea visible (los `.input:focus` ya tienen anillo).

- [ ] **Step 3: Verificación integral en navegador**

Recorre el flujo completo con un plan real: cargar → parámetros → auditar geo (severidad + navegación) → proyectos → resumen (filtros + selección + export) → guía. Sin errores en consola. La app abre con doble clic sin servidor.

- [ ] **Step 4: Commit**

```bash
git add misc-projects/AUDITORIARUTAS.html
git commit -m "polish(auditoria): dynamism and final taste-skill pass"
```

---

## Self-Review (cobertura del spec)

- §1 Estructura 6→4 pestañas → Task 5 (elimina Route Plan/Export) + Task 9 (renombrado). ✅
- §2 Parámetros editables + persistencia v3 → Task 1 + Task 2. ✅
- §3 Proyectos por vehículo → Task 3. ✅
- §4 Severidad 🟡/🔴 + colores mapa → Task 4. ✅
- §5 Bordes más marcados → Task 6. ✅
- §6 Resumen B&N + filtros + selección + export consolidado + plan/sesión → Task 7 (CSS) + Task 8. ✅
- §7 Navegación + atajos + progreso → Task 10. ✅
- §8 Guía integrada + pistas → Task 11 + Task 9 (pistas). ✅
- §9 Estado (params, severity, filtros, selección; eliminar PROY_CONDUCTORS/tabs) → Tasks 1,3,4,5,8. ✅
- §11 Criterios de aceptación → cubiertos por los pasos de verificación de cada tarea. ✅

Sin placeholders; nombres de funciones consistentes entre tareas (`isExcludedVeh`, `isProjectVeh`, `getFilteredSummaryRows`, `rowKey`, `redrawComunaBorders`, `updateGeoProgress`). `redrawComunaBorders` se referencia en Task 2 y se define en Task 6 (nota incluida para crear stub temporal).

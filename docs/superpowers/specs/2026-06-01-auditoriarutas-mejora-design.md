# Mejora de uso — AUDITORIARUTAS.html

**Fecha:** 2026-06-01
**Archivo objetivo:** `misc-projects/AUDITORIARUTAS.html` (un solo archivo HTML, JS vanilla)
**Objetivo:** que cualquier persona pueda auditar rutas **visualmente** (mapa), con filtros que funcionen bien, severidad automática de conflictos, tablas en blanco y negro, exportación selectiva, y guía integrada.

---

## Principios y restricciones

- **Self-contained:** un solo archivo HTML, sin build, sin módulos, sin imports entre archivos. CDNs ya pinneadas (SheetJS `xlsx/0.18.5`, Leaflet `1.9.4`, lz-string `1.5.0`) — no se agregan dependencias nuevas.
- **No se toca el motor geo** (`pointInPolygon`, `comunaFromCoords`, `haversine`, `comunas_data.js`), el parser de archivos, ni la mecánica base de mapas/markers/polylines, salvo lo descrito aquí.
- **Estética taste-skill:** subir tipografía, espaciado y estados; evitar look genérico. Cambios de estética solo en el `<style>` inline.
- **Persistencia:** SOLO se persisten los **parámetros de configuración** en `localStorage` bajo `aud_params_v3` (y el flag de guía `aud_guide_seen_v3`). Los **datos de auditoría** (plan cargado, conflictos, progreso de revisión) **NO se persisten**: arrancan vacíos en cada sesión. `saveState`/`loadState` solo manejan `params`; además `saveState` limpia claves de datos `aud_*_v2` de versiones previas.
- **Idioma:** UI y dominio en español.

---

## 1. Estructura de pestañas (6 → 4)

| Antes | Ahora |
|---|---|
| 📥 Cargar Plan | 📥 **Cargar Plan** + sección plegable ⚙ Parámetros |
| 📋 Route Plan | ❌ **Eliminada** |
| 📍 Wrong Commune | 📍 **Auditoría Geo** (mapa — centro de la auditoría) |
| 🚧 Proyectos | 🚧 **Proyectos** (mapa) |
| 📊 Summary | 📊 **Resumen** (tabla B&N + filtros + selección + exportación) |
| 🚀 Exportar | ❌ **Eliminada** (su contenido se mueve a Resumen) |

Flujo objetivo: **Cargar → revisar mapa → marcar ✓/⚠ → Resumen → exportar lo seleccionado.**

Eliminar del HTML: pestaña/botón `tab-vehiculos` y `tab-export`; del JS: `renderRoutePlan`, `expandAllVehicles`, `collapseAllVehicles`, `toggleExclude` (el botón por-vehículo), `updateExportTab` se reubica en Resumen. Quitar referencias en `switchTab`, `updateAllUI`, `updateBadges`.

---

## 2. Parámetros (panel editable en "Cargar Plan")

Sección plegable **⚙ Parámetros** dentro de `tab-plan`. Estado nuevo persistido:

```js
let params = {
  excludedVehicles: ['VPV01','VPV02','VPV03','VPV04'], // fuera de auditoría geo
  projectVehicles:  ['VPR01','VPR02'],                  // van a Proyectos
  borderKm: 0.1,                                        // umbral amarilla/roja
  borderWeight: 2.2, borderOpacity: 0.75               // grosor/opacidad de bordes
}
```

- Clave `aud_params_v3` (JSON). Cargar en `loadState`, guardar en `saveState`.
- **Vehículos excluidos** y **vehículos de proyecto**: UI de chips editables (input para agregar + chip con ✕ para quitar). Defaults pre-cargados.
- `excludedVehicles` deriva el `Set` que ya usa la lógica geo. Se elimina el `excludedVehicles` poblado por clics; ahora proviene de `params.excludedVehicles`.
- Cambiar parámetros re-ejecuta `runGeoAnalysis()` (o al menos recalcula severidad y re-renderiza) y persiste.

---

## 3. Detección de proyectos por vehículo

`getProyectosData()` deja de usar la lista de conductores y filtra por vehículo:

```js
function getProyectosData() {
  const set = new Set(params.projectVehicles.map(v => v.toUpperCase()))
  return routeData.filter(r => set.has((r.veh||'').toUpperCase()))
}
```

`runGeoAnalysis` debe excluir de los conflictos geo tanto `excludedVehicles` como `projectVehicles` (los vehículos de proyecto no generan conflictos de comuna).

---

## 4. Severidad automática 🟡/🔴

Nueva dimensión **automática** (independiente del estado manual del auditor).

- Calcular **distancia mínima del punto al borde (polígono) de la comuna declarada** (`kDir`).
  - Nueva función `distPointToPolygonKm(lat, lng, polygon)`: mínima distancia punto→segmento sobre todos los segmentos del anillo, usando `haversine` para convertir a km (proyección local suficiente a escala de comuna).
- Clasificación en cada conflicto:
  - `severity = (distBordeKm <= params.borderKm) ? 'amarilla' : 'roja'`
- Guardar `severity` y `distBordeKm` en cada objeto de `conflicts`.
- **Default umbral:** `0.1 km` (100 m). Editable en Parámetros.
- Estado manual (`pendiente`/`alerta`/`resuelto`) se mantiene **separado** de la severidad.

### Colores en el mapa (markers)
- Pendiente + 🔴 roja → rojo (con pulso).
- Pendiente + 🟡 amarilla → amarillo/ámbar.
- Resuelto → verde con ✓ (sin pulso).
- Foco → resaltado actual.
- (Las tablas son B&N; los mapas conservan color porque la severidad es su propósito.)

---

## 5. Bordes de comuna más marcados

En `drawComunaPolygons`, usar `params.borderWeight` (def. 2.2) y `params.borderOpacity` (def. 0.75), color de línea más definido (p. ej. `#475569`/`#334155`), manteniendo `fillOpacity` baja para no tapar el mapa. Redibujar al cambiar parámetros.

---

## 6. Resumen — tabla B&N, filtros, selección y exportación

### 6.1 Diseño en blanco y negro (taste-skill)
- `.dash-table` en escala de grises: sin azul/rojo/verde en celdas. Encabezado fijo, borde fino, hover sutil, buena tipografía/espaciado.
- Estado y severidad por **símbolo + peso**, no por color: `⏳ Pendiente`, `✓ Resuelto`, `⚠ Alerta`; severidad `🔴/🟡` se muestra como texto/símbolo monocromo (p. ej. "● Lejos" / "○ Borde") o glifo, manteniendo legibilidad sin color.

### 6.2 Filtros que funcionan bien ("elegir qué filtrar")
- **Pills de estado:** Todos / Pendiente / Alerta / Resuelto (ya existen).
- **Pills de severidad:** Todas / 🔴 Rojas / 🟡 Amarillas (nuevo).
- **Filtro por vehículo** (nuevo).
- **Filtrado tipo Excel por columna** (con diseño cuidado, sin pinta de Excel): clic en encabezado → popover con buscador + checkboxes de los valores distintos de esa columna. Combinable entre columnas (AND), y con el buscador global y los pills. Ícono de embudo en columnas con filtro activo + botón **Limpiar filtros**.
- Todos los filtros se aplican en una sola función de "filas visibles" usada por render, exportación y copiar.

### 6.3 Selección de filas para exportar
- Columna de **checkbox** + checkbox "seleccionar todo" (respeta el filtro vigente).
- Contador "**N seleccionadas**".
- **Exportar / copiar respeta la selección.** Si no hay nada seleccionado, exporta/copia lo visible (filtrado).

### 6.4 Contenido consolidado
- Una sola tabla **geo + proyectos** juntos. Columnas: `ISO · Vehículo · Dirección · Tipo · Detalle (comuna dir → real) · Severidad · Estado`.
  - Proyectos aparecen con Tipo "Proyecto" (sin severidad geo).
- Botones en la barra de Resumen:
  - **↓ Exportar seleccionadas** (XLSX, una hoja, formato cuidado: encabezado, anchos, fila fija).
  - **📋 Copiar** (selección/visible).
  - **↓ Plan completo (XLSX)** y **↓ Sesión (JSON)** — reubicados aquí desde la antigua pestaña Exportar (pueden ir en un sub-bloque "Datos / Sesión").

### 6.5 Dinamismo
- Transiciones suaves al filtrar/ordenar, contadores en vivo, micro-animación de filas al cambiar de estado.

---

## 7. Navegación al auditar (pestaña mapa)

- **Anterior / Siguiente** entre conflictos: botones en el panel del mapa + teclas `←`/`→`. Cada salto hace `flyToISO` y abre el popup.
- Opción **"Siguiente pendiente"** (salta los ya revisados).
- Atajos de teclado: `R` resolver foco · `A` marcar alerta foco · `Esc` quitar foco. (Solo activos en la pestaña de mapa, sin romper escritura en inputs.)
- **Indicador de progreso** "12 / 28 revisados" visible en el header del panel del mapa.

---

## 8. Guía integrada ("cómo auditar")

- Botón **❔ ¿Cómo auditar?** en el header → `modal-overlay` con:
  - Pasos numerados: 1) Cargar plan, 2) Revisar Auditoría Geo en el mapa, 3) Interpretar 🔴 lejos / 🟡 borde, 4) Marcar ✓ resuelto / ⚠ alerta (botones o atajos), 5) Revisar Resumen y filtrar, 6) Seleccionar filas y exportar.
  - Leyenda de marcadores y atajos de teclado.
- **Pista contextual** (una línea) por pestaña.
- Auto-mostrar la primera vez (flag `aud_guide_seen_v3` en `localStorage`), luego bajo demanda.

---

## 9. Estado (resumen de cambios en variables)

- **Nuevo:** `params` (objeto, persistido en `aud_params_v3`).
- **Modificado:** `excludedVehicles` pasa a derivarse de `params.excludedVehicles`.
- **Modificado:** cada `conflict` gana `severity` y `distBordeKm`.
- **Nuevo:** estado de filtros de columna del Resumen (p. ej. `summaryColFilters = {}`), filtro de severidad (`severityFilter`), filtro de vehículo, y selección (`selectedRows = new Set()`).
- **Eliminado:** lógica de `tab-vehiculos` y `tab-export` y la lista `PROY_CONDUCTORS`.

---

## 10. Fuera de alcance (no tocar)

Motor geo y `comunas_data.js`, parser de archivos, estructura de archivo único, CDNs, mecánica base de Leaflet (más allá de estilo de bordes y colores de markers por severidad).

---

## 11. Criterios de aceptación

1. La pestaña Route Plan ya no existe; auditar es principalmente visual en el mapa.
2. Parámetros editables con defaults `VPV01-04` (excluidos) y `VPR01/VPR02` (proyecto); cambios persisten y recalculan.
3. Proyectos se detectan por vehículo `VPR01/VPR02`; esos vehículos no generan conflictos geo.
4. Conflictos geo se clasifican 🟡 (≤ 0.1 km del borde) / 🔴 (más lejos); colores correctos en el mapa.
5. Bordes de comuna claramente más visibles.
6. Resumen en blanco y negro; filtros (estado, severidad, vehículo, por columna tipo Excel) funcionan y se combinan; limpiar filtros funciona.
7. Selección de filas; exportar/copiar respeta la selección (o lo visible si no hay selección).
8. Plan completo y sesión JSON exportables desde Resumen.
9. Navegación anterior/siguiente + atajos + progreso en el mapa.
10. Guía integrada accesible y auto-mostrada la primera vez.
11. Todo en un solo archivo HTML, sin dependencias nuevas, abre con doble clic.

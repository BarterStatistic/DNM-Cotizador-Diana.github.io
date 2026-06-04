# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripción

Cotizador web estático para financiamiento de motocicletas, desarrollado para **Prospect Team / Agencia Dinamo Saltillo**. Sin backend, sin build, sin dependencias instalables — todo está en `index.html`.

## Desarrollo y pruebas

Abrir `index.html` directamente en el navegador. No hay servidor, build ni paso de compilación.

Para desplegar en Netlify CLI:
```bash
netlify deploy --prod
```

## Arquitectura

Todo el código vive en `index.html` en tres secciones: CSS (`:root` variables + reglas), HTML (estructura de la SPA) y `<script>` (lógica JS vanilla).

### Datos (`<script>`, ~línea 845)

- **`MODELS`** — Array de tuplas `[nombre, precioLista, costoServicioPreventivo]`. Editar aquí para agregar o actualizar modelos.
- **`SCHEMES`** — Objeto con los esquemas de crédito disponibles. Cada esquema define:
  - `min` / `max` — rango de enganche permitido (%)
  - `termUnit` — `"quincenas"` o `"semanas"`
  - `terms` — plazos disponibles
  - `levels` — array de rangos de enganche con sus multiplicadores (`m`) por plazo. La mensualidad/quincena/semana se calcula como `montoPorFinanciar × multiplicador`.

Esquemas actuales: `motonomina`, `credinamo`, `credinamo_flex`, `motoxpress`, `motoxpress_flex`, `enganche50`.

### Flujo de 3 pasos

| Paso | Panel | Función clave |
|------|-------|---------------|
| 0 — Modelo | `#panel0` | `selectModel(idx, card)`, `filterModels(q)` |
| 1 — Esquema | `#panel1` | `selectScheme(card)` |
| 2 — Enganche | `#panel2` | `validateDown()`, `generate()` |

`goStep(n)` controla qué panel es visible y el estado de los indicadores de paso.

### Cálculo (`generate()`, ~línea 1095)

1. Precio efectivo = precio lista + servicio preventivo (si `svcActive`).
2. Monto a financiar = precio efectivo − enganche.
3. Para cada plazo del esquema: buscar en `levels` el rango que contiene el `dp%` → multiplicar monto a financiar por el coeficiente.
4. El resultado se renderiza como tabla en el panel de resultados.

### Descarga (`doDownload()`, ~línea 1267)

Usa `html2canvas` (cargado via CDN) para capturar el panel de resultados como imagen JPG descargable. Soporta dos modos: un solo plazo o varios plazos combinados.

## Variables CSS relevantes

Definidas en `:root`:
- `--gold` / `--gold2` / `--gold3` — familia dorada (#C99A3B, #EDB95A, #F5D080)
- `--cyan` / `--cyan2` — azul (#3AADE0, #69C5EE)
- `--bg` / `--bg2` / `--bg3` / `--surf` — escala de fondos oscuros
- `--ff-head` (`Urbanist`), `--ff-body` (`Manrope`), `--ff-mono` (`DM Mono`)

## Despliegue

`netlify.toml` ya configura headers de seguridad, caché (HTML sin caché, favicon 1 año) y redirección 404. El directorio de publicación es la raíz (`.`).

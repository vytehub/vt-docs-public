---
sidebar_position: 2
title: Colors
---

# Primitive & Semantic Colors

> **Objetivo:** que el equipo diseñe y construya UI usando **intenciones** (roles) y no “colores sueltos”.
> Esto hace que el sistema sea **consistente**, facilite el **theming Light/Dark**, y permita cambiar el **primary** sin tocar componentes.

---

## 1) Cómo pensar el sistema

### 1.1 Primitives vs Semantics (la regla de oro)

- **Primitives** ([`primitive-colors.tokens.json`](primitive-colors.tokens.json)) = “materia prima”: escalas de color (gray/indigo/rose/…).
  - Ej: `indigo.600`, `gray.100`, `common.white`.
  - **No** se usan directo en componentes ni en pantallas (salvo casos muy puntuales en exploración).
- **Semantic tokens** ([`semantic-colors.light.json`](semantic-colors.light.json) y 
[`semantic-colors.dark.json`](semantic-colors.dark.json)) = “intenciones”: tokens con significado de UI.
  - Ej: `surface.canvas`, `border.focus`, `action.primary.bg.hover`, `feedback.error.bg.tertiary`.
  - Son los que usás para **diseñar componentes** y **pantallas**.

✅ Si mañana cambiás el color “primary” (rebrand), **no** cambiás botones ni pantallas: cambiás *1 vez* la asignación en semánticos (porque los componentes consumen semánticos).

---

## 3) En Figma: modes (Light/Dark)

En Figma, **Semantic Colors** vive en una sola colección con dos **modes**: `Light` y `Dark`.
Cada variable semántica es un **alias** que apunta a una variable en **Primitive Colors**.

---

## 3) Archivos y para qué sirve cada uno

- **[`primitive-colors.tokens.json`](primitive-colors.tokens.json)**
  - Paleta + escalas base (source-of-truth de valores de color).
  - Se importa primero (porque los semánticos lo referencian).

- **[`semantic-colors.light.json`](./semantic-colors.light.json)**
  - Semantic tokens para **modo light**.
  - Todos los `value` son referencias del estilo `{gray.50}` o `{indigo.600}`.

- **[`semantic-semantic.dark.json`](semantic-colors.dark.json)**
  - Semantic tokens para **modo dark**.
  - **Mismos nombres** que light (para que el código no cambie), pero con referencias distintas para mantener contraste.

---

## 3) Qué NO hacer (para no repetir el lío)

- ❌ No crear tokens tipo `button.primary.bg` dentro de la colección semántica principal.
  - Eso te ata la colección a componentes y crece sin control.
- ✅ En cambio, el Button (y cualquier control) debe construirse con:
  - `action.primary.*` + `content.*` + `border.*` + `ring.*`
- ✅ Solo creamos tokens “component-specific” **si**:
  - un componente tiene necesidades muy particulares,
  - y no encaja en los roles existentes,
  - y ya probaste que con roles se vuelve confuso.

Para un equipo chico, el 80/20 es: **roles claros + componentes consumiendo roles**.

---

## 4) Qué significa cada familia (y cómo se usa)

> **Idea:** cuando elegís un token, elegís un **rol**. El rol define el *“para qué”* del color.

### 4.1 `surface.*` — fondos y capas neutrales

- Son **superficies** (backgrounds) sobre las que vive el contenido.
- No expresan estado (para estados, usar `feedback.*` o `action.*`).
- Se usan en layouts: páginas, secciones, cards, overlays.

**Scope recomendado en Figma:** Fill (Frame/Shape)

|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`surface.canvas`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|Fondo global de la app / páginas.|
|`surface.elevated`|`#ffffff`|`#1f2937`|Fill (Frame/Shape)|Cards / paneles con elevación.|
|`surface.invert`|`#111827`|`#f9fafb`|Fill (Frame/Shape)|Secciones oscuras en modo light / contraste invertido.|
|`surface.overlay`|`#e5e7eb`|`#374151`|Fill (Frame/Shape)|Scrim/overlay detrás de modales o hover overlay.|
|`surface.subtle`|`#f3f4f6`|`#111827`|Fill (Frame/Shape)|Secciones suaves dentro de canvas.|
|`surface.sunken`|`#e5e7eb`|`#030712`|Fill (Frame/Shape)|Inputs/wells embutidos dentro de otra superficie.|


---

### 4.2 `content.*` — texto e íconos

Piensa “contenido” como **lo que está encima de una superficie**.

#### `content.text.*`

- `primary.*`: máxima jerarquía (títulos, body principal).
- `secondary.*`: jerarquía media (metadata, labels secundarios).
- `muted`: baja jerarquía (ayudas, placeholder, hint).
- `inverse`: para usar sobre `surface.invert` u otros fondos oscuros.
- `on.primary` / `on.secondary`: para texto encima de backgrounds de acción (botones filled).
- `link.*`: links.
- `accent.*`: texto destacado (no CTA, sino “highlight”).
- `feedback.*`: texto en estados.

**Scope recomendado en Figma:** Text

|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`content.text.accent.default`|`#e11d48`|`#fda4af`|Text|Texto destacado (accent) para resaltar valores puntuales.|
|`content.text.accent.disabled`|`#9ca3af`|`#6b7280`|Text|Accent disabled.|
|`content.text.accent.disabledLight`|`#d1d5db`|`#4b5563`|Text|Accent disabled sobre fondos claros.|
|`content.text.accent.focus`|`#9f1239`|`#fecdd3`|Text|Accent focus.|
|`content.text.accent.hover`|`#be123c`|`#fecdd3`|Text|Accent hover.|
|`content.text.feedback.error`|`#991b1b`|`#fecaca`|Text|Texto para mensajes de estado (error).|
|`content.text.feedback.info`|`#075985`|`#bae6fd`|Text|Texto para mensajes de estado (info).|
|`content.text.feedback.success`|`#166534`|`#bbf7d0`|Text|Texto para mensajes de estado (success).|
|`content.text.feedback.warning`|`#78350f`|`#fde68a`|Text|Texto para mensajes de estado (warning).|
|`content.text.link.default`|`#4f46e5`|`#a5b4fc`|Text|Link normal.|
|`content.text.link.disabled`|`#9ca3af`|`#6b7280`|Text|Link disabled.|
|`content.text.link.disabledLight`|`#d1d5db`|`#4b5563`|Text|Link disabled sobre fondos claros.|
|`content.text.link.focus`|`#3730a3`|`#c7d2fe`|Text|Link focus (teclado).|
|`content.text.link.hover`|`#4338ca`|`#c7d2fe`|Text|Link hover.|
|`content.text.on.accent`|`#ffffff`|`#111827`|Text|**Deprecated.** Alias de `on.secondary`. Mantener solo durante migración.|
|`content.text.on.primary`|`#ffffff`|`#111827`|Text|Texto sobre background de acción primaria (botón filled).|
|`content.text.on.secondary`|`#ffffff`|`#111827`|Text|Texto sobre background de acción secondary (botón filled rose).|
|`content.text.primary.default`|`#111827`|`#f9fafb`|Text|Texto principal (body/headers).|
|`content.text.primary.disabled`|`#9ca3af`|`#6b7280`|Text|Texto principal en disabled.|
|`content.text.primary.inverse`|`#ffffff`|`#111827`|Text|Texto sobre superficies invertidas.|
|`content.text.primary.muted`|`#6b7280`|`#9ca3af`|Text|Texto de bajo énfasis (cuando use primary pero más suave).|
|`content.text.primary.secondary`|`#374151`|`#e5e7eb`|Text|Texto primario.|


#### `content.icon.*`

- `default/muted/disabled/inverse`: set base de iconografía.
- `on.primary` / `on.accent`: iconos sobre backgrounds de acción.
- `action.*`: iconos dentro de componentes de acción (cuando el icono es parte del control).
- `feedback.*`: iconos para estados.
- `brand`: iconos con color de marca sin ser CTA.
- `magic`: “momento especial” (AI/featured). Usar muy poco.

**Scope recomendado en Figma:** Fill/Stroke (Icon)

|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`content.icon.accent.active`|`#be123c`|`#fb7185`|Fill/Stroke (Icon)|—|
|`content.icon.accent.default`|`#f43f5e`|`#fda4af`|Fill/Stroke (Icon)|—|
|`content.icon.accent.hover`|`#e11d48`|`#fecdd3`|Fill/Stroke (Icon)|—|
|`content.icon.active`|`#111827`|`#f3f4f6`|Fill/Stroke (Icon)|—|
|`content.icon.brand.active`|`#4338ca`|`#818cf8`|Fill/Stroke (Icon)|—|
|`content.icon.brand.default`|`#6366f1`|`#a5b4fc`|Fill/Stroke (Icon)|—|
|`content.icon.brand.hover`|`#4f46e5`|`#c7d2fe`|Fill/Stroke (Icon)|—|
|`content.icon.default`|`#4b5563`|`#d1d5db`|Fill/Stroke (Icon)|Icono neutro por defecto.|
|`content.icon.disabled`|`#d1d5db`|`#4b5563`|Fill/Stroke (Icon)|Icono deshabilitado.|
|`content.icon.feedback.error`|`#dc2626`|`#fca5a5`|Fill/Stroke (Icon)|Iconos para estados (error).|
|`content.icon.feedback.info`|`#0284c7`|`#7dd3fc`|Fill/Stroke (Icon)|Iconos para estados (info).|
|`content.icon.feedback.success`|`#16a34a`|`#86efac`|Fill/Stroke (Icon)|Iconos para estados (success).|
|`content.icon.feedback.warning`|`#f59e0b`|`#fcd34d`|Fill/Stroke (Icon)|Iconos para estados (warning).|
|`content.icon.hover`|`#1f2937`|`#e5e7eb`|Fill/Stroke (Icon)|—|
|`content.icon.inverse`|`#f9fafb`|`#111827`|Fill/Stroke (Icon)|Icono sobre superficie invertida.|
|`content.icon.inverseSecondary`|`#d1d5db`|`#374151`|Fill/Stroke (Icon)|—|
|`content.icon.magic`|`#8b5cf6`|`#c4b5fd`|Fill/Stroke (Icon)|Icono “especial/AI” (usar muy poco).|


---

### 4.3 `border.*` — strokes, divisores y contornos

- `default/hover/focus/disabled`: bordes típicos de interacción.
- `muted`: divisores suaves (hairlines).
- `strong`: borde de énfasis (selección, separadores fuertes).
- `brand`: borde con marca sin que sea un control de acción.
- `feedback.*`: bordes de validación/estado.

**Scope recomendado en Figma:** Stroke

|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`border.brand.active`|`#3730a3`|`#818cf8`|Stroke|—|
|`border.brand.default`|`#4f46e5`|`#a5b4fc`|Stroke|—|
|`border.brand.disabled`|`#d1d5db`|`#374151`|Stroke|—|
|`border.brand.hover`|`#4338ca`|`#c7d2fe`|Stroke|—|
|`border.default`|`#d1d5db`|`#374151`|Stroke|Borde por defecto en componentes interactivos.|
|`border.disabled`|`#e5e7eb`|`#1f2937`|Stroke|Borde de componentes disabled.|
|`border.feedback.error`|`#ef4444`|`#f87171`|Stroke|Stroke/borde para estados (error).|
|`border.feedback.info`|`#0ea5e9`|`#38bdf8`|Stroke|Stroke/borde para estados (info).|
|`border.feedback.success`|`#22c55e`|`#4ade80`|Stroke|Stroke/borde para estados (success).|
|`border.feedback.warning`|`#f59e0b`|`#fbbf24`|Stroke|Stroke/borde para estados (warning).|
|`border.focus`|`#4338ca`|`#a5b4fc`|Stroke|Borde/focus visible (accesibilidad).|
|`border.strong`|`#4b5563`|`#9ca3af`|Stroke|Borde de alto contraste (selección/énfasis).|
|`border.subtle`|`#e5e7eb`|`#1f2937`|Stroke|—|


---

### 4.4 `ring.*` — focus ring (accesibilidad)

**Qué es “ring”:** es el *anillo de foco* visible cuando navegás con teclado (Tab).
En código suele ser un `box-shadow` con **dos capas**:

- `inner`: borde interno (nítido).
- `outer`: halo externo (suave) para que se vea sobre cualquier fondo.

También existen rings para:
- `action.*`: foco en controles interactivos.
- `feedback.*`: foco/estado cuando un campo tiene error/success y además está focused.

**Scope recomendado en Figma:** Effect

|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`ring.feedback.error`|`#ef4444`|`#f87171`|Effect (Focus ring)|Ring para estados (validación) cuando se requiere.|
|`ring.feedback.info`|`#0ea5e9`|`#38bdf8`|Effect (Focus ring)|Ring para estados (validación) cuando se requiere.|
|`ring.feedback.success`|`#22c55e`|`#4ade80`|Effect (Focus ring)|Ring para estados (validación) cuando se requiere.|
|`ring.feedback.warning`|`#f59e0b`|`#fbbf24`|Effect (Focus ring)|Ring para estados (validación) cuando se requiere.|
|`ring.focus`|`#6366f1`|`#a5b4fc`|Effect (Focus ring)|—|


---

### 4.5 `action.*` — Primary, Secondary, Neutral, and Danger

Esta familia es la que hace que “mi botón cambie solo si cambio el primary”.

#### `action.primary.*`
- **Primary** = tu CTA principal (lo más importante).
- Debe funcionar bien para:
  - filled buttons,
  - outlined buttons,
  - icon buttons,
  - estados: default/hover/active/focus/disabled.

#### `action.secondary.*`
- **secondary**: alternative/second-level CTA (rose). Use when a second important action exists alongside primary.
> Rule: primary flow → `action.primary.*`; secondary important action → `action.secondary.*`

#### `action.neutral.*` (soft)
- **neutral**: safe soft actions (Cancel, Back, Close). No strong color signal. Gray-based.

#### `action.danger.*`
- **danger**: destructive actions (Delete, Remove, Revoke). Red-based for clear warning signal.

> ⚠️ **Deprecated alias:** `action.accent` is a temporary CSS-variable alias pointing to `action.secondary`. It will be removed in a future release. Migrate to `action.secondary.*`.

**Scopes recomendados en Figma:**
- `*.bg.*` → Fill
- `*.border.*` → Stroke
- `*.text.*` → Text
- `*.icon.*` → Fill/Stroke

#### Tabla: `action.primary.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`action.primary.filled.bg.default`|`#4f46e5`|`#818cf8`|Fill (Frame/Shape)|—|
|`action.primary.filled.bg.disabled`|`#d1d5db`|`#374151`|Fill (Frame/Shape)|—|
|`action.primary.filled.bg.disabledLight`|`#e5e7eb`|`#1f2937`|Fill (Frame/Shape)|—|
|`action.primary.filled.bg.hover`|`#4338ca`|`#a5b4fc`|Fill (Frame/Shape)|—|
|`action.primary.filled.bg.pressed`|`#3730a3`|`#6366f1`|Fill (Frame/Shape)|—|
|`action.primary.filled.text`|`#ffffff`|`#111827`|All|—|
|`action.primary.outlined.bg.default`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|—|
|`action.primary.outlined.bg.disabled`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|—|
|`action.primary.outlined.bg.focus`|`#e0e7ff`|`#312e81`|Fill (Frame/Shape)|—|
|`action.primary.outlined.bg.hover`|`#eef2ff`|`#1e1b4b`|Fill (Frame/Shape)|—|
|`action.primary.outlined.bg.pressed`|`#c7d2fe`|`#1e1b4b`|Fill (Frame/Shape)|—|
|`action.primary.outlined.border.default`|`#4f46e5`|`#a5b4fc`|Stroke|—|
|`action.primary.outlined.border.disabled`|`#d1d5db`|`#374151`|Stroke|—|
|`action.primary.outlined.border.focus`|`#3730a3`|`#c7d2fe`|Stroke|—|
|`action.primary.outlined.border.hover`|`#4338ca`|`#c7d2fe`|Stroke|—|
|`action.primary.outlined.text.default`|`#4f46e5`|`#a5b4fc`|Text|—|
|`action.primary.outlined.text.disabled`|`#9ca3af`|`#6b7280`|Text|—|
|`action.primary.outlined.text.focus`|`#3730a3`|`#c7d2fe`|Text|—|
|`action.primary.outlined.text.hover`|`#4338ca`|`#c7d2fe`|Text|—|


#### Tabla: `action.secondary.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`action.secondary.filled.bg.default`|`#e11d48`|`#fb7185`|Fill (Frame/Shape)|—|
|`action.secondary.filled.bg.disabled`|`#d1d5db`|`#374151`|Fill (Frame/Shape)|—|
|`action.secondary.filled.bg.disabledLight`|`#e5e7eb`|`#1f2937`|Fill (Frame/Shape)|—|
|`action.secondary.filled.bg.hover`|`#be123c`|`#fda4af`|Fill (Frame/Shape)|—|
|`action.secondary.filled.bg.pressed`|`#9f1239`|`#f43f5e`|Fill (Frame/Shape)|—|
|`action.secondary.filled.text`|`#ffffff`|`#111827`|All|—|
|`action.secondary.outlined.bg.default`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|—|
|`action.secondary.outlined.bg.disabled`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|—|
|`action.secondary.outlined.bg.focus`|`#ffe4e6`|`#881337`|Fill (Frame/Shape)|—|
|`action.secondary.outlined.bg.hover`|`#fff1f2`|`#4c0519`|Fill (Frame/Shape)|—|
|`action.secondary.outlined.bg.pressed`|`#fecdd3`|`#4c0519`|Fill (Frame/Shape)|—|
|`action.secondary.outlined.border.default`|`#e11d48`|`#fda4af`|Stroke|—|
|`action.secondary.outlined.border.disabled`|`#d1d5db`|`#374151`|Stroke|—|
|`action.secondary.outlined.border.focus`|`#9f1239`|`#fecdd3`|Stroke|—|
|`action.secondary.outlined.border.hover`|`#be123c`|`#fecdd3`|Stroke|—|
|`action.secondary.outlined.text.default`|`#e11d48`|`#fda4af`|Text|—|
|`action.secondary.outlined.text.disabled`|`#9ca3af`|`#6b7280`|Text|—|
|`action.secondary.outlined.text.focus`|`#9f1239`|`#fecdd3`|Text|—|
|`action.secondary.outlined.text.hover`|`#be123c`|`#fecdd3`|Text|—|


#### Tabla: `action.neutral.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`action.neutral.filled.bg.default`|`#f3f4f6`|`#374151`|Fill (Frame/Shape)|Fondo filled neutral default.|
|`action.neutral.filled.bg.disabled`|`#e5e7eb`|`#4b5563`|Fill (Frame/Shape)|Filled neutral disabled.|
|`action.neutral.filled.bg.disabledLight`|`#f3f4f6`|`#374151`|Fill (Frame/Shape)|Filled neutral disabled (light variant).|
|`action.neutral.filled.bg.hover`|`#e5e7eb`|`#4b5563`|Fill (Frame/Shape)|Filled neutral hover.|
|`action.neutral.filled.bg.pressed`|`#d1d5db`|`#1f2937`|Fill (Frame/Shape)|Filled neutral pressed.|
|`action.neutral.filled.text`|`#111827`|`#f9fafb`|All|Texto sobre filled neutral.|
|`action.neutral.outlined.bg.default`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|Outlined neutral default bg.|
|`action.neutral.outlined.bg.disabled`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|Outlined neutral disabled bg.|
|`action.neutral.outlined.bg.focus`|`#e5e7eb`|`#1f2937`|Fill (Frame/Shape)|Outlined neutral focus bg.|
|`action.neutral.outlined.bg.hover`|`#f3f4f6`|`#111827`|Fill (Frame/Shape)|Outlined neutral hover bg.|
|`action.neutral.outlined.bg.pressed`|`#e5e7eb`|`#111827`|Fill (Frame/Shape)|Outlined neutral pressed bg.|
|`action.neutral.outlined.border.default`|`#d1d5db`|`#374151`|Stroke|Outlined neutral border default.|
|`action.neutral.outlined.border.disabled`|`#e5e7eb`|`#1f2937`|Stroke|Outlined neutral border disabled.|
|`action.neutral.outlined.border.focus`|`#6b7280`|`#6b7280`|Stroke|Outlined neutral border focus.|
|`action.neutral.outlined.border.hover`|`#9ca3af`|`#4b5563`|Stroke|Outlined neutral border hover.|
|`action.neutral.outlined.text.default`|`#374151`|`#e5e7eb`|Text|Outlined neutral text default.|
|`action.neutral.outlined.text.disabled`|`#9ca3af`|`#6b7280`|Text|Outlined neutral text disabled.|
|`action.neutral.outlined.text.focus`|`#111827`|`#f9fafb`|Text|Outlined neutral text focus.|
|`action.neutral.outlined.text.hover`|`#1f2937`|`#f3f4f6`|Text|Outlined neutral text hover.|


#### Tabla: `action.danger.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`action.danger.filled.bg.default`|`#dc2626`|`#f87171`|Fill (Frame/Shape)|Fondo filled danger default.|
|`action.danger.filled.bg.disabled`|`#d1d5db`|`#374151`|Fill (Frame/Shape)|Filled danger disabled.|
|`action.danger.filled.bg.disabledLight`|`#e5e7eb`|`#1f2937`|Fill (Frame/Shape)|Filled danger disabled (light variant).|
|`action.danger.filled.bg.hover`|`#b91c1c`|`#fca5a5`|Fill (Frame/Shape)|Filled danger hover.|
|`action.danger.filled.bg.pressed`|`#991b1b`|`#ef4444`|Fill (Frame/Shape)|Filled danger pressed.|
|`action.danger.filled.text`|`#ffffff`|`#111827`|All|Texto sobre filled danger.|
|`action.danger.outlined.bg.default`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|Outlined danger default bg.|
|`action.danger.outlined.bg.disabled`|`#f9fafb`|`#030712`|Fill (Frame/Shape)|Outlined danger disabled bg.|
|`action.danger.outlined.bg.focus`|`#fee2e2`|`#7f1d1d`|Fill (Frame/Shape)|Outlined danger focus bg.|
|`action.danger.outlined.bg.hover`|`#fef2f2`|`#450a0a`|Fill (Frame/Shape)|Outlined danger hover bg.|
|`action.danger.outlined.bg.pressed`|`#fecaca`|`#450a0a`|Fill (Frame/Shape)|Outlined danger pressed bg.|
|`action.danger.outlined.border.default`|`#dc2626`|`#fca5a5`|Stroke|Outlined danger border default.|
|`action.danger.outlined.border.disabled`|`#d1d5db`|`#374151`|Stroke|Outlined danger border disabled.|
|`action.danger.outlined.border.focus`|`#991b1b`|`#fecaca`|Stroke|Outlined danger border focus.|
|`action.danger.outlined.border.hover`|`#b91c1c`|`#fecaca`|Stroke|Outlined danger border hover.|
|`action.danger.outlined.text.default`|`#dc2626`|`#fca5a5`|Text|Outlined danger text default.|
|`action.danger.outlined.text.disabled`|`#9ca3af`|`#6b7280`|Text|Outlined danger text disabled.|
|`action.danger.outlined.text.focus`|`#991b1b`|`#fecaca`|Text|Outlined danger text focus.|
|`action.danger.outlined.text.hover`|`#b91c1c`|`#fecaca`|Text|Outlined danger text hover.|


---

### 4.6 `feedback.*` — estados (success / warning / error / info)

Feedback es **solo** para estados (validación, alertas, toasts).

Cada estado tiene:
- `bg.primary`: fondo fuerte (badge sólido, chip, highlight intenso).
- `bg.secondary`: fondo fuerte “oscuro” (más contraste / pressed / encabezado de banner).
- `bg.tertiary`: fondo suave (alert suave / fondo de validación).
- `text`, `icon`, `border`: contenido y contorno del estado.

**Scope recomendado en Figma:** depende del token (Fill/Text/Stroke/Icon).  
En la tabla ya figura por token.

#### `feedback.success.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`feedback.success.bg.primary`|`#16a34a`|`#4ade80`|Fill (Frame/Shape)|Fondo fuerte (badge/etiqueta sólida). para estado success.|
|`feedback.success.bg.secondary`|`#15803d`|`#22c55e`|Fill (Frame/Shape)|Fondo fuerte oscuro (pressed/alto contraste). para estado success.|
|`feedback.success.bg.tertiary`|`#f0fdf4`|`#14532d`|Fill (Frame/Shape)|Fondo suave (alert suave / highlight). para estado success.|
|`feedback.success.border`|`#22c55e`|`#4ade80`|Stroke|Border para estado success.|
|`feedback.success.icon`|`#16a34a`|`#86efac`|Fill/Stroke (Icon)|Icon para estado success.|
|`feedback.success.text`|`#166534`|`#bbf7d0`|Text|Text para estado success.|


#### `feedback.warning.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`feedback.warning.bg.primary`|`#d97706`|`#fbbf24`|Fill (Frame/Shape)|Fondo fuerte (badge/etiqueta sólida). para estado warning.|
|`feedback.warning.bg.secondary`|`#b45309`|`#f59e0b`|Fill (Frame/Shape)|Fondo fuerte oscuro (pressed/alto contraste). para estado warning.|
|`feedback.warning.bg.tertiary`|`#fffbeb`|`#78350f`|Fill (Frame/Shape)|Fondo suave (alert suave / highlight). para estado warning.|
|`feedback.warning.border`|`#f59e0b`|`#fbbf24`|Stroke|Border para estado warning.|
|`feedback.warning.icon`|`#d97706`|`#fcd34d`|Fill/Stroke (Icon)|Icon para estado warning.|
|`feedback.warning.text`|`#78350f`|`#fde68a`|Text|Text para estado warning.|


#### `feedback.error.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`feedback.error.bg.primary`|`#dc2626`|`#f87171`|Fill (Frame/Shape)|Fondo fuerte (badge/etiqueta sólida). para estado error.|
|`feedback.error.bg.secondary`|`#b91c1c`|`#ef4444`|Fill (Frame/Shape)|Fondo fuerte oscuro (pressed/alto contraste). para estado error.|
|`feedback.error.bg.tertiary`|`#fef2f2`|`#7f1d1d`|Fill (Frame/Shape)|Fondo suave (alert suave / highlight). para estado error.|
|`feedback.error.border`|`#ef4444`|`#f87171`|Stroke|Border para estado error.|
|`feedback.error.icon`|`#dc2626`|`#fca5a5`|Fill/Stroke (Icon)|Icon para estado error.|
|`feedback.error.text`|`#991b1b`|`#fecaca`|Text|Text para estado error.|


#### `feedback.info.*`
|Token|Light value|Dark value|Figma scope|Uso|
|---|---|---|---|---|
|`feedback.info.bg.primary`|`#0284c7`|`#38bdf8`|Fill (Frame/Shape)|Fondo fuerte (badge/etiqueta sólida). para estado info.|
|`feedback.info.bg.secondary`|`#0369a1`|`#0ea5e9`|Fill (Frame/Shape)|Fondo fuerte oscuro (pressed/alto contraste). para estado info.|
|`feedback.info.bg.tertiary`|`#f0f9ff`|`#0c4a6e`|Fill (Frame/Shape)|Fondo suave (alert suave / highlight). para estado info.|
|`feedback.info.border`|`#0ea5e9`|`#38bdf8`|Stroke|Border para estado info.|
|`feedback.info.icon`|`#0284c7`|`#7dd3fc`|Fill/Stroke (Icon)|Icon para estado info.|
|`feedback.info.text`|`#075985`|`#bae6fd`|Text|Text para estado info.|


---

## 5) Recetas rápidas (para una diseñadora junior)

### 5.1 Estoy diseñando una pantalla normal
- Fondo general: `surface.canvas`
- Sección/toolbar suave: `surface.subtle`
- Card/panel: `surface.elevated`
- Divisor: `border.muted`
- Texto:
  - título: `content.text.primary.default`
  - descripción: `content.text.secondary.default`
  - hint: `content.text.secondary.muted`
- Iconos: `content.icon.default`

### 5.2 Estoy diseñando un botón primario (filled)
- Fondo: `action.primary.bg.default`
- Texto/Icono: `content.text.on.primary` / `content.icon.on.primary`
- Hover: `action.primary.bg.hover`
- Active: `action.primary.bg.active`
- Disabled: `action.primary.bg.disabled` + `action.primary.text.disabled` + `action.primary.icon.disabled`
- Focus: `ring.action.inner` + `ring.action.outer` (además de `border.focus` si aplica)

### 5.3 Estoy mostrando un error de validación
- Borde del input: `feedback.error.border` o `border.feedback.error`
- Fondo sutil: `feedback.error.bg.tertiary`
- Texto: `feedback.error.text`
- Icono: `feedback.error.icon`
- Focus + error: podés usar `ring.feedback.error`

---

## 6) Cómo evoluciona el sistema sin romper nada

### 6.1 Cambiar el color primary (rebrand)
1. Elegí la nueva escala en primitives (por ejemplo, en vez de `indigo.*` usar `sky.*`).
2. Actualizá **solo** referencias en:
   - `action.primary.*`
   - `border.brand`
   - `content.icon.brand`
   - (y cualquier otro token de “marca” que exista)
3. **No toques** componentes ni pantallas.

### 6.2 Cuándo agregar nuevos tokens
Agregá un token nuevo si:
- aparece un rol nuevo repetido (ej: “surface for data-grid zebra rows”),
- y no encaja bien en los roles actuales.

No agregues tokens si:
- es un caso único del componente (eso va en el componente, no en el sistema).

### 6.3 Regla de estabilidad
- Los nombres de tokens son API.
- Si renombrás, rompe diseño + código + documentación.
- Preferí **agregar** y luego deprecar (documentado) antes que renombrar.

---

## 7) Apéndice: “Scope” recomendado en Figma (resumen)

|Familia|Scope recomendado|Notas|
|---|---|---|
|`surface.*`|Fill|Fondos de layout y contenedores.|
|`content.text.*`|Text|Siempre.|
|`content.icon.*`|Fill/Stroke|Dependiendo del tipo de ícono (filled vs stroke).|
|`border.*`|Stroke|Bordes/divisores/contornos.|
|`ring.*`|Effect|Anillos de foco (efecto/halo).|
|`action.*.bg.*`|Fill|Fondos de botones/chips/etc.|
|`action.*.text.*`|Text|Texto dentro de controles.|
|`action.*.icon.*`|Fill/Stroke|Iconos dentro de controles.|
|`feedback.*`|Según token|`bg`=Fill, `text`=Text, `border`=Stroke, `icon`=Fill/Stroke.|

---

### Checklist rápido de calidad (antes de “dar por terminado”)

- [ ] ¿Los componentes usan semánticos y no primitives?
- [ ] ¿Primary/Secondary/Neutral/Danger funcionan en light y dark (contraste)?
- [ ] ¿Hay tokens duplicados con nombres distintos para el mismo rol?
- [ ] ¿Los estados `default/hover/active/disabled/focus` están completos para `action.primary`?
- [ ] ¿Los errores/warnings tienen `bg.tertiary` legible y borde visible?


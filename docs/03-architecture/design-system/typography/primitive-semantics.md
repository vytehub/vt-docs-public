---
sidebar_position: 1
title: Primitive & Semantic
---

# Primitive & Semantic Typography

> **Convención:** los **nombres** de tokens quedan en **inglés** (contrato con código).
> Esta documentación está en **español**.

---

## 1) Objetivo

Los typography tokens existen para que:

- Todos los MFEs usen **la misma escala tipográfica** (sin inventar valores).
- Podamos ajustar tamaño por **breakpoint** (desktop/tablet/mobile) sin tocar componentes.
- Diseño y código compartan un contrato: *un token = una decisión*.

---

## 2) Capas y responsabilidades

### A) Primitive Sizes (en otra colección)
Guarda **números reutilizables** (ej. `size.14`, `size.16`, `size.18`).

### B) Primitive Typography ([`primitive-typography.tokens.json`](primitive-typography.tokens.json))
Guarda piezas tipográficas **atómicas** (valores base, sin intención de UI):

- `font.family.*` → fuentes (Inter, JetBrains Mono)
- `font.weight.*` → pesos (400–700)
- `font.letterSpacing.*` → tracking (0 / tight / wide)
- `font.lineHeight.*` → line-height (tight / normal / relaxed)

✅ Este archivo ya incluye `font.lineHeight.*`. fileciteturn0file4

> **Pregunta frecuente:** “¿Primitive Typography debería referenciar Primitive Sizes?”  
> **No.** Primitives deberían ser valores crudos.  
> La relación con `size.*` va en **Semantic Typography**.

---

## 3) Semantic Typography por breakpoint

Archivos:
- [`semantic-typography.desktop.json`](semantic-typography.desktop.json)
- [`semantic-typography.tablet.json`](semantic-typography.tablet.json)
- [`semantic-typography.mobile.json`](semantic-typography.mobile.json)

Qué definen (en cada breakpoint):

### 3.1 Tamaños (alias a Primitive Sizes)
- `text.size.sm` → `{size.14}`
- `text.size.base` → `{size.16}`
- `text.size.lg` → `{size.18}` (en mobile lo bajamos a `{size.16}`)

### 3.2 Familias tipográficas
- `font.family.body` → `{font.family.sans}`
- `font.family.code` → `{font.family.mono}`

### 3.3 Pesos
- `font.weight.regular` → `{font.weight.regular}`
- `font.weight.medium` → `{font.weight.medium}` ✅ (UI labels / buttons)
- `font.weight.semibold` → `{font.weight.semibold}` (si lo usás)
- `font.weight.bold` → `{font.weight.bold}`

> Nota: antes solo teníamos regular/bold. Agregamos medium/semibold porque ya existen como primitives y son útiles en UI.

### 3.4 Line-height (el “bug típico”)
- `leading.height.tight` → `{font.lineHeight.tight}`
- `leading.height.normal` → `{font.lineHeight.normal}`

✅ Esto es lo correcto.  
`letterSpacing` NO es line-height: tracking es espacio entre letras; line-height es altura entre líneas.

### 3.5 Tracking (letter-spacing)
- `tracking.letterSpacing.normal|tight|wide` → `{font.letterSpacing.*}`

---

## 4) Cómo se usa en componentes (regla simple)

### Caso típico: NO crees tokens específicos por componente
Para la mayoría de componentes, consumís semantic directo:

- Texto del botón:
  - size → `text.size.sm`
  - weight → `font.weight.medium`
  - line-height → `leading.height.normal`
  - tracking → `tracking.letterSpacing.normal`

Los estados (hover/pressed/disabled) **no deberían cambiar tipografía**; cambian colores/opacity.

### Cuándo SÍ crear tokens tipográficos del componente
Crealos solo si el componente tiene una regla tipográfica propia y estable (ej. un `Badge` con tracking wide, o un `KPI` con escala especial) y querés exponer una **API visual** del componente.

Ejemplo:
- `component.badge.label.size = {text.size.sm}`
- `component.badge.label.tracking = {tracking.letterSpacing.wide}`

---

## 5) Recomendación práctica: Text Styles

Tokens te dan valores; **Text Styles** te dan combinaciones listas para usar.

Recomendación mínima:
- `Text/Body/Sm`
- `Text/Body/Base`
- `Text/Body/Lg`
- `Text/Heading/H1..H6` (cuando los necesites)

Cada style debería usar variables (no hardcode):
- family: `font.family.body`
- size: `text.size.*`
- weight: `font.weight.*`
- line-height: `leading.height.*`
- letter-spacing: `tracking.letterSpacing.*`

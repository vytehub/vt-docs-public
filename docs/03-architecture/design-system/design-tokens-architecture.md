---
sidebar_position: 1
title: Design Tokens Architecture
---

# Design Tokens Architecture

> **Nota:** los nombres de tokens se mantienen en **English**.  
> En Figma normalmente se ven con `/` (por ejemplo `gray/50`).  
> En código y docs a veces se usa `.` (por ejemplo `gray.50`). Es la misma idea: **path segments**.

## 1) Qué queremos lograr

Con este sistema buscamos:

- **Consistencia** visual entre todos los microfrontends.
- Soportar **Light/Dark** (y futuros themes) sin re-trabajar componentes.
- Permitir cambios de **branding** (ej. cambiar el “primary”) tocando lo mínimo.
- Tener una base lista para **automación** (export, sync, generación de CSS vars, etc).

---

## 2) Qué es un design token

Un **design token** es una variable nombrada que representa una decisión de diseño.

Ejemplos:

- `gray.50` (primitive)
- `surface.canvas` (semantic)
- `action.primary.filled.bg.hover` (semantic)
- `component.button.filled.bg.hover` (component-specific, *solo si hace falta*)

Un token **no** es “un color”: es una decisión con intención.

---

## 3) Capas y responsabilidades

### 3.1 Primitive tokens (valores base)

Son la “materia prima” del sistema.  
Ej: escalas de color, números de tamaños, familias tipográficas.

**Colecciones actuales (Figma):**
- **Primitive Colors**
- **Primitive Sizes**
- **Primitive Typography**

**Regla:** los primitives **no deberían usarse** directo en componentes/pantallas.  
Se usan para construir los **semantic tokens**.

---

### 3.2 Semantic tokens (intenciones)

Son los tokens que expresan el **rol** en UI: superficies, texto, borde, acciones, feedback.

**Colecciones actuales (Figma):**
- **Semantic Colors** (modes: `Light`, `Dark`)
- **Semantic Sizes** (modes: `Desktop`, `Tablet`, `Mobile`)
- **Semantic Typography** (modes: `Desktop`, `Tablet`, `Mobile`)

**Regla:** componentes y pantallas consumen **semantics**, no primitives.

---

### 3.3 Component-specific tokens (solo cuando vale la pena)

A veces un componente necesita una decisión propia, por ejemplo:

- Un `Button` con sombras especiales.
- Un `Card` con un borde/overlay específico.
- Un `Chip` con paddings distintos al resto de “controls”.

Ahí podés crear tokens tipo `component.button.*`.

**Regla práctica:**  
Creá tokens específicos de componente **solo si**:
1) el componente tiene un comportamiento visual *que no encaja bien* en `action.*`, `surface.*`, etc, o  
2) querés permitir **variantes muy distintas** sin romper el resto.

Si no, mantenete en semantic global (más simple).

---

## 4) Tokens vs Variants (Figma Components)

- **Tokens** = decisiones atómicas (color/size/typography).
- **Variants** = “configuraciones” de un componente (ej. `variant=filled|outlined`, `size=sm|md`, `state=default|hover|disabled`).

Los variants **usan tokens**.  
Los tokens **no** deberían “conocer” componentes (salvo los component-specific).

---

## 5) Checklist rápido de “está bien?”

- ✅ Semantics **referencian** primitives (alias), no hex/px hardcode.
- ✅ Light/Dark se resuelve cambiando **mode** (no duplicando componentes).
- ✅ Responsive se resuelve cambiando **mode** (Desktop/Tablet/Mobile).
- ✅ Los componentes consumen `surface.*`, `text.*`, `action.*`, etc.

---

## 6) Qué global tokens suelen faltar (próximos pasos)

No es obligatorio tenerlos ya, pero son los próximos candidatos:

- `opacity.*` (disabled, subtle, overlay)
- `shadow.*` / `elevation.*` (card, popover, modal)
- `motion.duration.*` + `motion.easing.*` (animaciones)
- `zIndex.*` (modal, popover, toast, tooltip)
- `breakpoint.*` (si querés documentarlo como tokens)


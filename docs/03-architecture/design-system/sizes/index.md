---
sidebar_position: 3
title: Sizes
---

# Primitive & Semantic Sizes

> **Objetivo:** que el equipo diseñe layouts y componentes con **consistencia** (misma “sensación” de espaciado, radios y alturas) y que el sistema pueda **evolucionar** sin redibujar todo.
>  
> **Regla de oro:** los componentes y pantallas **usan tokens semánticos**, nunca tamaños “a ojo”.

---

## 1) Qué es un size token (y por qué importa)

Un **size token** es un número con unidad (por ejemplo `16px`) que representa una **decisión del sistema**.

- Si ponés `padding: 16px` directo, estás “inventando” un valor.
- Si usás `layout.page.paddingX`, estás diciendo: “este es el padding estándar de página”.

Eso permite:
1. **Consistencia visual** (todo respira igual).
2. **Mantenibilidad** (cambiar un token actualiza todo lo que lo usa).
3. **Escalabilidad** (un nuevo diseñador tiene reglas claras).

---

## 2) Cómo está organizado este set

En Figma este set existe como **Semantic Sizes** con 3 modos: `Desktop`, `Tablet`, `Mobile`.

- [`Desktop.size.tokens.json`](Desktop.size.tokens.json)
- [`Tablet.size.tokens.json`](Tablet.size.tokens.json)
- [`Mobile.size.tokens.json`](Mobile.size.tokens.json)

Todos son **alias** a [`Primitive.size.tokens.json`](Primitive.size.tokens.json).


Este set se divide en:

### A) `size.*` (primitives) — ver [`Primitive.size.tokens.json`](Primitive.size.tokens.json)
La escala base en píxeles. Son “ladrillos”.  
Ejemplo: `size.16 = 16px`, `size.24 = 24px`.

- **Se pueden ajustar** si el sistema cambia (por ejemplo, si el producto necesita más densidad o más aire).
- **No se usan** directamente en componentes/pantallas (salvo casos muy técnicos, y con criterio).


### B) Semantic sizes
Tokens con intención (por ejemplo “padding de página”, “radio de card”, “alto de control”).

Archivos (uno por breakpoint):
- [`Desktop.size.tokens.json`](Desktop.size.tokens.json)
- [`Tablet.size.tokens.json`](Tablet.size.tokens.json)
- [`Mobile.size.tokens.json`](Mobile.size.tokens.json)

> Importante: aunque existan 3 archivos, **los nombres de tokens se mantienen**, lo que cambia son algunos valores en `layout.*`.

---

## 3) Principios de diseño que estamos aplicando

### 3.1 Grid base (4px)
La mayoría de tokens siguen una grilla de **4px** (4, 8, 12, 16, 20, 24, 32...).  
Esto hace que todo “encaje” visualmente.

### 3.2 Sin decimales
Evitamos valores tipo `10.5px` o escalas con `1.25` porque:
- complica el mantenimiento,
- hace más difícil elegir,
- suele ser un “error acumulado” de decisiones sueltas.

### 3.3 Primitives ≠ semantic
- **Primitives:** números base.
- **Semantic:** reglas del sistema (“para esto se usa este tamaño”).

---

## 4) Categorías semánticas y cómo usarlas

## 4.1 `space.*` — Espaciado general

Estos tokens se usan para:
- gaps entre elementos,
- paddings internos genéricos,
- separación vertical entre bloques dentro de un componente.

**Nunca uses valores inventados:** elegí el token más cercano.

|Token|Valor (ref)|Cuándo usarlo|
|---|---:|---|
|`space.2xs`|`{size.4}`|Separaciones micro (icono-texto, chips compactos)|
|`space.xs`|`{size.8}`|Separación compacta entre elementos|
|`space.sm`|`{size.12}`|Separación estándar en componentes densos|
|`space.md`|`{size.16}`|Separación estándar general (la más común)|
|`space.lg`|`{size.20}`|Componentes con más aire|
|`space.xl`|`{size.24}`|Bloques internos grandes|
|`space.2xl`|`{size.32}`|Separación entre secciones pequeñas|
|`space.3xl`|`{size.40}`|Separación entre secciones medianas|
|`space.4xl`|`{size.48}`|Separación entre secciones grandes|
|`space.5xl`|`{size.64}`|Hero/zonas muy aireadas|
|`space.6xl`|`{size.80}`|Casos excepcionales (landing, grandes vacíos)|

> Tip práctico: en un layout normal, si dudás, arrancá con `space.md` y ajustá a `sm` o `lg`.

---

## 4.2 `layout.*` — Reglas de layout (responsive)

Estos tokens definen el “esqueleto” de pantalla.

### `layout.page.*`
- `layout.page.paddingX` y `paddingY`: padding de la página completa.
- `layout.page.maxContentWidth`: ancho máximo del contenido (para no estirar demasiado).

### `layout.section.gapY`
Separación vertical **entre secciones** (por ejemplo entre “Perfil” y “Preferencias”).

### `layout.grid.gutter`
Espacio entre columnas/tiles en grillas.

### `layout.card.padding`
Padding interno estándar de Cards / Panels.

### `layout.modal.*`
Padding y maxWidth sugeridos para modales.

#### Tabla rápida por breakpoint

|Token|Mobile|Tablet|Desktop|
|---|---:|---:|---:|
|`layout.page.paddingX`|`{size.16}`|`{size.24}`|`{size.32}`|
|`layout.section.gapY`|`{size.24}`|`{size.32}`|`{size.40}`|
|`layout.grid.gutter`|`{size.16}`|`{size.24}`|`{size.24}`|
|`layout.card.padding`|`{size.16}`|`{size.20}`|`{size.24}`|

> **Regla:** si estás diseñando una pantalla Mobile, usás el set Mobile; si es Desktop, el set Desktop.
> Los tokens tienen el mismo nombre: solo cambia el set que está activo.

---

## 4.3 `radius.*` — Radios (esquinas)

Los radios controlan el “carácter” del sistema: más redondeado = más amigable.

|Token|Valor (ref)|Cuándo usarlo|
|---|---:|---|
|`radius.none`|`{size.0}`|Casos técnicos (tablas, divisores, layouts muy rectos)|
|`radius.sm`|`{size.4}`|Componentes pequeños (chips)|
|`radius.md`|`{size.8}`|Inputs, buttons (base)|
|`radius.lg`|`{size.12}`|Cards, panels|
|`radius.xl`|`{size.16}`|Modales, contenedores grandes|
|`radius.2xl`|`{size.24}`|Casos hero / contenedores muy grandes|
|`radius.full`|`{size.9999}`|Pills (botón pill, tags totalmente redondeadas)|

---

## 4.4 `stroke.*` — Grosor de bordes y focus ring

|Token|Valor (ref)|Cuándo usarlo|
|---|---:|---|
|`stroke.hairline`|`{size.1}`|Divisores sutiles, bordes finos|
|`stroke.thin`|`{size.2}`|Bordes estándar (inputs)|
|`stroke.thick`|`{size.4}`|Énfasis (selección fuerte)|
|`stroke.focusRing`|`{size.2}`|Contorno de foco accesible (teclado)|

> Nota: el **color** del focus ring viene de semantic colors (`border.focus`), el **grosor** viene de `stroke.focusRing`.

---

## 4.5 `component.*` — Tamaños estándar de componentes

Esto NO es “tokens del Button”. Es un set genérico para **cualquier control** (Button, Input, Select, etc.).

### `component.control.height.*`
Alturas estándar para controles:
- `sm` (compacto), `md` (default), `lg` (grande), `xl` (muy grande).

### `component.control.paddingX / paddingY`
Padding interno recomendado para controles.

### `component.icon.size.*`
Tamaños estándar de iconos.

|Token|Valor (ref)|Cuándo usarlo|
|---|---:|---|
|`component.control.height.sm`|`{size.32}`|Toolbars densas, mobile compacto|
|`component.control.height.md`|`{size.40}`|Default general|
|`component.control.height.lg`|`{size.48}`|CTA grande / accesibilidad|
|`component.control.height.xl`|`{size.56}`|Casos especiales (hero)|

---

## 5) Qué NO hacer (errores comunes)

1. **No uses tamaños hardcodeados** (“le pongo 17px porque se ve bien”).  
   Elegí el token más cercano y mantené consistencia.

2. **No uses primitives en pantallas** (salvo excepciones muy justificadas).  
   Si necesitás un nuevo caso, creá un token semántico.

3. **No “inventes” un nuevo radio** por un componente.  
   Si el radio no existe, el sistema necesita discutirlo como regla.

---

## 6) Flujo recomendado de trabajo (para equipo chico)

1. Diseñás usando `layout.*`, `space.*`, `radius.*`, `component.*`.
2. Si un diseño “no entra” con esos tokens, lo discutimos y **extendemos el set** (no hardcodeamos).
3. Cuando queramos “más aire” o “más densidad”, tocamos:
   - primero `layout.*` (afecta pantallas),
   - luego `space.*` (afecta micro-layout),
   - y solo al final `size.*` si cambiara la grilla base.



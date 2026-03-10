---
title: Checkbox
sidebar_position: 3
---

# Checkbox

El **Checkbox** es un control binario o ternario utilizado para activar/desactivar opciones en formularios, listas o filtros.  

Representa datos, no acciones.

Este documento es la fuente de verdad para:

- Implementación en Figma  
- Implementación en Angular / React  
- Uso de tokens  
- Reglas de comportamiento

---

## Responsibility

El Checkbox:

- Captura y muestra estados seleccionados: `checked`  `unchecked` / `indeterminate` 
- Expone Interaction State (`default`, `hover`, `focus`, `disabled`)  
- Muestray expresa sizes claras: 'sm' | 'md' | 'lg'
  checked: boolean
  indeterminate: boolean
- Está basado en tokens semánticos  

El Checkbox **NO**:

- Representa acciones directas (usa Buttons para CTA)  
- Hardcodea colores  
- Mezcla semántica de Button dentro del Checkbox  

---

## State Model

El sistema de estados del Checkbox se compone de **dos dimensiones independientes**:

### 1. Interaction State

| State    | Descripción                                              |
|----------|----------------------------------------------------------|
| default  | Estado base                                              |
| hover    | Estado cuando el usuario pasa el mouse sobre el checkbox |
| focus    | Estado cuando el checkbox recibe foco (keyboard/tab)     |
| disabled | Estado inhabilitado; bloquea interacción y foco          |

**Reglas**

- `disabled` bloquea la interacción y no es focusable  
- `focus` debe mostrar siempre un **ring visible**  

---

### 2. Data State (Checked / Unchecked / Indeterminate)

| Data State    | Aplicación                                                          |
|---------------|---------------------------------------------------------------------|
| unchecked     | Checkbox sin selección                                              |
| checked       | Checkbox activado                                                   |
| indeterminate | Checkbox parcialmente seleccionado (ej. selección parcial de grupo) |

**Impacta:**

- Visualización del check / guion (para indeterminate)  
- Colores del icono (`content.icon.*`)  

---

## Sizes

El size controla dimensiones del checkbox, además de iconos y spacing.

| **Size** | **Box (w × h)** |
|------|-------------|
| lg   | `24 × 24 px`  |
| md   | `20 × 20 px`  |
| sm   | `16 × 16 px`  |

> Width del checkbox es control de layout, no del componente.

---

## Internal Structure

```text
Checkbox Container
├─ Check Icon (optional, hidden if unchecked)
├─ Native Input
├─ Label (optional)
├─ Description / Helper Text (optional)

---

Reglas estructurales:

- Icono debe escalar según size (content.icon.*)
- Label y description alineados con padding del container (space.*)
- Hit area se ajusta según size
- Debe usarse Auto Layout en Figma / Flex en Angular
- Counter/description no debe alterar altura del checkbox

```

## Token Mapping

El Checkbox utiliza únicamente **tokens semánticos**.

**Variant**: `unchecked`

| **State**      | **Background**      | **Border**        | **Icon** |  **Ring**     | 
|----------------|---------------------|-------------------|----------|---------------| 
| default        | `surface.canvas`    | `border.default`  | -        | -             |
| hover          | `surface.canvas`    | `border.hover`    | -        | -             |
| focus          | `surface.canvas`    | `border.focus`    | -        | `ring.focus`  |
| disabled       | `surface.disabled`  | `border.disabled` | -        | -             |


**Variant**: `checked`

| **State**  | **Background**                      | **Border**        | **Icon**              |  **Ring**     | 
|------------|-------------------------------------|-------------------|-----------------------|---------------| 
| default    | `action.primary.filled.bg.default`  | `border.default`  | content.icon.inverse  | -             |
| hover      | `action.primary.filled.bg.hover`    | `border.hover`    | content.icon.inverse  | -             |
| focus      | `action.primary.filled.bg.default`  | `border.focus`    | content.icon.inverse  | `ring.focus`  |
| disabled   | `surface.disabled`                  | `border.disabled` | content.icon.disabled | -             |


**Variant**: `indeterminate` 

| **State**  | **Background**                      | **Border**        | **Icon**                 |  **Ring**     | 
|------------|-------------------------------------|-------------------|--------------------------|---------------| 
| default    | `action.primary.filled.bg.default`  | `border.default`  | content.icon.inverse     | -             |
| hover      | `action.primary.filled.bg.hover`    | `border.hover`    | content.icon.inverse     | -             |
| focus      | `action.primary.filled.bg.default`  | `border.focus`    | content.icon.inverse     | `ring.focus`  |
| disabled   | `surface.disabled`                  | `border.disabled` | content.icon.disabled    | -             |

---

## Implementation Contract

### Angular MUST

- No hardcodear colores
- Usar solo tokens semánticos
- No vincular width con size
- No depender de `:has()`
- Implementar estados mediante host classes explícitas

### Figma MUST

- Usar variables (Design Tokens)
- No usar HEX
- Modelar State, Validation y Size como propiedades independientes
- Mantener Width separado de Size
- Evitar duplicación visual innecesaria

---

## Combinatorics

- Interaction (4) × Data State (3) × Size (3) = 36 combinaciones posibles
- No todas deben renderizarse, pero comportamiento debe estar definido

---

## Future Extensions

- CheckboxGroup (multi-select)
- Floating label derivado (si aplica)
- Animaciones de check / indeterminate
- Control con counter / helper text

---

## Ejemplo de implementación (React / Angular conceptual)

```tsx
<label
  className="checkbox-container"
  aria-disabled={disabled}
  aria-checked={indeterminate ? "mixed" : checked}
>
  <input
    type="checkbox"
    checked={checked}
    disabled={disabled}
  />
  <span className="checkbox-icon">
    {indeterminate ? "—" : checked && "✓"}
  </span>
  <span className="checkbox-label">{label}</span>
</label>

---
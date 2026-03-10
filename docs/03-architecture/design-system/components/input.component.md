---
title: Input
sidebar_position: 2
---

# Input

El componente **Input** es un control de entrada de datos utilizado para
capturar valores proporcionados por el usuario (text, email, password,
tel, number).

Representa **datos**, no acciones.

Este documento es la **fuente de verdad** para:

-   Implementación en Figma
-   Implementación en Angular
-   Uso de tokens
-   Reglas de comportamiento


---

##  Responsibility

El Input:

-   Captura y muestra datos editables
-   Expone Interaction State
-   Expone Validation State
-   Es accesible
-   Es predecible
-   Está basado en tokens semánticos

El Input NO:

-   Representa acciones (Button)
-   Codifica decisiones de layout como width fijo
-   Hardcodea colores
-   Usa `action.*` tokens

------------------------------------------------------------------------

## State Model

El sistema de estados del Input se compone de **tres dimensiones
independientes**.

------------------------------------------------------------------------

### 1. Interaction State

-   default
-   hover
-   focus
-   disabled
-   readonly

### Reglas

-   `readonly` ≠ `disabled`
-   `disabled` bloquea interacción y no es focusable
-   `readonly` permite focus pero no permite edición
-   `focus` debe mostrar siempre un ring visible
-   `disabled` cancela estilos visibles de Validation

------------------------------------------------------------------------

### 2. Validation State

-   none
-   error
-   success

### Reglas

-   Validation sobrescribe **border y ring**
-   Validation NO cambia el background por defecto
-   El background solo puede cambiar si se utilizan tokens explícitos
    `feedback.*.subtle`

------------------------------------------------------------------------

### 3.  Data State

-   empty
-   filled

### Afecta

-   Visibilidad del placeholder
-   Comportamiento futuro de floating label
-   Énfasis de icon (opcional)
-   Visibilidad del counter

------------------------------------------------------------------------

## Appearance

El Input soporta variantes de Appearance:

-   outlined (default)
-   filled (future)
-   subtle (future)

Outlined es la implementación base.

------------------------------------------------------------------------

## Sizes

Size controla el ritmo vertical y la tipografía.

Size | Height   | horizontal padding | Typography
---- | ------------------| ------------------------ | ----------
lg   | 48px              | 20                       | body.lg
md   | 40px              | 16                       | body.base
sm   | 32px              | 12                       | body.sm

Size controla:

-   Height
-   Padding
-   Typography
-   Icon size
-   Espaciado del counter

Size NO controla width.

------------------------------------------------------------------------

##  Width

Width es una decisión de layout.

Opciones soportadas:

-   intrinsic (default)
-   full

No se permiten min/max width fijos dentro del componente.

------------------------------------------------------------------------

##  Internal Structure

Input Container  
├─ Leading Icon (optional)  
├─ Native Input  
├─ Trailing Icon (optional)  
├─ Counter (optional)

### Reglas estructurales

-   El padding se ajusta cuando hay icons
-   El tamaño del icon escala según Size
-   El counter reduce el espacio disponible del texto
-   Debe usarse Auto Layout en Figma y flex en Angular
-   Los icons deben usar `content.icon.*` tokens

------------------------------------------------------------------------

## Token Mapping

El Input utiliza únicamente tokens semánticos.

------------------------------------------------------------------------

### 1. Default (Validation = none)

bg → `surface.canvas`  
border → `border.default`  
text → `content.text.primary`  
ring → `focus`
placeholder → `content.text.placeholder`

### 2. Hover (Validation = none)

bg → `surface.default`  
text → `content.text.primary`    
border → `border.hover`  

**Hover NO debe usar `action.*` tokens**.

### 3. Focus (Validation = none)

bg → `surface.default`  
text → `content.text.focus`  
border → `border.focus`  
ring → `ring.focus` (grosor stroke.focusRing)

**Focus debe ser siempre visible**.

### 4. Disabled

bg → `surface.disabled` 
border → `border.disabled` 
text → `content.text.primary.disabled`  
placeholder → `content.text.placeholder.disabled`

**Disabled elimina visuales de Validation**.

------------------------------------------------------------------------

## Validation Mapping

**Regla de oro:** validation se superpone al state cuando aplica (`ej. error > focus`). disabled cancela validations visibles.

------------------------------------------------------------------------

###  Error

border → `border.feedback.error`  
ring (si focus) → `ring.feedback.error`
helper text → `content.text.feedback.error` 
icon (si aplica) → `content.icon.feedback.error`  

**Background no cambia por default (salvo feedback.error.bgSubtle si decides añadirlo)**.

------------------------------------------------------------------------

###  Success

border → `border.feedback.success`  
ring (si focus) → `ring.feedback.success`  
helper text → `content.text.feedback.success`  

**Background no cambia por default (salvo feedback.error.bgSubtle si decides añadirlo)**.

------------------------------------------------------------------------

##  Accessibility

-   Debe exponer aria-label o label asociado
-   Placeholder NO reemplaza al label
-   Focus ring visible obligatorio
-   Debe cumplir contraste mínimo WCAG AA
-   Readonly debe ser focusable
-   Disabled no debe ser focusable

------------------------------------------------------------------------

##  Implementation Contract

### Angular MUST

-   No hardcodear colores
-   No usar `action.*` tokens
-   No mezclar semántica de Button dentro de Input
-   No vincular width con Size
-   No depender de `:has()` para estilos de estado
-   No usar ngModel internamente al implementar ControlValueAccessor
-   Implementar estados mediante host classes explícitas

### Figma MUST

-   Usar Variables (Design Tokens)
-   No usar colores HEX
-   Modelar Interaction, Validation y Size como propiedades
    independientes
-   Incluir readonly
-   Mantener Width separado de Size
-   Evitar duplicación visual innecesaria

------------------------------------------------------------------------

##  Combinatorics

Interaction (5) × Validation (3) × Size (3) = 45 combinaciones posibles.

No todas deben renderizarse visualmente, pero el comportamiento debe
estar definido.

------------------------------------------------------------------------

##  Future Extensions

-   Floating label
-   Password toggle component
-   Phone input derivado
-   Masked input
-   Trailing action input (componente separado)





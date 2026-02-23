---
sidebar_position: 15
title: Typography Styles
---

# Typography Styles

Guía oficial de **Text Styles** del Design System.\
Define qué estilos existen, qué tokens usan y cómo deben aplicarse.

------------------------------------------------------------------------

# Objetivo

-   Normalizar tipografía entre Figma y código.
-   Separar responsabilidades (Typography vs Color vs Component).
-   Evitar valores hardcodeados.
-   Facilitar integración con Tailwind y herramientas de IA.

------------------------------------------------------------------------

# Principios

1.  Un **Text Style representa un rol semántico**, no un tamaño aislado.
2.  Nunca usar valores manuales (px, %, hex) en componentes.
3.  Los componentes **seleccionan un Text Style**, no redefinen
    tipografía.
4.  Typography define forma. Color define apariencia.

------------------------------------------------------------------------

# Roles Tipográficos

## Body

Texto general de UI y contenido.

Usa: - font.family.body - font.weight.regular - text.size.\* -
leading.height.normal - tracking.letterSpacing.normal

------------------------------------------------------------------------

## Label

Texto de componentes interactivos (Buttons, Tabs, Chips).

Usa: - font.family.body - font.weight.medium - text.size.\* -
leading.height.tight - tracking.letterSpacing.normal

------------------------------------------------------------------------

## Heading

Jerarquía visual (títulos de pantalla y secciones).

Usa: - font.family.body - font.weight.bold (o semibold) - text.size.\*
(según nivel) - leading.height.tight - tracking.letterSpacing.normal

------------------------------------------------------------------------

## Code

Contenido técnico (snippets, IDs, tokens).

Usa: - font.family.code - font.weight.regular - text.size.sm/base -
leading.height.normal - tracking.letterSpacing.normal

------------------------------------------------------------------------

# Set Oficial de Text Styles

## Body

-   Text/Body/Sm
-   Text/Body/Base
-   Text/Body/Lg

## Label

-   Text/Label/Sm
-   Text/Label/Base
-   Text/Label/Lg

## Heading

-   Text/Heading/H1
-   Text/Heading/H2
-   Text/Heading/H3
-   Text/Heading/H4

## Code

-   Text/Code/Inline
-   Text/Code/Block

------------------------------------------------------------------------

# Mapeo de Tokens

  -----------------------------------------------------------------------------------------------------------------------
  Text Style         Family             Weight                Size              Line Height             Tracking
  ------------------ ------------------ --------------------- ----------------- ----------------------- -----------------
  Text/Body/Sm       font.family.body   font.weight.regular   text.size.sm      leading.height.normal   tracking.normal

  Text/Body/Base     body               regular               base              normal                  normal

  Text/Body/Lg       body               regular               lg                normal                  normal

  Text/Label/Sm      body               medium                sm                tight                   normal

  Text/Label/Base    body               medium                base              tight                   normal

  Text/Label/Lg      body               medium                lg                tight                   normal

  Text/Heading/H1    body               bold                  text.size.3xl\*   tight                   normal

  Text/Heading/H2    body               bold                  text.size.2xl\*   tight                   normal

  Text/Heading/H3    body               bold                  text.size.xl\*    tight                   normal

  Text/Heading/H4    body               bold                  text.size.lg      tight                   normal

  Text/Code/Inline   font.family.code   regular               sm                normal                  normal

  Text/Code/Block    code               regular               base              normal                  normal
  -----------------------------------------------------------------------------------------------------------------------

*Si no existen tamaños xl/2xl/3xl, pueden agregarse en Semantic
Typography.*

------------------------------------------------------------------------

# Uso en Componentes

## Button

Debe usar: - Text/Label/Sm → size=sm - Text/Label/Base → size=md -
Text/Label/Lg → size=lg

El componente NO define: - font-family - font-size - line-height -
tracking

Solo selecciona el Text Style correspondiente.

------------------------------------------------------------------------

# Lo que NO pertenece a Typography

❌ Colores\
❌ Estados (hover, focus, disabled)\
❌ Uppercase (decisión del componente)\
❌ Opacidad\
❌ Sombras

------------------------------------------------------------------------

# Gobernanza

-   Nuevos tamaños deben agregarse como tokens primero.
-   No crear Text Styles ad-hoc.
-   No modificar valores manualmente dentro de un componente.
-   Toda tipografía debe estar centralizada.

------------------------------------------------------------------------

# Resultado Esperado

Con esta estructura:

-   Figma usa Text Styles.
-   Tailwind usa tokens equivalentes.
-   El Toolkit consume roles claros.
-   La IA puede generar código sin ambigüedad.

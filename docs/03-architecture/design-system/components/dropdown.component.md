---
title: Dropdown
sidebar_position: 3
---

# Dropdown Panel (Popover Architecture)

Documento condensado y accionable para diseñadores y desarrolladores.  
Basado en la misma metodología de **Input / Button / Checkbox / Option**: foco en tokens semánticos, estructura interna estable, accesibilidad, y contrato de implementación.

---

## ¿Qué problema resuelve?

El Dropdown necesita mostrar una lista de opciones sin romper el layout del contenedor que lo contiene.  
En muchas interfaces el trigger del dropdown se encuentra dentro de contenedores con `overflow`, `z-index` o layouts complejos que pueden cortar o limitar la visualización del panel.

Para resolver esto, el panel del Dropdown se implementa como **Popover** (panel flotante).

Esto permite:

- Evitar clipping por `overflow: hidden`
- Controlar correctamente el `z-index`
- Posicionar el panel dinámicamente
- Mantener separado el trigger del contenido del panel
- Reutilizar la misma arquitectura para **Dropdown, Combobox y Select**

---

## Terminología

| Término   | Descripción                                      |
|-----------|--------------------------------------------------|
| Trigger   | Elemento que abre el dropdown (button o input)   |
| Popover   | Panel flotante que contiene la lista de opciones |
| Listbox   | Contenedor accesible que agrupa las opciones     |
| Option    | Item seleccionable dentro de la lista            |
| Placement | Posición del popover respecto al trigger         |

## Estructura del componente

```
Dropdown
├─ Trigger (Input)
└─ Popover
   └─ Listbox
      ├─ Option
      ├─ Option
      └─ Option
```

## Descripción de cada parte

**Trigger**

Elemento interactivo que controla la apertura del dropdown. (hereda los estados de Input / Select.)

Puede ser:

- Button (dropdown clásico)
- Input (combobox)


Responsabilidades:

- Abrir / cerrar el panel
- Exponer estado `aria-expanded`
- Conectar con el panel mediante `aria-controls`
- Permitir state de validation (error/success) si es un Input

## Semantic Tokens Trigger

| State      | Background             | Text                    | Border            | Icon                               |  Ring         | 
|------------|------------------------|-------------------------|-------------------|------------------------------------|---------------| 
| default    | `surface.canvas`       | `content.text.default`  | `border.default`  | content.icon.default  (true|false) | -             |
| hover      | `surface.canvas`       | `content.text.default`  | `border.hover`    | content.icon.default  (true|false) | -             |
| focus      | `surface.canvas`       | `content.text.default`  | `border.focus`    | content.icon.default  (true|false) | `ring.focus`  |
| disabled   | `surface.disabled`     | `content.text.disabled` | `border.disabled` | content.icon.disabled (true|false) | -             |

---

**Popover**

Panel flotante que contiene la lista de opciones.

Características:

- Se renderiza fuera del flujo del layout
- Puede posicionarse dinámicamente
- Puede tener scroll interno
- Puede limitar su altura máxima

Propiedades comunes:

- placement (bottom / top / auto)
- alignment (start / center / end)
- max-height
- offset respecto al trigger

## Semantic Tokens Popover

| State      | Background             | Border            |
|------------|------------------------|-------------------|
| default    | `surface.canvas`       | `border.default`  |

---

**Listbox**

Contenedor semántico de las opciones.

Responsabilidades:

- Manejar navegación por teclado
- Exponer roles accesibles
- Gestionar el focus activo

Role accesible:

role="listbox"

## Semantic Tokens listbox

| State      | Background   | Border            |
|------------|--------------|-------------------|
| default    | -            | `border.default`  |

---

**Option**

Elemento seleccionable dentro del dropdown.

### Qué problema resuelve
- Provee un patrón reutilizable para cada item de listas de selección (dropdowns, comboboxes, menús, listas con multi-select).  
- Normaliza la presentación (label, optional description, optional checkbox, trailing icons) y los estados (focus / hover / selected / disabled).  
- Evita duplicación de estilos y reduce la complejidad en Figma y en código al exponer propiedades en vez de variantes planas.

Puede soportar:

- estado seleccionado
- estado disabled
- estado hover
- iconos o checkbox

Roles accesibles:

role="option"
aria-selected="true | false"

## Semantic Tokens Option

| State      | Background             | Text                    | Icon                                  |  Ring         | 
|------------|------------------------|-------------------------|---------------------------------------|---------------| 
| default    | `surface.canvas`       | `content.text.default`  | `content.icon.default`   (true|false) | -             |
| hover      | `surface.canvas`       | `content.text.default`  | `content.icon.default`   (true|false) | -             |
| focus      | `surface.canvas`       | `content.text.default`  | `content.icon.default`   (true|false) | `ring.focus`  |
| disabled   | `surface.disabled`     | `content.text.disabled` | `content.icon.disabled`  (true|false) | -             |

---

## Comportamiento

### Apertura

El dropdown se abre mediante:

- click en el trigger
- tecla `Enter`
- tecla `Space`
- `ArrowDown` (abre y enfoca primera opción)

---

### Cierre

El dropdown se cierra mediante:

- selección de una opción
- tecla `Escape`
- click fuera del componente
- pérdida de foco (según implementación)

---

## Principios de diseño (resumen)

- **Separación de concerns**: Trigger (control) ≠ Panel (lista).  
- **Tokens semánticos**: usar `surface.*`, `border.*`, `content.*`, `ring.*`, `feedback.*`, `action.*` solo cuando corresponde.  
- **Estructura estable**: nodos opcionales siempre presentes en el DOM/Figma, solo ocultos/mostrados.  
- **Accesible por defecto**: ARIA roles y keyboard-first UX.  
- **Composabilidad**: funciona con `Option` y `Checkbox` (reusar componentes).

---

## Estructura interna (DOM / Figma)

Dropdown Root
├─ Trigger (Input)
├─ Popup Layer (portal)         // position, z-index
│   └─ Panel (auto layout vertical)
│       ├─ Search (optional, for combobox)
│       ├─ Options List (scrollable)
│       │   ├─ Option (reusable component)
│       │   ├─ Option
│       └─  └─ ...        
└─ 

--- 

## Consideraciones de Layout

El panel debe:

- alinearse con el trigger
- evitar salirse del viewport
- poder invertir su posición si no hay espacio

--- 

## Reglas de implementación

Para mantener consistencia en el sistema:

1. El panel debe renderizarse como **popover flotante**
2. El dropdown no debe depender del layout del contenedor padre
3. Las opciones deben ser reutilizables como componente independiente (`Option`)
4. El mismo patrón debe funcionar para:

- Dropdown
- Combobox
- Select
- Multi-select

---

### Nota de arquitectura

El uso de Popover permite reutilizar la misma infraestructura para varios componentes interactivos del sistema de diseño.
Esto asegura consistencia entre:

- Dropdown
- Combobox
- Autocomplete
- Context Menu
- Select

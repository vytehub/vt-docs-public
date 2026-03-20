---
title: Flow 16 — Timeline Experience (Stream & Graph Views)
description: >
  El usuario accede al tab Timeline y visualiza su tiempo en un stream scrolleable
  vertical (vista principal) o en un grafo tipo git-merge (vista alternativa).
  Puede ver, confirmar, rechazar y gestionar bookings/eventos inline. Si tiene
  Agreements, puede mergear timelines ajenos y verlos superpuestos con controles
  Solo/Mute/Hide por branch.
status: draft
version: 1
last-reviewed: 2026-03-20
---

# Flow 16 — Timeline Experience (Stream & Graph Views)

## 1. Summary

| | |
|---|---|
| **Goal** | Que el usuario visualice y gestione su tiempo de forma fluida mediante un stream vertical scrolleable (sin calendario grid) y opcionalmente un grafo tipo git-merge, con acciones inline sobre bookings y eventos. |
| **Actors** | Usuario autenticado (consumer, provider, o ambos); Delegado con ManageSchedule (via Agreement) |
| **Out of scope** | Creación de ConflictRules (Flow 08), booking flow completo (Flow 02), reagendamiento y cancelación (Flow 11), creación de offerings (Flow 05/06) |
| **Surfaces** | `vt-bookings-mfe`: `/timeline` (stream), `/timeline/graph` (git-merge view) |
| **Decision ref** | ADR-0005 (Navigation Architecture — Timeline es tab 2) |

---

## 2. Domain Context

### El Timeline no es un calendario

El Timeline es el **motor de tiempo** de VyteMerge (ver `00-core-domain/04-bounded-contexts/02-supply/01.timelines/01.timeline.md`). No se renderiza como grilla de horas. Se renderiza como un **stream continuo** donde solo aparece lo que importa.

| Calendario tradicional | Timeline VyteMerge |
|---|---|
| Grilla fija de horas (7am–9pm) | Stream de eventos; espacios vacíos colapsados |
| Un contexto por vista | Múltiples branches (personal, laboral, familia) mergeados |
| Visual primero | Semántico primero — "qué pasa y por qué" |
| Click para actuar | Acciones inline en el stream (confirmar, rechazar) |
| Vista estática | Scroll infinito bidireccional (pasado ↔ futuro) |

### Branches = Timelines

Cada Timeline es una **branch** en la metáfora git-merge:
- **Branch personal**: eventos privados, compromisos
- **Branch laboral**: bookings recibidos como provider
- **Branch compartida**: timelines de otros (via Agreements tipo Sharing)

Cuando dos branches tienen un evento al mismo tiempo (ej: booking del médico + booking del paciente), eso es un **merge point** — el punto donde dos timelines convergen.

### Bounded Contexts involucrados

| BC | Módulo backend | Rol en este flow |
|---|---|---|
| **Supply BC** | `Timeline` | Fuente de verdad: Events, Slots, ConflictRules |
| **Booking BC** | `Booking` | Bookings que generan Events en el Timeline |
| **Agreement BC** | `Agreements` | Habilita ver timelines ajenos (Sharing/Delegation) |
| **Communication BC** | `Notifications` | Notificaciones de conflictos, recordatorios |

---

## 3. Preconditions

1. Usuario autenticado con Profile activo.
2. Timeline por defecto creado al registrarse (Flow 00).
3. Si el actor es Delegado: Agreement activo con permiso `ManageSchedule`.
4. Para ver timelines ajenos: Agreement tipo `Sharing` con `VisibilityMode ≠ None`.

---

## 4. Trigger

- Usuario toca **tab Timeline** en la bottom bar (ADR-0005).
- Deep link a `/timeline` o `/timeline?date=2026-03-25`.
- Notificación de conflicto o booking pendiente → abre Timeline en la fecha del evento.

---

## 5. Main Flow

### Capacidad A — Stream View (vista principal)

#### A.1 — Carga inicial

1. Usuario abre `/timeline`.
2. Frontend solicita `GET /timelines/events?from={today-7d}&to={today+30d}`.
3. Backend retorna Events del Timeline principal + timelines compartidos (según Agreements activos).
4. Frontend renderiza el **stream vertical**:

```
┌─────────────────────────────────────┐
│  ← Mar 2026 →          [Today] [⋮] │  ← mini nav + overflow menu
├─────────────────────────────────────┤
│                                     │
│  ─── Hoy, jueves 20 ───────────── │  ← sticky date header
│                                     │
│  ┌─────────────────────────────┐   │
│  │ 🟢 09:00  Corte de pelo     │   │  ← booking (provider)
│  │    👤 María García          │   │     avatar del participante
│  │    45 min · Peluquería Centro│   │     duración + place
│  │               [✓ Confirmar] │   │     ← CTA inline
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ 🔵 14:00  Dentista          │   │  ← booking (consumer)
│  │    👤 Dr. Pérez             │   │
│  │    30 min · Consultorio Norte│   │
│  └─────────────────────────────┘   │
│                                     │
│  ─── Vie 21 ────────────────────── │  ← día sin eventos → colapsado
│     Sin eventos                     │
│                                     │
│  ─── Sáb 22 ───────────────────── │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ 🟡 10:00  Acto del colegio  │   │  ← evento privado
│  │    📅 Timeline: Familia      │   │     branch indicator
│  │    2h                        │   │
│  └─────────────────────────────┘   │
│                                     │
│  ─── Lun 24 ───────────────────── │
│  ...                                │
└─────────────────────────────────────┘
```

#### A.2 — Scroll infinito bidireccional

5. Al hacer scroll hacia abajo → frontend carga eventos futuros (`GET /timelines/events?from={lastDate}&to={lastDate+30d}`).
6. Al hacer scroll hacia arriba → frontend carga eventos pasados (`GET /timelines/events?from={firstDate-30d}&to={firstDate}`).
7. **No hay pull-to-refresh** — el scroll vertical ES la navegación temporal.
8. Datos se cargan por ventanas de 30 días con prefetch de la ventana siguiente.

#### A.3 — Sticky date headers

9. Las fechas actúan como **sticky headers** (patrón WhatsApp/iMessage).
10. El header de "Hoy" tiene highlight visual (bold + color accent).
11. **Días sin eventos se colapsan** a una línea mínima (patrón Fantastical):
    - Texto: "Vie 21 — Sin eventos" en gris claro.
    - Tap para expandir (muestra franja vacía opcional).
12. Al hacer scroll rápido, el sticky header siempre muestra en qué fecha estás.

#### A.4 — Floating "Today" button

13. Si el viewport actual NO incluye "Hoy", aparece un **floating button "Hoy"** (arriba o abajo según dirección del scroll):
    - Scroll hacia el futuro → botón "Hoy ↑" aparece arriba.
    - Scroll hacia el pasado → botón "Hoy ↓" aparece abajo.
14. Al tocar el botón → smooth scroll hasta la fecha actual.
15. Si "Hoy" ya es visible → botón desaparece.

---

### Capacidad B — Event cards en el stream

#### B.1 — Anatomía de un event card

16. Cada evento se renderiza como una card dentro del stream con:

| Elemento | Descripción |
|---|---|
| **Color lateral** | Indica la branch/timeline (ej: azul=personal, verde=laboral, naranja=familia) |
| **Hora** | Hora de inicio en formato local (con indicador de timezone si difiere) |
| **Título** | Nombre del evento o servicio |
| **Avatar** | Foto del otro participante (si aplica) |
| **Duración** | Barra visual proporcional + texto (ej: "45 min") |
| **Place** | Nombre del lugar (si tiene Place asociado) |
| **Branch badge** | Si hay múltiples branches visibles: mini chip con nombre del timeline |
| **Status indicator** | Dot de color: verde=confirmado, amarillo=pendiente, rojo=conflicto |
| **CTA primario** | Siempre visible (ej: "Confirmar" para pending bookings) |

#### B.2 — Tipos de eventos

17. El stream puede contener:

| Tipo | Color default | CTA | Origen |
|---|---|---|---|
| Booking recibido (provider) | Verde | Confirmar / Rechazar | Booking module |
| Booking hecho (consumer) | Azul | Ver detalle | Booking module |
| Evento privado | Gris | Editar / Eliminar | Timeline module |
| Booking pendiente | Amarillo | Confirmar / Rechazar | Booking module |
| Evento de timeline compartido | Color de la branch | — (solo lectura) | Agreement sharing |
| Conflicto detectado | Rojo | Resolver | ConflictDetection |

#### B.3 — Acciones inline

18. **CTA primario siempre visible** — el botón más importante está permanentemente en la card:
    - Booking `Pending` (provider) → "Confirmar" (verde).
    - Conflicto → "Resolver".
19. **Acciones secundarias** en mobile: **swipe right** para acción primaria (confirmar), **swipe left** para menú con más opciones (rechazar, reagendar, cancelar).
20. **Acciones secundarias** en desktop: hover revela toolbar con botones adicionales.
21. **Patrón "Undo over Confirm"** para acciones no destructivas:
    - Al confirmar un booking → acción se ejecuta inmediatamente + toast "Booking confirmado" con botón [Deshacer] visible por 5 segundos.
    - Al rechazar → modal de confirmación (acción destructiva, requiere confirm).
22. **Overflow menu** (⋮) para acciones poco frecuentes: reagendar, ver detalle completo, copiar link, agregar nota.

---

### Capacidad C — Multi-branch (timelines mergeados)

#### C.1 — Branch selector

23. Si el usuario tiene Agreements tipo Sharing o múltiples timelines propios, aparece un **branch selector** arriba del stream:

```
┌──────────────────────────────────────┐
│  Branches:                           │
│  [🟢 Mi agenda ✓] [🔵 Personal ✓]   │
│  [🟠 Familia 👁] [🟣 Clínica ✓]    │
│                          [+ Agregar] │
└──────────────────────────────────────┘
```

24. Cada branch tiene 3 estados (inspirado en DAW Solo/Mute):
    - **Visible** (✓): eventos incluidos en el stream, mezclados cronológicamente.
    - **Muted** (👁): eventos visibles pero tenues (opacity reducida). No disparan acciones.
    - **Hidden** (—): eventos completamente ocultos del stream.
25. Al tocar una branch → cicla entre Visible → Muted → Hidden.
26. **Solo mode**: doble-tap en una branch → Solo (solo esa branch visible, las demás se ocultan). Doble-tap de nuevo → restaura estado anterior.

#### C.2 — Merge points

27. Cuando dos branches tienen eventos al mismo tiempo, el stream los muestra **apilados** con un conector visual:

```
│  ─── Lun 24 ───────────────────── │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ 🟢 09:00  Corte (María)     │   │  ← mi booking
│  │ ┊                            │   │  ← merge connector
│  │ 🟠 09:00  Acto colegio       │   │  ← timeline familia (conflicto)
│  │    ⚠️ Conflicto detectado    │   │
│  │               [Resolver]     │   │
│  └─────────────────────────────┘   │
```

28. Los conflictos se destacan visualmente con borde rojo y badge de alerta.

#### C.3 — Vista de secretaría (Delegation)

29. Un Delegado con Agreement `ManageSchedule` (ej: secretaria de hospital) puede ver **todos los timelines de los médicos mergeados**:
    - Cada médico es una branch con su color.
    - La secretaria puede confirmar/rechazar bookings de cualquier médico en su branch.
    - Los eventos privados del médico se ven como "Ocupado" (BusyOnly) sin detalles (según VisibilityMode del Agreement).

---

### Capacidad D — Graph View (git-merge)

#### D.1 — Toggle de vista

30. En la toolbar del Timeline hay un toggle: **Stream** (default) ↔ **Graph**.
31. Al cambiar a Graph, el stream se reemplaza por la visualización git-merge.

#### D.2 — Anatomía del grafo

32. El graph view muestra:

```
     Mi agenda    Personal    Familia    Clínica
        │            │           │          │
        ●────────────●           │          │     ← 09:00 Consulta (merge: yo + clínica)
        │            │           │          │
        │            │           ●          │     ← 10:00 Acto colegio
        │            │           │          │
        ●            │           │          │     ← 14:00 Dentista
        │            │           │          │
        │            │           ●──────────●     ← 15:00 Control hijos (merge: familia + clínica)
        │            │           │          │
```

- Eje Y = tiempo (arriba=pasado, abajo=futuro, scrolleable).
- Eje X = branches (cada timeline es una columna/carril).
- **Nodos** (●) = eventos/bookings.
- **Líneas horizontales** = merge points (evento compartido entre branches).
- **Colores** = mismos que en el branch selector.

#### D.3 — Interacción en graph view

33. **Tap en nodo** → popover con detalle del evento + acciones inline.
34. **Pinch to zoom** (mobile) / scroll wheel (desktop) → zoom temporal:
    - Zoom out: vista de semana/mes (nodos se agrupan en clusters).
    - Zoom in: vista de día (nodos expandidos con hora y título).
35. **Scroll vertical** = navegar en el tiempo.
36. **Scroll horizontal** (si muchas branches) = ver más timelines.

#### D.4 — Librería de renderizado

37. Implementación recomendada: **@swimlane/ngx-graph** (Angular native, Dagre layout engine).
    - Nodos customizados via `ng-template` (permite render de avatars, status dots, CTAs).
    - Layout Dagre para posicionamiento automático de branches y merges.
    - Performance: virtualización para >100 nodos visibles.
38. Alternativa: Cytoscape.js (si ngx-graph no soporta el layout requerido).

---

### Capacidad E — Empty states

#### E.1 — Usuario nuevo (sin eventos)

39. Timeline vacío muestra:

```
┌─────────────────────────────────────┐
│                                     │
│         📅 Tu Timeline              │
│                                     │
│    Acá vas a ver tus bookings,      │
│    eventos y compromisos.           │
│                                     │
│    [Explorar servicios]             │  ← navega a Home/feed
│    [Crear un evento privado]        │  ← abre form de evento
│                                     │
└─────────────────────────────────────┘
```

#### E.2 — Día sin eventos

40. Se muestra como una línea mínima colapsada (no se oculta completamente, para que el usuario sepa que el día "existe" y puede agregar eventos).

#### E.3 — Sin branches (sin Agreements)

41. El branch selector no aparece si el usuario solo tiene un timeline.
42. Cuando crea su segundo timeline o recibe un Agreement Sharing → el selector aparece con transición suave.

---

### Capacidad F — Navegación temporal

#### F.1 — Mini-calendar strip

43. Opcional: mini strip de 7 días debajo de la navbar para saltar a un día específico:

```
┌──────────────────────────────────┐
│  L18  M19  [J20]  V21  S22  D23 │  ← hoy resaltado
└──────────────────────────────────┘
```

44. Tap en un día → scroll al date header correspondiente.
45. Swipe horizontal en el strip → cambia semana.

#### F.2 — Scrubber de scroll rápido

46. Al hacer scroll rápido (flick), aparece un **scrubber lateral** (patrón iOS contacts) que muestra la fecha actual como tooltip.
47. El usuario puede arrastrar el scrubber para navegar rápido por meses.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|---|---|
| Timeline con >100 eventos en viewport | Virtualización: solo renderiza eventos visibles ±buffer. Graph view agrupa nodos en clusters. |
| Agreement Sharing con VisibilityMode=BusyOnly | Eventos del timeline compartido se muestran como "Ocupado" sin título ni detalles. Avatar genérico. |
| Agreement revocado mientras se visualiza | Events del timeline revocado desaparecen del stream (eventual, en próximo refresh). Branch se elimina del selector. |
| Timezone diferente entre user y evento | Se muestra hora local del usuario con indicador: "10:00 (11:00 en Buenos Aires)". |
| Evento que cruza medianoche | Se muestra en ambos días con indicador visual (barra que continúa al día siguiente). |
| Booking cancelado mientras se ve en stream | Card se actualiza con status "Cancelado" (strikethrough) + desaparece en el próximo scroll. |
| Conflicto resuelto mientras se ve en stream | Card de conflicto se actualiza a estado resuelto con feedback visual. |
| Scroll muy hacia el futuro (>6 meses) | Se carga bajo demanda; más allá del horizonte de proyección de Slots → solo eventos confirmados. |
| Mobile landscape | Stream se adapta; Graph view aprovecha el ancho extra para más branches visibles. |
| Offline / conexión lenta | Eventos cacheados se muestran inmediatamente; nuevos se cargan cuando hay conexión. Acciones inline se encolan. |

---

## 7. Data Model

No se introducen entidades nuevas. Este flow consume los modelos existentes:

| Entidad | Módulo | Referencia |
|---|---|---|
| `Timeline` | Timeline | Flow 08, sec. 7 |
| `TimelineEvent` | Timeline | Flow 08, sec. 7 |
| `ConflictRule` | Timeline | Flow 08, sec. 7 |
| `ConflictDetection` | Timeline | Flow 08, sec. 7 |
| `Booking` | Booking | Flow 02, sec. 7 |
| `Agreement` | Agreements | `00-core-domain/04-bounded-contexts/01-foundation-governance/3.agreements/` |

### Read model: Timeline Stream Projection

```
TimelineStreamItem {
  id:             UUID
  timelineId:     UUID        -- a cuál branch pertenece
  eventId:        UUID        -- FK → TimelineEvent o Booking-derived Event
  type:           booking_received | booking_made | private_event | shared_event | conflict
  title:          string      -- o "Ocupado" si BusyOnly
  start:          DateTime
  end:            DateTime
  participantId:  UUID?       -- Profile del otro participante
  placeId:        UUID?       -- Place asociado
  status:         confirmed | pending | cancelled | conflict
  branchColor:    string      -- hex color de la branch
  actions:        string[]    -- CTAs disponibles: ["confirm", "reject", "reschedule", "detail"]
  sortKey:        DateTime    -- para ordenamiento cronológico
}
```

---

## 8. Queries

| Query | Módulo | Descripción |
|---|---|---|
| `GetTimelineStream` | `Timeline` | Retorna TimelineStreamItems paginados por rango de fechas. Incluye branches del usuario + Agreements activos. |
| `GetTimelineBranches` | `Timeline` | Lista de timelines visibles (propios + compartidos) con color y VisibilityMode. |
| `GetTimelineGraphData` | `Timeline` | Datos estructurados para graph view: nodos (eventos) + edges (merge points) + branches (carriles). |
| `GetBookingActions` | `Booking` | Acciones disponibles para un booking según su status y el rol del actor. |

---

## 9. Commands (invocados desde acciones inline)

| Command | Módulo | Disparado desde |
|---|---|---|
| `ConfirmBooking` | `Booking` | CTA "Confirmar" en booking pendiente (provider) |
| `RejectBooking` | `Booking` | Swipe left / overflow → "Rechazar" (provider) |
| `ResolveConflict` | `Timeline` | CTA "Resolver" en card de conflicto |
| `AddTimelineEvent` | `Timeline` | FAB / tap en día vacío → form de evento privado |

---

## 10. Events consumed

| Event | Origen | Efecto en Timeline UI |
|---|---|---|
| `BookingCreated` | `Booking` | Nuevo event card aparece en el stream |
| `BookingConfirmed` | `Booking` | Card cambia status de pending a confirmed |
| `BookingCancelled` | `Booking` | Card se marca como cancelada |
| `TimelineEventAdded` | `Timeline` | Nuevo event card de tipo privado |
| `ConflictDetected` | `Timeline` | Card de conflicto aparece en merge point |
| `ConflictResolved` | `Timeline` | Card de conflicto se marca como resuelta |
| `AgreementActivated` | `Agreements` | Nueva branch aparece en el selector |
| `AgreementRevoked` | `Agreements` | Branch desaparece del selector |

---

## 11. Invariants

1. El stream SIEMPRE muestra "Hoy" como punto de referencia (sticky header con highlight).
2. Los días sin eventos se colapsan pero nunca se eliminan completamente del DOM.
3. Las acciones inline solo están disponibles si el actor tiene permisos sobre el booking/evento.
4. Los timelines compartidos con VisibilityMode=BusyOnly NUNCA revelan título ni detalles — solo "Ocupado" + bloque temporal.
5. El graph view y el stream view muestran los mismos datos, solo cambia la representación visual.
6. El branch selector solo aparece cuando hay ≥2 branches visibles.
7. La acción "Confirmar" usa patrón "undo over confirm" (ejecutar + toast con deshacer). La acción "Rechazar" usa modal de confirmación (destructiva).
8. El floating "Today" button solo es visible cuando "Hoy" no está en el viewport.

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo:** `Timeline` (existente, extender con queries de stream/graph).

**Endpoints nuevos:**

| Método | Ruta | Handler | Descripción |
|---|---|---|---|
| `GET` | `/timelines/stream` | `GetTimelineStreamQuery` | Stream items paginados por cursor temporal |
| `GET` | `/timelines/branches` | `GetTimelineBranchesQuery` | Branches disponibles (propios + shared) |
| `GET` | `/timelines/graph` | `GetTimelineGraphDataQuery` | Nodos + edges para graph view |

**Endpoints existentes reutilizados:**

| Método | Ruta | Handler | Módulo |
|---|---|---|---|
| `GET` | `/timelines/events` | `GetTimelineEventsQuery` | `Timeline` |
| `POST` | `/timelines/events` | `AddTimelineEventCommand` | `Timeline` |
| `POST` | `/bookings/{id}/confirm` | `ConfirmBookingCommand` | `Booking` |
| `POST` | `/bookings/{id}/reject` | `RejectBookingCommand` | `Booking` |
| `POST` | `/timelines/conflicts/{id}/resolve` | `ResolveConflictCommand` | `Timeline` |

### Frontend — `vt-bookings-mfe`

**Rutas:**

| Ruta | Componente | Vista |
|---|---|---|
| `/timeline` | `TimelineStreamPage` | Stream view (default) |
| `/timeline/graph` | `TimelineGraphPage` | Graph view |

**Feature structure:**

```
src/app/features/timeline/
├── ui/
│   ├── timeline-stream/          -- stream container + infinite scroll
│   ├── timeline-event-card/      -- individual event card
│   ├── timeline-date-header/     -- sticky date header
│   ├── timeline-branch-selector/ -- branch pills with Solo/Mute/Hide
│   ├── timeline-graph/           -- ngx-graph wrapper
│   ├── timeline-graph-node/      -- custom node template
│   ├── timeline-empty-state/     -- empty state component
│   ├── timeline-today-button/    -- floating "Today" FAB
│   └── timeline-mini-calendar/   -- 7-day strip navigation
├── data-access/
│   ├── timeline-stream.service.ts
│   ├── timeline-branch.service.ts
│   └── timeline-graph.service.ts
└── feature/
    ├── timeline-stream.page.ts
    └── timeline-graph.page.ts
```

**Toolkit components necesarios (ver BL-122):**

| Componente toolkit | Descripción |
|---|---|
| `vt-timeline-stream` | Container scrolleable con infinite scroll bidireccional + virtualización |
| `vt-date-header` | Sticky header con fecha, indicador "Hoy", colapsable |
| `vt-timeline-event` | Card de evento con avatar, hora, duración, color, CTA inline, swipe actions |
| `vt-timeline-empty` | Empty state para día sin eventos o timeline nuevo |

**UI States:**

`loading` → `empty` → `stream-active` → `graph-active`

**Interacciones mobile:**
- Vertical scroll = navegación temporal
- Swipe right en event card = acción primaria (confirmar)
- Swipe left en event card = menú acciones secundarias
- Long press en event card = context menu
- Tap en floating "Today" = scroll to today
- Doble-tap en branch pill = Solo mode

**Interacciones desktop:**
- Vertical scroll = navegación temporal
- Hover en event card = reveal action toolbar
- Click en event card = expand detail inline
- Keyboard: ↑↓ navega entre eventos, Enter = acción primaria, Esc = close detail

---

## 13. Acceptance Criteria

```gherkin
Scenario: Stream view - carga inicial
  Given usuario autenticado con bookings y eventos
  When abre /timeline
  Then ve stream vertical con eventos ordenados cronológicamente
  And sticky date header de "Hoy" visible con highlight
  And días sin eventos están colapsados

Scenario: Scroll infinito bidireccional
  Given usuario en stream view
  When hace scroll hacia abajo
  Then se cargan eventos futuros automáticamente
  When hace scroll hacia arriba
  Then se cargan eventos pasados automáticamente

Scenario: Floating Today button
  Given usuario ha scrolleado lejos de la fecha actual
  Then aparece botón flotante "Hoy"
  When toca el botón
  Then smooth scroll hasta la fecha actual
  And el botón desaparece

Scenario: Confirmar booking inline (provider)
  Given usuario es provider con booking en status Pending
  When toca "Confirmar" en la card del booking
  Then booking cambia a Confirmed inmediatamente
  And toast aparece con botón "Deshacer" por 5 segundos
  And notificación enviada al cliente

Scenario: Rechazar booking (acción destructiva)
  Given usuario es provider con booking en status Pending
  When swipe left → "Rechazar"
  Then modal de confirmación aparece
  When confirma en el modal
  Then booking cambia a Rejected

Scenario: Multi-branch selector
  Given usuario tiene ≥2 timelines visibles (propios + shared)
  Then branch selector aparece arriba del stream
  And cada branch tiene nombre + color
  When toca una branch
  Then cicla entre Visible → Muted → Hidden
  And el stream se actualiza filtrando/mostrando eventos de esa branch

Scenario: Solo mode
  Given usuario tiene 3+ branches visibles
  When doble-tap en una branch
  Then solo esa branch queda visible, las demás se ocultan
  When doble-tap de nuevo
  Then se restaura el estado anterior de todas las branches

Scenario: Graph view
  Given usuario tiene eventos en múltiples branches
  When toggle a Graph view
  Then ve grafo tipo git-merge con branches como columnas
  And nodos son eventos posicionados por fecha
  And merge points conectan eventos compartidos

Scenario: Tap en nodo del grafo
  Given usuario en graph view
  When toca un nodo (evento)
  Then popover aparece con detalle + acciones inline

Scenario: Timeline compartido con BusyOnly
  Given usuario tiene Agreement Sharing con VisibilityMode=BusyOnly
  When visualiza eventos del timeline compartido
  Then eventos aparecen como "Ocupado" sin título ni detalles
  And avatar genérico en lugar de foto del participante

Scenario: Empty state - usuario nuevo
  Given usuario sin eventos ni bookings
  When abre /timeline
  Then ve empty state con CTAs "Explorar servicios" y "Crear evento"

Scenario: Delegado con ManageSchedule
  Given secretaria con Agreement ManageSchedule sobre 3 médicos
  When abre /timeline
  Then ve branches de los 3 médicos en el selector
  And puede confirmar/rechazar bookings de cualquier médico
  And eventos privados de médicos se ven como "Ocupado"

Scenario: Día colapsado sin eventos
  Given un día sin eventos entre dos días con eventos
  Then el día se muestra colapsado como línea mínima
  When toca el día colapsado
  Then se expande mostrando franja vacía
```

---

## 14. NEEDS-CLARIFICATION

| ID | Pregunta | Impacto |
|---|---|---|
| NC-01 | **Graph view prioridad:** ¿se implementa en v1 o se difiere a v2? Graph view requiere ngx-graph y layout custom. Stream view es más simple y cubre el 90% de los casos. | Define scope de Sprint 12 |
| NC-02 | **Undo window para confirmar:** ¿5 segundos es suficiente? ¿Qué pasa si el toast desaparece y el usuario quiere deshacer? | UX de undo |
| NC-03 | **Offline actions queue:** ¿se implementa queue de acciones offline en v1, o se requiere conexión activa para acciones inline? | Complejidad de implementación |
| NC-04 | **Mini-calendar strip:** ¿se incluye en v1 o es suficiente con el floating Today button + scroll rápido? | Scope de navegación temporal |
| NC-05 | **Branch colors:** ¿el usuario elige el color de cada branch, o se asignan automáticamente? | UI customization |
| NC-06 | **Horizonte de carga:** ¿30 días es el default correcto? Usuarios con muchos bookings podrían necesitar más; usuarios nuevos podrían necesitar menos. | Performance vs UX |
| NC-07 | **Keyboard navigation en desktop:** ¿se implementan shortcuts en v1? (↑↓ entre eventos, Enter para acción, etc.) | Accessibility + power users |

---

## 15. Research References

Las decisiones de UX en este flow se basan en investigación de:

- **Stream view:** Fantastical (collapse empty days), Sunrise Calendar (avatar-first layout), Amie (natural language headers, progressive disclosure).
- **Git-graph visualization:** @swimlane/ngx-graph (Angular native, Dagre layout), Cytoscape.js (performance alternative).
- **Multi-timeline merge:** DAW Solo/Mute/Hide pattern (Logic Pro, Ableton), Teamup Calendar (swimlanes), Google Calendar (color overlay).
- **Inline actions:** Gmail (undo over confirm), Slack (hover toolbar), Linear (primary CTA always visible), Instagram (always-visible actions).
- **Mobile gestures:** iOS scroll patterns, Fantastical scrubber, conditional floating button, swipe-to-act (Mail.app pattern).
- **Navigation decision:** ADR-0005 (3 tabs + avatar profile).

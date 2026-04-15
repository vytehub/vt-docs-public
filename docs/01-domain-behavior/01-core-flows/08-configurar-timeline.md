---
title: Flow 08 — Configurar Timeline
description: >
  Un usuario (o Delegado con ManageSchedule) configura su Timeline: agrega eventos privados,
  gestiona ConflictRules cross-timeline y resuelve los conflictos detectados con Slots o
  Bookings existentes. El módulo Timeline (sucesor de Supply) proyecta los Slots desde los
  Listings publicados y ejecuta las ConflictRules.
status: draft
version: 1
---

# Flow 08 — Configurar Timeline

## 1. Resumen
- **Goal:** que un usuario pueda gestionar su Timeline: agregar eventos privados, configurar
  reglas de conflicto con timelines externos (ConflictRules) y resolver los conflictos
  detectados contra Slots o Bookings existentes.
- **Actores:**
  - **Primary:** cualquier usuario con un Timeline propio.
  - **Secondary:** Delegado con Agreement activo que incluya permiso `ManageSchedule`.
- **Surfaces:** `vt-timeline-mfe` — rutas `/timeline`, `/timeline/events/*`, `/timeline/rules/*`.

---

## 2. Domain Context

### Timeline como configuración de agenda (ADR-0006)
El Timeline **no es un contenedor de eventos**. Es la configuración que define:
1. **De quién es** la agenda (owner, type, privacy).
2. **Quién puede ver/gestionar** (members).
3. **Qué ConflictRules** aplican (cross-timeline).

Los **Events** son entidades independientes que se linkean a Timelines via `EventTimelineLink` (N:M).
Un evento puede ser visible en múltiples timelines simultáneamente.

### Distribución de responsabilidades (ADR-0006)

| Concepto | Módulo | Descripción |
|----------|--------|-------------|
| Timeline (config) | **Timelines** | Privacy, members, conflict rules |
| Event (tiempo ocupado) | **Events** | Entidad independiente, linked a N timelines |
| ConflictRule (qué observar) | **Timelines** | Define qué timelines observar |
| ConflictDetection (solapamientos) | **Events** | Detecta y registra overlaps entre eventos |
| Slot Projection | **Events** | Calcula disponibilidad desde Listing rules - Events ocupados |

### Excepciones de disponibilidad
Las fechas/franjas donde **no se proyecta un Slot** se definen en el **Listing** (Flow 06),
no en el Timeline directamente.

### ConflictRules — cross-timeline only
Los conflictos **dentro del mismo Timeline** se resuelven mediante **shared capacity** (cupos).
Las ConflictRules son exclusivamente **cross-timeline**: el usuario declara que los eventos
de otro Timeline son relevantes para el suyo.

Ejemplos:
- Un acto escolar en el "Timeline familiar" genera conflicto en la agenda laboral del usuario.
- La no-disponibilidad de la sala de reuniones (Timeline del recurso) bloquea la agenda personal.

Action en v1: solo `NotifyOnly` — notificación al usuario; sin cancelación ni ocultamiento automático.
`HideSlots` queda reservado para v2.

---

## 3. Preconditions
- Usuario tiene cuenta activa en VyteMerge.
- El Timeline por defecto fue creado automáticamente al registrarse (Flow 00).
- Si el actor es un Delegado: tiene Agreement activo con permiso `ManageSchedule` sobre
  el Timeline del Owner.

---

## 4. Trigger
- Usuario navega a `/timeline` en `vt-timeline-mfe`.
- Delegado con `ManageSchedule` navega al Timeline del Owner desde su panel de Agreements.

---

## 5. Main Flow

### Capacidad A — Vista calendario

1. Usuario abre `/timeline`.
2. Sistema renderiza el calendario con:
   - **Slots disponibles** (proyectados desde Listings publicados).
   - **Bookings existentes** (confirmados o pendientes).
   - **Eventos privados** del usuario (solo visibles para él / su Delegado).
   - **Conflictos detectados** (destacados visualmente).
3. Usuario puede navegar por semanas/meses.

### Capacidad B — Agregar evento privado

4. Usuario pulsa **"Nuevo evento"** (o clic en una franja horaria del calendario).
5. Sistema muestra formulario: título (opcional), fecha/hora inicio y fin, notas (opcional).
6. Usuario guarda el evento.
7. Sistema persiste el `Event` (entidad independiente, linked al Timeline del usuario) y evalúa si genera conflicto con Slots o Bookings existentes.
8. **Si hay conflicto:**
   - Sistema muestra modal de resolución:
     **Conservar todo** / **Cancelar Slot/Booking** / **Reagendar Slot/Booking**.
   - Usuario elige.
   - Sistema aplica la decisión: cancela el Booking afectado o lo marca para reagendamiento.
9. **Si no hay conflicto:** evento guardado; Calendar View se actualiza.

### Capacidad C — Configurar ConflictRules

10. Usuario navega a `/timeline/rules`.
11. Sistema lista las ConflictRules configuradas (puede ser vacía).
12. Usuario pulsa **"Nueva regla"** → `/timeline/rules/new`.
13. Sistema muestra formulario con picker de Source Timeline (Timelines accesibles al usuario).
14. Usuario selecciona el Source Timeline y guarda.
15. Sistema persiste la `ConflictRule` y re-evalúa los eventos existentes del Source Timeline
    contra los Slots/Bookings del Timeline del usuario.
16. Si se detectan conflictos → genera notificaciones in-app al usuario.

### Capacidad D — Resolver conflicto desde notificación

17. Usuario recibe notificación in-app: "Conflicto detectado en tu Timeline".
18. Usuario abre el detalle del conflicto.
19. Sistema muestra: qué Event del Source Timeline genera el conflicto y con qué Slot/Booking.
20. Usuario elige: **Conservar todo** / **Cancelar Slot/Booking** / **Reagendar Slot/Booking**.
21. Sistema aplica la decisión y marca el conflicto como resuelto.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Timeline sin ConflictRules | Válido; el sistema acepta Bookings sin restricciones externas |
| Evento privado sin conflicto | Se guarda directamente; no se dispara modal de resolución |
| ConflictRule agregada con eventos históricos en Source | Sistema re-evalúa; notifica conflictos encontrados |
| ConflictRule eliminada | Notificaciones pendientes de esa regla se limpian; resoluciones anteriores no se revierten |
| Source Timeline eliminado | ConflictRule se elimina automáticamente; conflictos pendientes de esa regla se cierran |
| Delegado con ManageSchedule | Puede agregar eventos privados y gestionar ConflictRules; privacidad del Owner aplicable (ver NEEDS-CLARIFICATION) |
| Intento de duplicar ConflictRule | Sistema rechaza: ya existe una regla con ese Source Timeline |
| Evento privado editado para solapar con Booking | Sistema re-evalúa y muestra modal de resolución si hay nuevo conflicto |
| Timeline sin Listings publicados | Vista calendario vacía; estado vacío con CTA "Crear Listing" |
| Usuario elige "Reagendar" en modal | Se abre selector de nueva fecha/hora para el Slot/Booking afectado (NEEDS-CLARIFICATION: flujo exacto) |

---

## 7. Data Model (v1 minimal)

```
Timeline {                          -- módulo Timelines (configuración)
  id:          UUID
  profileId:   UUID                 -- FK Profile
  name:        string
  privacy:     public | followersOnly | private
  isDefault:   bool
  createdAt:   DateTime
}

ConflictRule {                      -- módulo Timelines (configuración)
  id:                UUID
  timelineId:        UUID           -- el Timeline que aplica esta regla
  sourceTimelineId:  UUID           -- el Timeline externo a observar (≠ timelineId)
  action:            NotifyOnly     -- v1; HideSlots reservado para v2
  createdAt:         DateTime
}

Event {                             -- módulo Events (entidad independiente — ADR-0006)
  id:          UUID
  title:       string?
  start:       DateTime
  end:         DateTime
  type:        manual | booking | block | external
  sourceId:    UUID?                -- bookingId si type=booking
  createdBy:   UUID                 -- profileId del creador
  status:      active | cancelled
  notes:       string?
  visibility:  public | busyOnly | private
  placeId:     UUID?
  createdAt:   DateTime
  updatedAt:   DateTime
}

EventTimelineLink {                 -- módulo Events (relación N:M — ADR-0006)
  eventId:     UUID                 -- FK Event
  timelineId:  UUID                 -- FK Timeline
  role:        primary | viewer | readonly
  linkedAt:    DateTime
}

ConflictDetection {                 -- módulo Events (detección de solapamientos)
  id:                   UUID
  conflictRuleId:        UUID
  sourceEventId:         UUID       -- Event del Source que genera el conflicto
  targetEventId:         UUID       -- Event afectado en este Timeline
  status:               Pending | Resolved | Dismissed
  resolution:           KeepAll | CancelTarget | RescheduleTarget | null
  detectedAt:           DateTime
  resolvedAt:           DateTime?
}
```

---

## 8. Commands

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `AddEvent` | `Events` | Usuario autenticado; Timeline propio o Delegado con ManageSchedule. Crea Event + EventTimelineLink. |
| `UpdateEvent` | `Events` | Event existe; solo type=manual/block |
| `RemoveEvent` | `Events` | Event existe; solo type=manual/block |
| `AddConflictRule` | `Timelines` | Source Timeline accesible; no existe regla duplicada; sourceId ≠ timelineId |
| `RemoveConflictRule` | `Timelines` | ConflictRule existe en este Timeline |
| `ResolveConflict` | `Events` | ConflictDetection en status Pending |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `EventCreated` | `AddEvent` | Conflict detection: evalúa solapamientos con eventos de otros timelines |
| `EventUpdated` | `UpdateEvent` | Re-evalúa conflictos del evento modificado |
| `EventCancelled` | `RemoveEvent` | Limpia ConflictDetections asociadas |
| `ConflictRuleAdded` | `AddConflictRule` | Re-evalúa eventos históricos del Source Timeline |
| `ConflictRuleRemoved` | `RemoveConflictRule` | Limpia notificaciones pendientes de esa regla |
| `ConflictDetected` | (sistema, post-evaluación) | Notification in-app al usuario |
| `ConflictResolved` | `ResolveConflict` | Si resolution=CancelTarget: dispara cancelación de Booking en Booking module |

---

## 10. Invariants

1. Un Timeline pertenece a exactamente un Profile.
2. No pueden existir dos ConflictRules con el mismo `timelineId` + `sourceTimelineId`.
3. El `sourceTimelineId` de una ConflictRule no puede ser igual al `timelineId` (no auto-referencial).
4. Una ConflictRule solo genera notificaciones (v1); no cancela ni oculta Slots/Bookings automáticamente.
5. Los Slots son proyecciones del módulo Timeline desde Listings publicados; no se crean directamente.
6. Los conflictos intra-timeline (mismo Timeline) se resuelven mediante shared capacity, no con ConflictRules.
7. Un Timeline sin ConflictRules es válido y acepta Bookings sin restricciones externas.
8. Resolver un conflicto no revierte resoluciones anteriores ni afecta otras ConflictRules.

---

## 11. Outputs

- `Event` (type=manual) persistido y linked al Timeline del usuario, visible en su calendario.
- `ConflictRule` configurada; conflictos cross-timeline detectados y notificados al usuario.
- Decisiones de resolución aplicadas (cancelaciones propagadas al módulo Booking).
- Vista calendario actualizada con estado coherente (Slots, Bookings, eventos privados, conflictos resueltos/pendientes).

---

## 12. Technical Mapping (Draft)

### Backend

**Dos módulos backend involucrados (ADR-0006):**

**Módulo `Timelines`** — configuración de agenda
- Posee el aggregate `Timeline` y `ConflictRule`.
- **DbContext:** `TimelinesDbContext` (schema: `timelines`)

**Módulo `Events`** — lo que ocupa tiempo
- Posee `Event`, `EventTimelineLink`, `ConflictDetection`.
- Responsable de Slot Projection.
- **DbContext:** `EventsDbContext` (schema: `events`)

**Endpoints (distribuidos entre módulos):**
```
-- Timelines module --
GET    /timelines                                → lista timelines del autenticado
POST   /timelines                                → CreateTimeline
PATCH  /timelines/{id}/privacy                   → SetTimelinePrivacy
GET    /timelines/{id}/conflict-rules             → lista ConflictRules
POST   /timelines/{id}/conflict-rules             → AddConflictRule
DELETE /timelines/conflict-rules/{id}             → RemoveConflictRule

-- Events module --
GET    /timelines/{id}/events                    → Events linked a este timeline
POST   /timelines/{id}/events                    → AddEvent (manual/block)
PATCH  /timelines/{id}/events/{eventId}          → UpdateEvent
DELETE /timelines/{id}/events/{eventId}          → RemoveEvent
GET    /timelines/{id}/conflicts                 → conflictos pendientes
POST   /timelines/conflicts/{id}/resolve         → ResolveConflict
GET    /availability?listingId=X&from=Y&to=Z     → Slot Projection
```

**Integración Booking:** `ConflictResolved` (resolution=CancelTarget) → Booking module cancela el Booking afectado.

### Frontend

**MFE:** `vt-timeline-mfe` (nuevo)

```
/timeline                    → calendario (vista semanal/mensual)
/timeline/events/new         → formulario nuevo evento privado
/timeline/events/:id         → detalle / editar evento privado
/timeline/rules              → panel de ConflictRules
/timeline/rules/new          → crear ConflictRule (picker de Source Timeline)
```

**Toolkit components (candidatos):** NEEDS-CLARIFICATION — pendiente inventario `vt-toolkit`:
- Calendar component (semanal/mensual, multi-tipo de eventos)
- Conflict highlight / badge de conflicto
- Resolution modal (Conservar / Cancelar / Reagendar)
- Timeline picker (selector de Source en ConflictRule)
- Event form (título, datetime range, notas)
- Notification badge (conflictos pendientes)

**UI States:**
- **`/timeline`:** estado vacío (sin Listings publicados) + CTA "Crear Listing"; calendario con Slots, Bookings y eventos privados; conflictos destacados visualmente con CTA de resolución.
- **Modal de resolución:** aparece al guardar evento privado con conflicto; opciones Conservar / Cancelar / Reagendar.
- **`/timeline/rules`:** estado vacío + CTA "Nueva regla"; lista de reglas con Source Timeline y estado de conflictos.
- **`/timeline/rules/new`:** picker de Timeline accesibles + preview de conflictos que generaría la nueva regla.

---

## 13. Acceptance Criteria

- [ ] Usuario puede ver su calendario con Slots, Bookings y eventos privados.
- [ ] Usuario puede agregar un evento privado con título (opcional), fecha/hora inicio y fin.
- [ ] Al guardar un evento privado que solapa con un Booking, el sistema muestra modal de resolución.
- [ ] Usuario puede elegir Conservar, Cancelar o Reagendar el Booking afectado.
- [ ] Si elige Cancelar, el Booking es cancelado por el módulo Booking.
- [ ] Usuario puede agregar una ConflictRule seleccionando un Source Timeline.
- [ ] No se pueden crear dos ConflictRules con el mismo Source Timeline.
- [ ] Source Timeline no puede ser el mismo Timeline (no auto-referencial).
- [ ] Al agregar una ConflictRule, el sistema re-evalúa eventos históricos del Source y notifica conflictos.
- [ ] Usuario recibe notificación in-app al detectarse un nuevo conflicto.
- [ ] Usuario puede resolver un conflicto pendiente eligiendo Conservar, Cancelar o Reagendar.
- [ ] Al eliminar una ConflictRule, las notificaciones pendientes de esa regla desaparecen.
- [ ] Delegado con `ManageSchedule` puede agregar eventos privados y gestionar ConflictRules.
- [ ] Timeline sin ConflictRules es válido y no bloquea ningún Booking.
- [ ] Architecture tests del módulo Timeline pasan sin errores.

---

## 14. NEEDS-CLARIFICATION

- **Migración Supply → Timeline:** ¿el módulo `Supply` se renombra a `Timeline`, o se crea un módulo nuevo y Supply se depreca gradualmente? Requiere ADR formal (ADR-0005 pendiente).
- **Source Timeline picker:** ¿el usuario ve todos los Timelines públicos, o solo los compartidos con él vía Agreement (type=Sharing)?
- **Slot projection horizon:** `ListingPublished` dispara proyección. ¿Cuál es el horizonte de proyección (ej: 60 días)? ¿Y la frecuencia de re-proyección?
- **Modal "Reagendar":** ¿abre un selector de nueva fecha/hora in-situ, o redirige a otro flujo de reagendamiento (vt-booking-mfe)?
- **Delegado y privacidad:** ¿un Delegado con `ManageSchedule` puede ver eventos privados (`isPrivate=true`) del Owner, o solo los no-privados?
- **Toolkit calendar:** ¿existe un componente de calendario en `vt-toolkit`, o hay que construirlo desde cero para este MFE?
- **ConflictRule y HideSlots (v2):** al implementar HideSlots, ¿se ocultan los Slots al público general, o también al Owner en su propia vista?

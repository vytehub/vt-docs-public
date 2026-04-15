---
title: Flow 10 — Gestionar Bookings (Proveedor)
description: >
  El proveedor (o Delegado con ManageBookings) gestiona los Bookings de su agenda:
  visualiza el dashboard de bookings, confirma solicitudes pendientes, completa
  servicios realizados, marca no-shows y cancela reservas. La cancelación de un
  Booking Confirmed por el proveedor emite reembolso al cliente (v1 simplificado;
  integración completa con módulo Policies pendiente).
status: draft
version: 1
---

# Flow 10 — Gestionar Bookings (Proveedor)

## 1. Resumen
- **Goal:** que el proveedor pueda operar su agenda de Bookings: ver el estado de todas
  sus reservas, confirmar solicitudes pendientes (ManualConfirm), registrar la finalización
  del servicio, marcar no-shows y cancelar si es necesario.
- **Actores:**
  - **Primary:** cualquier usuario con Bookings activos como proveedor.
  - **Secondary:** Delegado con Agreement activo que incluya `ManageBookings`.
- **Surfaces:** `vt-timeline-mfe` — rutas `/timeline/bookings` y `/timeline/bookings/:id`.
  La vista calendario (`/timeline`) muestra Bookings visualmente; estas rutas proveen
  la vista de lista + detalle con acciones.

---

## 2. Domain Context

### Rol del proveedor en el Booking
El proveedor es el dueño del Listing sobre el que se creó el Booking. Tiene autoridad para:
- **Confirmar** solicitudes `Pending` (cuando `confirmationPolicy = ManualConfirm`)
- **Completar** Bookings `Confirmed` una vez prestado el servicio
- **Marcar NoShow** si el cliente no se presentó
- **Cancelar** Bookings en cualquier estado no terminal

El cliente (attendee) realiza la reserva (Flow 02) y puede cancelar desde su propia vista;
ese caso no se cubre aquí.

### Relación con Policies y Commerce (v1 simplificado)
El módulo `Policies` (Flow 11 y BC 07) no está implementado en v1. Para este flow:

| Acción | v1 (simplificado) | Post-Policies |
|--------|-------------------|---------------|
| Proveedor cancela `Pending` | Sin penalidad; soft-hold liberado | Igual |
| Proveedor cancela `Confirmed` | 100% reembolso al cliente | Según `CancellationPolicy` del Listing |
| Cliente cancela `Confirmed` | 100% reembolso (v1) | Según ventana + policy |
| NoShow del cliente | Penalidad fija según config del Listing (v1 simple) | Según `NoShowPolicy` |

### Relación con Timeline
Los Bookings `Confirmed` generan un Event independiente linked a los timelines del proveedor y del cliente (ADR-0006).
Al cancelar o completar, el Event se actualiza y todas las vistas de timeline reflejan el cambio automáticamente.

---

## 3. Preconditions
- El proveedor tiene Listings Published con Bookings activos.
- Si el actor es un Delegado: tiene Agreement activo con `ManageBookings` sobre el Profile del Owner.

---

## 4. Trigger
- Proveedor navega a `/timeline/bookings` en `vt-timeline-mfe`.
- Proveedor recibe notificación in-app "Nueva solicitud de reserva" → accede al detalle.
- Delegado con `ManageBookings` accede al panel de Bookings del Owner.

---

## 5. Main Flow

### Capacidad A — Dashboard de Bookings

1. Proveedor navega a `/timeline/bookings`.
2. Sistema muestra lista de Bookings ordenados por fecha, con filtros:
   - Por **estado**: Pending | Confirmed | Completed | Cancelled | NoShow
   - Por **fecha**: hoy / esta semana / rango personalizado
   - Por **cliente**: búsqueda por nombre o email
3. Proveedor puede acceder al detalle de cualquier Booking.

### Capacidad B — Confirmar Booking (ManualConfirm)

4. Proveedor abre un Booking en estado `Pending`.
5. Sistema muestra: Listing, slot solicitado, datos del cliente, intake form completado.
6. Proveedor revisa y pulsa **"Confirmar"**.
7. Sistema muestra campo opcional de mensaje al cliente.
8. Proveedor confirma (con o sin mensaje).
9. Sistema:
   - Transiciona el Booking a `Confirmed`.
   - Crea un Event linked a los timelines del proveedor y del cliente (bloquea disponibilidad).
   - Envía notificación in-app + email al cliente con el mensaje incluido (si existe).
   - Actualiza la proyección de Slots del Listing.

### Capacidad C — Completar Booking

10. Proveedor abre un Booking en estado `Confirmed`.
11. Proveedor pulsa **"Marcar como completado"**.
12. Sistema transiciona el Booking a `Completed`.
13. Sistema:
    - Emite `BookingCompleted`.
    - Envía notificación in-app + email al cliente.
    - Si aplica: ejecuta incentivos anti-cancelación (devolución por compromiso).

### Capacidad D — Marcar NoShow

14. Proveedor abre un Booking `Confirmed` cuyo slot ya pasó.
15. Proveedor pulsa **"Marcar no-show"**.
16. Sistema transiciona a `NoShow`.
17. Sistema:
    - Emite `BookingNoShow`.
    - Commerce aplica penalidad al cliente automáticamente (según config del Listing).
    - Envía notificación in-app + email al cliente informando el no-show.

### Capacidad E — Cancelar Booking (proveedor)

18. Proveedor abre un Booking `Pending` o `Confirmed`.
19. Proveedor pulsa **"Cancelar"**.
20. Sistema muestra confirmación con las consecuencias:
    - Si `Pending`: sin penalidad; soft-hold liberado.
    - Si `Confirmed` (v1 simplificado): 100% reembolso al cliente.
21. Proveedor confirma la cancelación.
22. Sistema:
    - Transiciona a `Cancelled`.
    - Emite `BookingCancelled`.
    - Si `Confirmed`: Commerce emite reembolso (100% en v1).
    - Cancela el Event (todas las vistas de timeline linked reflejan el cambio).
    - Envía notificación in-app + email al cliente con motivo.
    - Reprojcta Slots del Listing.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Proveedor rechaza un Booking `Pending` | Equivale a cancelar (`Pending → Cancelled`); sin penalidad; cliente notificado |
| Booking `Confirmed` — proveedor quiere reagendar | Flujo de Reagendar (Flow 11); no es cancelación |
| Booking `Completed` — proveedor intenta cancelar | Sistema rechaza: estado terminal |
| Booking `Pending` sin `confirmationPolicy=ManualConfirm` | No aparece en cola de confirmación pendiente (fue auto-confirmado en Flow 02) |
| Delegado confirma en nombre del Owner | Permitido con `ManageBookings`; auditoría registra al Delegado como actor |
| Múltiples Bookings `Pending` simultáneos | No hay acción masiva en v1; se confirman uno a uno |
| Commerce falla al procesar reembolso | Booking queda en `Cancelled`; reembolso queda en estado `Pending` en Commerce; se reintenta |
| Slot reservado por Booking `Confirmed` liberado al cancelar | Timeline re-proyecta disponibilidad; slot vuelve a aparecer |
| NoShow sin configuración de penalidad en Listing | Sistema registra el NoShow sin cargo; notifica igualmente al cliente |

---

## 7. Data Model (v1 — cambios sobre Booking existente)

```
-- Adición al aggregate Booking:
Booking {
  ...                              -- campos existentes (Flow 02)
  confirmationMessage: string?     -- mensaje del proveedor al confirmar (opcional)
  cancelledBy:        Provider | Client | System
  cancellationReason: string?
  noShowRecordedAt:   DateTime?
  completedAt:        DateTime?
}
```

No se agregan nuevas entidades; se extiende el aggregate `Booking`.

---

## 8. Commands

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `ConfirmBooking` | `Booking` | Status = Pending; caller = Owner o Delegado con ManageBookings |
| `CompleteBooking` | `Booking` | Status = Confirmed; caller = Owner o Delegado |
| `MarkNoShow` | `Booking` | Status = Confirmed; caller = Owner o Delegado |
| `CancelBooking` (proveedor) | `Booking` | Status = Pending \| Confirmed; caller = Owner o Delegado |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `BookingConfirmed` | `ConfirmBooking` | Events: crea Event + links a timelines; Notification: in-app + email al cliente |
| `BookingCompleted` | `CompleteBooking` | Commerce: incentivos anti-cancelación si aplica; Notification: in-app + email al cliente |
| `BookingNoShow` | `MarkNoShow` | Commerce: aplica penalidad al cliente; Notification: in-app + email al cliente |
| `BookingCancelled` | `CancelBooking` | Events: cancela Event (refleja en todos los timelines linked); Commerce: reembolso si Confirmed (100% v1); Notification: in-app + email al cliente |

---

## 10. Invariants

1. Solo el Owner o Delegado con `ManageBookings` puede confirmar, completar, marcar NoShow o cancelar.
2. Solo un Booking `Pending` puede confirmarse.
3. Solo un Booking `Confirmed` puede completarse o marcarse NoShow.
4. `Cancelled`, `Completed` y `NoShow` son estados terminales; no pueden revertirse.
5. Al cancelar un Booking `Confirmed`, siempre se emite reembolso (100% en v1; según Policy post-v1).
6. Al cancelar un Booking `Pending`, no se aplica ninguna penalidad ni reembolso.
7. Un Booking completado o marcado NoShow conserva su Event como registro histórico (visible en los timelines linked).
8. La auditoría registra quién ejecutó cada acción (Owner o Delegado) y cuándo.

---

## 11. Outputs

- Booking en estado `Confirmed`, `Completed`, `NoShow` o `Cancelled` según la acción.
- Timeline del proveedor actualizado (Event creado, cancelado o cerrado).
- Reembolso emitido (si cancelación de `Confirmed`).
- Penalidad aplicada (si NoShow con policy configurada).
- Notificaciones in-app + email enviadas al cliente.

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo:** `Bookings` (extiende el módulo existente con nuevos commands del proveedor)

**Nuevos commands a agregar:**
```
src/Modules/Bookings/
└── Bookings.Application/
    └── Commands/
        ├── ConfirmBooking/
        ├── CompleteBooking/
        ├── MarkNoShow/
        └── CancelBooking/    (provider-initiated)
```

**Nuevos endpoints:**
```
GET    /bookings                              → lista de Bookings del proveedor autenticado
GET    /bookings/:id                          → detalle del Booking
POST   /bookings/:id/confirm                  → ConfirmBooking
POST   /bookings/:id/complete                 → CompleteBooking
POST   /bookings/:id/no-show                  → MarkNoShow
POST   /bookings/:id/cancel                   → CancelBooking
```

**Query params para GET /bookings:**
```
?role=provider                  → bookings donde el autenticado es proveedor
&status=pending|confirmed|...   → filtro de estado
&from=YYYY-MM-DD                → desde fecha
&to=YYYY-MM-DD                  → hasta fecha
&q=nombre_cliente               → búsqueda por cliente
```

**Integración Commerce (v1 simplificado):**
- `BookingCancelled` (Confirmed, cancelledBy=Provider) → `Commerce.IssueRefund(100%)`
- `BookingNoShow` → `Commerce.ApplyNoShowPenalty(listingNoShowConfig)`

**Integración Timeline:**
- `BookingConfirmed` → `Timeline.CreateEvent(bookingId)`
- `BookingCancelled` | `BookingCompleted` | `BookingNoShow` → `Timeline.CloseEvent(bookingId)`

### Frontend

**MFE:** `vt-timeline-mfe` (rutas nuevas dentro del MFE existente)

```
/timeline/bookings              → lista de Bookings (filtros: estado, fecha, cliente)
/timeline/bookings/:id          → detalle del Booking + acciones contextuales
```

**Toolkit components (candidatos):** NEEDS-CLARIFICATION — pendiente inventario `vt-toolkit`:
- Booking list item (estado badge, cliente, Listing, slot, CTA contextual)
- Booking detail card
- Confirm modal (con campo de mensaje opcional)
- Cancel confirm modal (con consecuencias visibles: reembolso / sin penalidad)
- NoShow confirm modal (con penalidad aplicable)
- Status filter chips (Pending / Confirmed / Completed / Cancelled / NoShow)
- Date range picker

**UI States:**
- **`/timeline/bookings`:** estado vacío (sin Bookings); lista con filtros; badge de Pending count (acción requerida)
- **`/timeline/bookings/:id` — Pending:** CTA "Confirmar" + "Rechazar"; detalle del cliente + intake form
- **`/timeline/bookings/:id` — Confirmed:** CTA "Completar" + "Marcar No-Show" + "Cancelar"
- **`/timeline/bookings/:id` — terminal:** read-only con historial de acciones

---

## 13. Acceptance Criteria

- [ ] Proveedor puede ver la lista de sus Bookings filtrada por estado y fecha.
- [ ] Un Booking `Pending` muestra CTA "Confirmar" y "Rechazar".
- [ ] Al confirmar, el proveedor puede incluir un mensaje opcional al cliente.
- [ ] Al confirmar, el cliente recibe notificación in-app + email con el mensaje.
- [ ] Al confirmar, el Event se crea en el Timeline del proveedor.
- [ ] Un Booking `Confirmed` muestra CTA "Completar", "Marcar No-Show" y "Cancelar".
- [ ] Al completar, el Booking pasa a `Completed` y el cliente es notificado.
- [ ] Al marcar NoShow, Commerce aplica la penalidad configurada en el Listing y el cliente es notificado.
- [ ] Al cancelar un Booking `Pending`, no se genera penalidad ni reembolso; cliente notificado.
- [ ] Al cancelar un Booking `Confirmed`, Commerce emite reembolso del 100% al cliente.
- [ ] `Cancelled`, `Completed` y `NoShow` son terminales: no aparecen CTAs de acción.
- [ ] El historial de acciones (quién, cuándo, qué) es visible en el detalle del Booking.
- [ ] Delegado con `ManageBookings` puede ejecutar todas las acciones anteriores.
- [ ] Un usuario sin `ManageBookings` recibe 403 al intentar operar Bookings del Owner.

---

## 14. NEEDS-CLARIFICATION

- **Integración Policies (post-v1):** cuando el módulo `Policies` esté disponible, `CancelBooking` debe llamar a `Policies.EvaluateCancellation` antes de emitir reembolso. Documentar como deuda técnica.
- **Rechazo vs Cancelación de Pending:** ¿el sistema diferencia "proveedor rechaza" de "proveedor cancela" en el historial (mismo estado final `Cancelled`, distinto `cancellationReason`)? Recomiendo sí para auditoría.
- **NoShow por el proveedor:** ¿existe el caso "proveedor no se presentó" (provider no-show) en v1? Los docs de Policies lo mencionan como `providerNoShow`. Si sí, requiere command y policy propios.
- **Acción masiva (bulk confirm):** para proveedores con alto volumen, ¿se contempla confirmar múltiples Pending a la vez en v2?
- **Reminder automático:** ¿el sistema envía recordatorios al proveedor antes del slot (ej: 24h antes) para facilitar el flujo de no-show? Ver `02.reminders.md`.

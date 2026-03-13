---
title: Flow 14 — Moderación (Block, Mute, Report)
description: >
  Un usuario puede bloquear o silenciar otro Profile, y reportar contenido o comportamientos
  inapropiados. Los reportes alimentan una cola de moderación interna. El módulo Trust & Safety
  aplica sanciones (warn, suspend, ban) según las políticas de la plataforma.
status: draft
version: 1
---

# Flow 14 — Moderación (Block, Mute, Report)

## 1. Resumen

- **Goal:** proteger a los usuarios de comportamientos inapropiados y mantener la integridad
  de la plataforma mediante herramientas de auto-moderación (block, mute) y reporte a
  moderación interna (report).
- **Actores:**
  - **Reporter / Blocker:** cualquier usuario autenticado.
  - **Moderator (interno):** staff de VyteMerge con acceso al panel de moderación.
  - **Reportado:** Profile o contenido (Post, Listing) denunciado.
- **Surfaces:**
  - Block / Mute: menú contextual en Profile card, Post card, Listing detail.
  - Report: modal de reporte (Profile, Post, Booking) — accesible desde el mismo menú.
  - Panel de moderación: `/admin/moderation` (uso interno; fuera de scope MFEs usuario).

---

## 2. Domain Context

### Block
Relación bilateral que impide toda interacción entre dos Profiles:
- El bloqueado no puede ver el Profile del bloqueador ni enviarle Follow Requests.
- Se elimina el Follow en ambas direcciones si existía.
- Silencioso: el bloqueado no recibe notificación de que fue bloqueado.

### Mute
Relación unilateral que oculta el contenido del Profile silenciado en el feed del actor:
- No afecta el Follow graph.
- El silenciado no se entera.
- El actor sigue viendo el Profile del silenciado en su following list.

### Report
Denuncia de contenido o comportamiento inapropiado:
- Tipos: `Profile`, `Post`, `Listing`, `Booking` (comportamiento fuera de plataforma).
- El reporte no genera acción automática; alimenta una cola de revisión manual (v1).
- Acumulación de reportes puede disparar revisión automática (v2).

### Relación con Social module
Block y Mute son parte del Social BC (`social` schema). Reports pertenecen al
Trust & Safety BC (`trust_safety` schema).

---

## 3. Preconditions

- Reporter tiene cuenta activa.
- El target (Profile, Post o Listing) existe.
- Block: el Reporter no puede bloquearse a sí mismo.
- Mute: el Reporter no puede silenciarse a sí mismo.

---

## 4. Trigger

- Usuario pulsa "Bloquear" en el menú contextual de un Profile o Post card.
- Usuario pulsa "Silenciar" en el menú contextual de un Post o Profile card.
- Usuario pulsa "Reportar" en el menú contextual de un Post, Profile o Listing.

---

## 5. Main Flow

### Capacidad A — Block

1. Actor abre menú contextual sobre un Profile card o Post card.
2. Actor pulsa "Bloquear a [nombre]".
3. Sistema muestra modal de confirmación: "¿Querés bloquear a [nombre]? No podrá ver tu perfil
   ni enviarte solicitudes de seguimiento."
4. Actor confirma.
5. Sistema:
   - Crea el registro `Block { blockerId, blockedId }`.
   - Elimina el Follow en ambas direcciones si existía.
   - El bloqueado desaparece del feed del Actor.
   - Actor desaparece del feed del bloqueado.
6. (Opcional) Actor puede "Desbloquear" desde `/profiles/:id` si navega directamente.

### Capacidad B — Mute

1. Actor abre menú contextual sobre un Post card o Profile card.
2. Actor pulsa "Silenciar a [nombre]".
3. Sistema registra `Mute { muterId, mutedId }` sin notificación al silenciado.
4. Posts del silenciado dejan de aparecer en el Feed del Actor.
5. El Follow graph no cambia.
6. Actor puede deshacer el Mute desde su lista de silenciados o desde el mismo perfil.

### Capacidad C — Report

1. Actor abre menú contextual sobre un Profile, Post o Listing.
2. Actor pulsa "Reportar".
3. Sistema muestra formulario de reporte:
   - **Tipo de contenido:** Profile | Post | Listing | Booking
   - **Razón:** `Spam | HateSpeech | FakeProfile | InappropriateContent | Harassment | Scam | Other`
   - **Descripción:** texto libre (opcional, max 500 chars)
4. Actor envía el reporte.
5. Sistema crea `Report { reporterId, targetType, targetId, reason, description }` en estado `Open`.
6. Sistema confirma al Actor: "Tu reporte fue enviado. Lo revisaremos."
7. (Background) El moderador revisa la cola de reportes en el panel interno.
8. Moderador toma decisión:
   - **Dismiss:** reporte inválido o ya resuelto.
   - **Warn:** enviar aviso al reportado.
   - **Suspend:** suspender temporalmente el Profile reportado.
   - **Ban:** suspensión permanente.
   - **RemoveContent:** eliminar Post o Listing reportado.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|----------------|
| Actor bloquea a alguien que ya lo bloqueó | Se crea un bloqueo bilateral; ambos ven al otro como bloqueado |
| Bloqueado intenta ver el Profile del bloqueador | Recibe `404 Not Found` (no revela que fue bloqueado) |
| Bloqueado intenta seguir al bloqueador | Sistema rechaza con `403` |
| Actor reporta el mismo contenido dos veces | Sistema rechaza segundo reporte: "Ya reportaste este contenido" |
| Reporte de Booking (comportamiento fuera de plataforma) | Se registra pero el Booking en sí no se cancela automáticamente |
| Actor bloquea a un Provider con Bookings futuros | Los Bookings existentes no se cancelan; el bloqueo aplica para interacciones futuras |
| Profile suspendido intenta hacer login | Sistema retorna `403` con mensaje de cuenta suspendida y contacto de soporte |
| Actor silencia a alguien y luego lo dessilencia | Posts vuelven a aparecer en el feed (no hay historial de mutes públicos) |

---

## 7. Data Model (v1 minimal)

```
-- Social schema
Block {
  id:         UUID
  blockerId:  UUID   -- FK Profile
  blockedId:  UUID   -- FK Profile
  createdAt:  DateTime
}

Mute {
  id:        UUID
  muterId:   UUID   -- FK Profile
  mutedId:   UUID   -- FK Profile
  createdAt: DateTime
}

-- Trust Safety schema
Report {
  id:            UUID
  reporterId:    UUID
  targetType:    Profile | Post | Listing | Booking
  targetId:      UUID
  reason:        Spam | HateSpeech | FakeProfile | InappropriateContent | Harassment | Scam | Other
  description:   string?     -- texto libre, max 500 chars
  status:        Open | UnderReview | Resolved | Dismissed
  resolution:    Warn | Suspend | Ban | RemoveContent | null
  moderatorId:   UUID?       -- staff que resolvió
  resolvedAt:    DateTime?
  createdAt:     DateTime
}

ProfileSanction {
  id:          UUID
  profileId:   UUID
  type:        Warning | Suspension | Ban
  reason:      string
  expiresAt:   DateTime?     -- null = permanente (Ban)
  createdAt:   DateTime
}
```

---

## 8. Commands

| Command | Aggregate | Precondición |
|---------|-----------|--------------|
| `BlockProfileCommand` | `Block` | Actor ≠ Target; no bloqueado ya |
| `UnblockProfileCommand` | `Block` | Block existe |
| `MuteProfileCommand` | `Mute` | Actor ≠ Target; no silenciado ya |
| `UnmuteProfileCommand` | `Mute` | Mute existe |
| `ReportContentCommand` | `Report` | Target existe; Actor no reportó mismo target reciente |
| `ResolveReportCommand` | `Report` | Status=Open/UnderReview; caller=Moderator |
| `SanctionProfileCommand` | `ProfileSanction` | caller=Moderator |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `ProfileBlocked` | `BlockProfileCommand` | Social: elimina Follow en ambas direcciones; Feed: excluye contenido |
| `ProfileUnblocked` | `UnblockProfileCommand` | — (no restaura Follow automáticamente) |
| `ProfileMuted` | `MuteProfileCommand` | Feed: excluye posts del silenciado para el muter |
| `ProfileUnmuted` | `UnmuteProfileCommand` | Feed: reactiva posts del des-silenciado |
| `ContentReported` | `ReportContentCommand` | Trust & Safety: Report en cola de moderación |
| `ReportResolved` | `ResolveReportCommand` | Notification al reporter (opcional v2) |
| `ProfileSanctioned` | `SanctionProfileCommand` | Auth: bloquea acceso si Suspension/Ban; Notification al sancionado |

---

## 10. Invariants

1. Un Profile no puede bloquearse a sí mismo ni silenciarse a sí mismo.
2. El Block es silencioso: el bloqueado no recibe notificación.
3. El Mute es silencioso: el silenciado no recibe notificación.
4. Un Booking existente no se cancela automáticamente al bloquearse las partes.
5. Un perfil Banned no puede crear contenido ni hacer Bookings.
6. El Reporter no puede reportar el mismo `targetId` con el mismo `reason` dentro de 24h.
7. Solo Moderators pueden ejecutar `ResolveReportCommand` y `SanctionProfileCommand`.
8. Un Report `Dismissed` o `Resolved` no puede reabrirse (se crea un nuevo Report si aplica).
9. Al bloquear: se elimina el Follow en ambas direcciones si existía; no se restaura al desbloquear.
10. `ProfileSanction` con `expiresAt=null` es una sanción permanente (Ban).

---

## 11. Outputs

- Registro de Block: el bloqueado no puede ver ni interactuar con el bloqueador.
- Registro de Mute: Posts del silenciado no aparecen en el Feed del actor.
- Report en cola de moderación interna: visible al equipo de Trust & Safety.
- ProfileSanction aplicada (si el moderador lo decide): acceso restringido.

---

## 12. Technical Mapping (Draft)

### Backend

**Social module** (`social` schema):
```
Commands: BlockProfile, UnblockProfile, MuteProfile, UnmuteProfile
Endpoints:
  POST   /profiles/:profileId/block    → BlockProfileCommand
  DELETE /profiles/:profileId/block    → UnblockProfileCommand
  POST   /profiles/:profileId/mute     → MuteProfileCommand
  DELETE /profiles/:profileId/mute     → UnmuteProfileCommand
  GET    /profiles/blocked             → lista de Profiles bloqueados por el autenticado
  GET    /profiles/muted               → lista de Profiles silenciados por el autenticado
```

> Block y Mute ya tienen endpoints definidos en `social.md`. Este flow los documenta con el contexto completo.

**Trust & Safety module** (nuevo; `trust_safety` schema):
```
Commands: ReportContent, ResolveReport, SanctionProfile
Endpoints:
  POST   /reports                      → ReportContentCommand
  GET    /admin/moderation/reports     → lista de reportes (Moderator only)
  PATCH  /admin/moderation/reports/:id → ResolveReportCommand
  POST   /admin/moderation/sanctions   → SanctionProfileCommand
```

### Frontend

**User-facing (vt-social-mfe o vt-shell):**
- Menú contextual en Profile card / Post card: "Bloquear", "Silenciar", "Reportar"
- Modal de confirmación para Block
- Modal de reporte con selector de razón + texto libre
- Settings → "Cuentas bloqueadas" / "Cuentas silenciadas"

**Admin panel (fuera de scope MFEs usuario):**
- `/admin/moderation`: tabla de Reports (Open/UnderReview)
- Detalle de Report con acciones: Dismiss, Warn, Suspend, Ban, RemoveContent

---

## 13. Acceptance Criteria

- [ ] Actor puede bloquear a un Profile; el bloqueado no puede ver el Profile del actor ni enviarle Follow Requests.
- [ ] Al bloquear, el Follow en ambas direcciones se elimina si existía.
- [ ] El bloqueado no recibe notificación de que fue bloqueado.
- [ ] Actor puede desbloquear; el Follow no se restaura automáticamente.
- [ ] Actor puede silenciar a un Profile; los Posts del silenciado no aparecen en el Feed del actor.
- [ ] El silenciado no recibe notificación de que fue silenciado.
- [ ] Actor puede reportar un Profile, Post o Listing seleccionando razón y descripción opcional.
- [ ] El reporte aparece en la cola de moderación interna.
- [ ] Un actor no puede reportar el mismo contenido con la misma razón en menos de 24h.
- [ ] Al bloquear a alguien que ya bloqueó al actor, el bloqueo es bilateral (ambos ven `404` al visitar el perfil del otro).

---

## 14. NEEDS-CLARIFICATION

- **Notificación al reporter:** ¿se notifica al reporter cuando su reporte es resuelto (Dismiss/Warn/Ban)? v1: no notificar. v2: notificación simple.
- **Admin panel scope:** ¿el panel de moderación es un MFE propio (`vt-admin-mfe`) o vive en el backend directamente? Recomendación v1: herramienta interna básica (tabla + acciones simples).
- **Sanción automática:** ¿se activa suspensión automática si un Profile acumula N reportes? v1: manual. v2: reglas automáticas.
- **Block y Bookings existentes:** si A bloquea a B y tienen un Booking pendiente, ¿se cancela automáticamente? Recomendación: no cancelar automáticamente; gestión manual.
- **Moderación de Listings:** si un Listing es reportado y el moderador lo remueve, ¿el Listing pasa a `Archived` o a un estado `Removed` distinto?

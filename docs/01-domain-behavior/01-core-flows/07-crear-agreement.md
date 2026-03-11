---
title: Flow 07 — Crear Agreement
description: >
  Un Owner (Profile) crea un Agreement formal con otro Profile (Participant),
  define tipo, scope, permisos y términos, y lo envía como invitación. El
  Participant acepta o rechaza. Una vez Active, el módulo Access lo usa para
  enforcement de visibilidad y operación.
status: draft
version: 1
---

# Flow 07 — Crear Agreement

## 1. Resumen
- **Goal:** que un Owner pueda formalizar una relación con otro Profile
  (Sharing, Delegation o Partner), definir permisos y scope, y activarla
  mediante aceptación del Participant.
- **Actores:**
  - **Owner:** Profile que inicia, configura y puede revocar el Agreement.
  - **Participant:** Profile invitado; solo puede aceptar o rechazar.
- **Surfaces:**
  - Creación: shortcut contextual en vt-user-profile-mfe / vt-catalog-mfe /
    vt-agreements-mfe → formulario en `vt-agreements-mfe`.
  - Aceptación: email + inbox en `/agreements/inbox`.
  - Gestión: `/agreements/:id` en `vt-agreements-mfe`.

---

## 2. Domain Context

### Agreement como capa de relación formal
Los Agreements no son visibles para el usuario final como "contratos" —
aparecen en UX como "Staff", "Calendarios compartidos" o "Partners".
Son el mecanismo subyacente que el módulo **Access** consulta para decidir
quién puede ver o hacer qué sobre un recurso.

### Tipos (v1)
| Tipo | Qué habilita | Ejemplo |
|------|-------------|---------|
| **Sharing** | Compartir visibilidad/lectura | Colegio → padres (timeline escolar) |
| **Delegation (Staff)** | Delegación operativa | Hospital → secretaria (gestión turnos) |
| **Partner / Reseller** | Relación comercial | Proveedor → revendedor de Listings |

### Scope
Define qué recursos cubre el Agreement:
| Scope | Recursos |
|-------|---------|
| `Profile` | Cuenta completa del Owner |
| `Timeline` | Una agenda específica |
| `Catalog` | Catálogo completo (Services/Products) |
| `Listing` | Una oferta puntual |

### Relación con Access
Access consulta el Agreement activo para aplicar permisos y VisibilityMode
sobre los recursos del Scope. Sin Agreement activo, Access usa solo
privacidad + follows.

---

## 3. Preconditions
- Owner tiene Profile activo.
- Participant tiene Profile activo y cuenta en VyteMerge
  (v1: el Participant debe tener cuenta previa; si no existe, el email
  incluye link de registro).
- Owner tiene control efectivo sobre los recursos del Scope declarado
  (invariante: no puede delegar lo que no controla).

---

## 4. Trigger

**Vía shortcut contextual** (camino principal):
- vt-user-profile-mfe → "Agregar colaborador" → `type=delegation`
- vt-catalog-mfe → "Asociar partner" → `type=partner, scope=listing`
- vt-agreements-mfe → "Compartir agenda" → `type=sharing, scope=timeline`
  *(shortcut temporal hasta que exista vt-timeline-mfe — ver NEEDS-CLARIFICATION)*

**Vía panel de Agreements:**
- Owner navega a `/agreements` → pulsa "Nuevo Agreement".

---

## 5. Main Flow

### Fase 1 — Configuración (Owner)

1. Owner abre el formulario de creación (desde shortcut o `/agreements/new`).
2. Sistema pre-carga `type` y `scope` si vienen como query params del shortcut.
3. Owner completa:
   - **Requerido:**
     - `type` — Sharing | Delegation | Partner.
     - `participantProfileId` — busca el Profile del Participant por nombre/email.
     - `scope[]` — al menos un scope resource (Profile, Timeline, Catalog o Listing).
     - `permissions[]` — al menos un permiso del set disponible para el `type` elegido.
   - **Opcional:**
     - `customTerms` — texto libre con condiciones adicionales.
     - `expiresAt` — fecha de vencimiento.
4. Sistema valida que el Owner controla los recursos del Scope declarado.
5. Sistema muestra preview del Agreement (términos, permisos, scope).
6. Owner pulsa **"Enviar invitación"**.

### Fase 2 — Invitación (sistema → Participant)

7. Sistema crea el `Agreement` en estado **Invited**.
8. Sistema envía:
   - Email al Participant con link de aceptación y resumen del Agreement.
   - Notificación in-app en `/agreements/inbox` del Participant.

### Fase 3 — Revisión y decisión (Participant)

9. Participant abre el link del email o navega a su inbox.
10. Sistema muestra: tipo, scope, permisos, Platform Terms, Custom Terms
    (si existen), Owner que invita, fecha de vencimiento (si existe).
11. Participant toma su decisión:
    - **Acepta** → pulsa "Aceptar". *(Ver paso 12)*
    - **Rechaza** → pulsa "Rechazar" (opcionalmente con motivo). *(Ver paso 14)*

### Fase 4 — Aceptación

12. Sistema transiciona el Agreement a **Active**.
13. Sistema:
    - Emite `AgreementAccepted` → Access module actualiza enforcement.
    - Notifica al Owner (email + in-app): "Fulanito aceptó el Agreement".
    - Muestra al Participant el Agreement Detail en estado Active.

### Fase 5 — Rechazo (alternate)

14. Sistema transiciona a **Rejected**.
15. Sistema notifica al Owner: "Fulanito rechazó el Agreement".
    El Owner puede crear un nuevo Agreement con condiciones modificadas.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|---------------|
| Participant no tiene cuenta | Email incluye link de registro. Tras registrarse, el link de aceptación redirige a la invitación pendiente. |
| Owner edita permisos de Agreement Active | `UpdateAgreementPermissions`: Agreement sigue Active; Participant recibe notificación del cambio; Access module actualiza enforcement inmediatamente. |
| Owner revoca Agreement de tipo Delegation con Bookings activos | Agreement pasa a Revoked; Participant pierde acceso inmediatamente; Bookings asignados al Participant quedan `unassigned`; Owner recibe alerta con lista de Bookings afectados. |
| Agreement llega a `expiresAt` | Sistema auto-transiciona a Expired (background job); Access enforcement se corta; Owner y Participant notificados. |
| Participant rechaza — Owner quiere re-intentar | Owner crea un nuevo Agreement. El Agreement Rejected permanece en historial de auditoría. |
| Owner intenta delegar recursos que no controla | `CreateAgreement` falla con error de validación; sistema indica qué recursos no están bajo su control. |
| Participant usa recursos después de Revoked/Expired | Access module retorna 403; UI muestra "No tenés permisos activos". |
| Agreement Accepted — recurso del Scope es archivado | Agreement permanece Active pero el recurso archivado ya no genera operación. (NEEDS-CLARIFICATION) |
| Owner es también Participant (mismo Profile) | Sistema rechaza la creación: no se puede crear Agreement consigo mismo. |
| Tipo Partner — Participant intenta publicar Listing derivado sin permiso `DeriveListing` | Access rechaza la operación con 403. |

---

## 7. Data Model (v1 minimal)

```
Agreement {
  id:                       UUID
  ownerProfileId:           UUID             -- FK Profile (inicia y controla)
  participantProfileId:     UUID             -- FK Profile (acepta o rechaza)
  type:                     Sharing | Delegation | Partner
  status:                   Draft | Invited | Active | Rejected | Revoked | Expired
  scope:                    AgreementScope[] -- al menos 1
  permissions:              Permission[]     -- al menos 1
  platformTermsVersion:     string           -- ej: "v1"; siempre incluido
  customTerms:              string?          -- texto libre; opcional
  expiresAt:                DateTime?        -- opcional
  invitedAt:                DateTime?
  acceptedAt:               DateTime?
  rejectedAt:               DateTime?
  revokedAt:                DateTime?
  createdAt:                DateTime
  updatedAt:                DateTime
}

AgreementScope {
  type:        Profile | Timeline | Catalog | Listing
  resourceId:  UUID                          -- ID del recurso específico del Owner
}

Permission (enum v1):
  ViewBusyOnly      -- ver solo ocupado/libre (mínimo)
  ViewDetails       -- ver detalle completo
  ManageSchedule    -- gestionar agenda: bloqueos, disponibilidad
  ManageBookings    -- confirmar, cancelar, reprogramar Bookings
  ManageListings    -- publicar, archivar, editar Listings
  DeriveListing     -- crear Listing derivado (Partner) -- NEEDS-CLARIFICATION: v1 o v2?
  Resell            -- revender / distribuir (Partner)  -- NEEDS-CLARIFICATION: v1 o v2?
```

---

## 8. Commands

| Command | Aggregate | Precondition |
|---------|-----------|--------------|
| `CreateAgreement` | `Agreement` | Owner activo; Participant activo; Owner controla los recursos del Scope |
| `SendAgreementInvitation` | `Agreement` | Status = Draft |
| `AcceptAgreement` | `Agreement` | Status = Invited; caller = Participant |
| `RejectAgreement` | `Agreement` | Status = Invited; caller = Participant |
| `UpdateAgreementPermissions` | `Agreement` | Status = Active; caller = Owner |
| `RevokeAgreement` | `Agreement` | Status = Active \| Invited; caller = Owner |
| `ExpireAgreement` | `Agreement` | Status = Active; `expiresAt` alcanzado (sistema) |

---

## 9. Events

| Event | Disparado por | Efectos downstream |
|-------|-------------|-------------------|
| `AgreementCreated` | `CreateAgreement` | — |
| `AgreementInvitationSent` | `SendAgreementInvitation` | Notification: email + in-app al Participant |
| `AgreementAccepted` | `AcceptAgreement` | Access: enforcement activado; Notification al Owner |
| `AgreementRejected` | `RejectAgreement` | Notification al Owner |
| `AgreementPermissionsUpdated` | `UpdateAgreementPermissions` | Access: enforcement actualizado; Notification al Participant |
| `AgreementRevoked` | `RevokeAgreement` | Access: enforcement cortado; Bookings → unassigned (si Delegation); Notification a ambos |
| `AgreementExpired` | `ExpireAgreement` | Access: enforcement cortado; Notification a ambos |

---

## 10. Invariants

1. Solo el **Owner** puede ejecutar `CreateAgreement`, `UpdateAgreementPermissions` y `RevokeAgreement`.
2. Solo el **Participant** puede ejecutar `AcceptAgreement` y `RejectAgreement`.
3. Un Agreement en estado `Revoked`, `Rejected` o `Expired` no puede volver a `Active`.
4. Platform Terms son siempre incluidas; su versión se registra en el momento de la aceptación.
5. El Owner no puede delegar permisos sobre recursos que no controla.
6. Un Agreement no puede tener `ownerProfileId == participantProfileId`.
7. Al revocar un Agreement de tipo Delegation, los Bookings asignados al Participant quedan `unassigned` (no se cancelan automáticamente).
8. Editar permisos de un Agreement Active no requiere nueva aceptación del Participant, pero dispara notificación.
9. Un Agreement Expired o Revoked se conserva como registro histórico (no se elimina).
10. El `platformTermsVersion` registrado en la aceptación es inmutable post-aceptación.

---

## 11. Outputs

- `Agreement` en estado **Active**, asociado a Owner y Participant.
- Acceso operacional del Participant sobre los recursos del Scope (enforcement vía Access module).
- Historial de auditoría: quién aceptó, qué versión de términos, cuándo.
- Notificaciones enviadas a ambas partes.

---

## 12. Technical Mapping (Draft)

### Backend

**Módulo nuevo:** `Agreements`
- **Aggregate:** `Agreement`
- **DbContext:** `AgreementsDbContext` (schema: `agreements`)
- **Endpoints draft:**
  ```
  POST   /agreements                   → CreateAgreement (Draft)
  POST   /agreements/{id}/invite       → SendAgreementInvitation
  POST   /agreements/{id}/accept       → AcceptAgreement
  POST   /agreements/{id}/reject       → RejectAgreement
  PATCH  /agreements/{id}/permissions  → UpdateAgreementPermissions
  POST   /agreements/{id}/revoke       → RevokeAgreement
  GET    /agreements                   → lista del Profile autenticado (owner o participant)
  GET    /agreements/inbox             → invitaciones pendientes (participant)
  GET    /agreements/{id}              → detalle
  ```
- **Integración Access:** handlers de `AgreementAccepted`, `AgreementPermissionsUpdated`,
  `AgreementRevoked`, `AgreementExpired` actualizan enforcement en el módulo Access.
- **Integración Bookings:** handler de `AgreementRevoked` (type=Delegation) marca como
  `unassigned` los Bookings con `staffProfileId = participantProfileId`.
- **Background job:** chequeo periódico de `expiresAt` → dispara `ExpireAgreement`.

### Frontend

**MFE nuevo:** `vt-agreements-mfe`
```
/agreements              → panel: mis agreements activos (owner + participant)
/agreements/inbox        → invitaciones pendientes de aceptar/rechazar
/agreements/new          → formulario de creación (acepta query params del shortcut)
/agreements/:id          → Agreement Detail (Active/Revoked/Expired, read-only)
/agreements/:id/edit     → editar permisos (owner only, status = Active)
```

**Shortcuts contextuales (deep-link a vt-agreements-mfe):**
- `vt-user-profile-mfe` (Profile Detail) → "Agregar colaborador"
  → `/agreements/new?type=delegation&scope=profile&resourceId={profileId}`
- `vt-catalog-mfe` (Listing Detail) → "Asociar partner"
  → `/agreements/new?type=partner&scope=listing&resourceId={listingId}`
- *(Futuro `vt-timeline-mfe`)* → "Compartir agenda"
  → `/agreements/new?type=sharing&scope=timeline&resourceId={timelineId}`

**Toolkit components:** NEEDS-CLARIFICATION (inventario pendiente). Candidatos:
permission checkboxes, scope selector, profile search/picker, status badge,
terms viewer (readonly), notification badge (inbox count).

**UI states:**
- Formulario de creación con pre-carga de type/scope desde shortcut
- Preview de términos antes de enviar
- Inbox con invitaciones pendientes (Aceptar / Rechazar + motivo)
- Agreement Detail: Active / Revoked / Expired (read-only + historial de auditoría)
- Panel de edición de permisos (owner only, status = Active)
- Estado vacío (sin agreements activos / sin inbox pendiente)

---

## 13. Acceptance Criteria

- [ ] Owner puede crear un Agreement de tipo Sharing, Delegation o Partner.
- [ ] Sistema valida que el Owner controla los recursos del Scope declarado.
- [ ] Al enviar la invitación, el Participant recibe email + notificación in-app.
- [ ] Participant puede aceptar o rechazar desde el email o el inbox.
- [ ] Al aceptar, el Agreement pasa a Active y el Access module actualiza enforcement.
- [ ] Al rechazar, el Owner recibe notificación y puede crear un nuevo Agreement.
- [ ] Owner puede editar permisos de un Agreement Active; el Participant es notificado.
- [ ] Al revocar un Agreement de tipo Delegation, los Bookings asignados quedan unassigned.
- [ ] Agreement con `expiresAt` alcanzado es auto-transicionado a Expired por el sistema.
- [ ] Un Agreement Revoked/Rejected/Expired no puede volver a Active.
- [ ] Shortcuts contextuales en vt-user-profile-mfe y vt-catalog-mfe pre-cargan type y scope.
- [ ] `/agreements/inbox` muestra solo invitaciones pendientes del usuario autenticado.
- [ ] Agreement Detail muestra historial de auditoría (quién aceptó, versión de términos, timestamps).

---

## 14. NEEDS-CLARIFICATION

- **`vt-timeline-mfe`:** el shortcut "Compartir agenda" necesita una superficie.
  ¿En v1 el shortcut vive temporalmente en vt-agreements-mfe, o se crea `vt-timeline-mfe`
  como parte de este flow? Decisión de arquitectura frontend pendiente.
- **Multi-scope:** ¿puede un Agreement cubrir múltiples recursos del mismo tipo
  (ej: 3 Listings para un Partner), o exactamente 1 recurso por scope en v1?
- **Permission set final MVP:** de los 7 permisos definidos, ¿`DeriveListing` y `Resell`
  entran en v1, o se reservan para v2 (Partner flow completo)?
- **Agreement Accepted + recurso archivado:** si el Scope incluye un recurso que luego
  es archivado, ¿el Agreement sigue siendo válido para otros resources del mismo Scope?
- **Notifications (BC 05):** ¿el módulo Communication puede enviar emails en v1,
  o la integración con Communication BC está pendiente?
- **CreateAgreement: 1 paso vs 2 pasos:** ¿el formulario crea Draft y luego "Enviar"
  es un segundo paso? ¿O es un único paso (crear + invitar)? Impacta UX y si el estado
  `Draft` tiene uso práctico en este flujo.

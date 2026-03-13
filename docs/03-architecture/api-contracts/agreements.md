# API Contracts — Agreements

Base path: `/api/v1`
Module: `Vt.Modules.Agreements`
Schema: `agreements`

> Cubre el ciclo de vida de Agreements formales entre Profiles (Sharing, Delegation, Partner).
> El módulo Access consulta los Agreements activos para enforcement de visibilidad y permisos.
> Ver flow `07-crear-agreement.md` y spec pack `DRAFT-create-agreement`.

---

## Agreement Types

| Tipo | Qué habilita |
|------|-------------|
| `Sharing` | Compartir visibilidad/lectura de un recurso |
| `Delegation` | Delegar operación (Staff) sobre Bookings, Listings, Timeline |
| `Partner` | Relación comercial: resell, distribución de Listings |

## Agreement Status Lifecycle

```
Draft → Invited → Active
                 ↘ Rejected
Active → Revoked
Active → Expired  (si expiresAt alcanzado)
```

---

## Create & Invite

### `POST /agreements` 🚧

Crea un Agreement en estado `Draft` o `Invited` (si se envía la invitación en el mismo paso).

**Auth:** usuario autenticado. Solo el Owner puede crear.

**Request:**
```json
{
  "ownerProfileId": "uuid",
  "participantProfileId": "uuid",
  "type": "Sharing | Delegation | Partner",
  "scope": [
    { "type": "Profile | Timeline | Catalog | Listing", "resourceId": "uuid" }
  ],
  "permissions": ["ViewBusyOnly | ViewDetails | ManageSchedule | ManageBookings | ManageListings | DeriveListing | Resell"],
  "customTerms": "string | null",
  "expiresAt": "ISO8601 | null",
  "sendInvitation": true
}
```

> Si `sendInvitation=true`, el Agreement se crea directamente en `Invited` y se despachan
> notificaciones. Si `false`, queda en `Draft` para envío posterior.

**Command dispatched:** `CreateAgreementCommand` → `SendAgreementInvitationCommand` (si `sendInvitation=true`)
**Events emitted:** `AgreementCreated` → `AgreementInvitationSent` (si aplica)

**Response:** `201 Created`
```json
{ "id": "uuid", "status": "Draft | Invited" }
```

**Errors:**
- `422` si `ownerProfileId == participantProfileId`.
- `422` si el Owner no controla los recursos del Scope declarado.
- `422` si `scope` está vacío o `permissions` está vacío.

---

### `POST /agreements/:agreementId/invite` 🚧

Envía la invitación de un Agreement en estado `Draft`.

**Command dispatched:** `SendAgreementInvitationCommand`
**Event emitted:** `AgreementInvitationSent` → Notification: email + in-app al Participant

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Invited" }
```

---

## Read

### `GET /agreements` 🚧

Lista los Agreements del usuario autenticado (como Owner o como Participant).

**Query params:** `?role=owner|participant&status=Active|Invited|Rejected|Revoked|Expired&page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "type": "Sharing | Delegation | Partner",
      "status": "Draft | Invited | Active | Rejected | Revoked | Expired",
      "ownerProfileId": "uuid",
      "ownerProfileName": "string",
      "participantProfileId": "uuid",
      "participantProfileName": "string",
      "scope": [{ "type": "string", "resourceId": "uuid" }],
      "expiresAt": "ISO8601 | null",
      "createdAt": "ISO8601"
    }
  ],
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

### `GET /agreements/inbox` 🚧

Lista invitaciones pendientes (status=`Invited`) recibidas por el usuario autenticado como Participant.

**Response:** mismo shape que `GET /agreements` filtrado por `status=Invited, role=participant`.

---

### `GET /agreements/:agreementId` 🚧

Retorna el detalle completo de un Agreement. Visible para Owner y Participant.

**Response:**
```json
{
  "id": "uuid",
  "type": "Sharing | Delegation | Partner",
  "status": "Draft | Invited | Active | Rejected | Revoked | Expired",
  "ownerProfileId": "uuid",
  "ownerProfileName": "string",
  "participantProfileId": "uuid",
  "participantProfileName": "string",
  "scope": [
    { "type": "Profile | Timeline | Catalog | Listing", "resourceId": "uuid" }
  ],
  "permissions": ["string"],
  "platformTermsVersion": "string",
  "customTerms": "string | null",
  "expiresAt": "ISO8601 | null",
  "invitedAt": "ISO8601 | null",
  "acceptedAt": "ISO8601 | null",
  "rejectedAt": "ISO8601 | null",
  "revokedAt": "ISO8601 | null",
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601"
}
```

---

## Participant Actions

### `POST /agreements/:agreementId/accept` 🚧

Participant acepta la invitación. Agreement → `Active`.

**Auth:** solo el Participant puede llamar este endpoint.

**Command dispatched:** `AcceptAgreementCommand`
**Events emitted:** `AgreementAccepted` → Access module activa enforcement; Notification al Owner

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Active" }
```

---

### `POST /agreements/:agreementId/reject` 🚧

Participant rechaza la invitación. Agreement → `Rejected`.

**Request:**
```json
{ "reason": "string | null" }
```

**Command dispatched:** `RejectAgreementCommand`
**Event emitted:** `AgreementRejected` → Notification al Owner

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Rejected" }
```

---

## Owner Actions

### `PATCH /agreements/:agreementId/permissions` 🚧

Owner actualiza los permisos de un Agreement `Active`. No requiere nueva aceptación del Participant.

**Auth:** solo el Owner puede llamar este endpoint.

**Request:**
```json
{
  "permissions": ["ViewBusyOnly | ViewDetails | ManageSchedule | ManageBookings | ManageListings"]
}
```

**Command dispatched:** `UpdateAgreementPermissionsCommand`
**Event emitted:** `AgreementPermissionsUpdated` → Access: enforcement actualizado; Notification al Participant

---

### `POST /agreements/:agreementId/revoke` 🚧

Owner revoca un Agreement `Active` o `Invited`.

**Auth:** solo el Owner puede llamar este endpoint.

**Command dispatched:** `RevokeAgreementCommand`
**Events emitted:** `AgreementRevoked`
  - Si `type=Delegation`: Booking module marca Bookings asignados al Participant como `unassigned`.
  - Notification a Owner y Participant.

**Response:** `200 OK`
```json
{ "id": "uuid", "status": "Revoked" }
```

---

## Permissions Reference

| Permission | Aplica en tipo | Descripción |
|------------|----------------|-------------|
| `ViewBusyOnly` | Sharing, Delegation | Ver solo ocupado/libre |
| `ViewDetails` | Sharing, Delegation | Ver detalle completo |
| `ManageSchedule` | Delegation | Gestionar agenda: bloqueos, disponibilidad |
| `ManageBookings` | Delegation | Confirmar, cancelar, reprogramar Bookings |
| `ManageListings` | Delegation | Publicar, archivar, editar Listings |
| `DeriveListing` | Partner | Crear Listing derivado (NEEDS-CLARIFICATION: v1 o v2) |
| `Resell` | Partner | Revender / distribuir (NEEDS-CLARIFICATION: v1 o v2) |

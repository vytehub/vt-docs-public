# API Contracts — Notifications

Base path: `/api/v1`
Module: `Vt.Modules.Communication` *(planificado — modulo propio, a implementar)*

> Cubre notificaciones in-app, preferencias de canal (email, push) y registro de dispositivos.
> Los eventos que disparan notificaciones estan documentados en `docs/vt-docs/public/01-domain-behavior/03-cross-cutting/notifications-reminders.md`.
>
> Ningun endpoint de este modulo esta implementado todavia. Este documento es el contrato de diseno target.

---

## Notifications

### `GET /notifications`

Lista las notificaciones del usuario autenticado. Paginadas, mas recientes primero.

**Auth:** requerida (usuario autenticado)

**Query params:** `?unreadOnly=false&page=1&pageSize=20`

**Response:** `200 OK`
```json
{
  "items": [
    {
      "id": "uuid",
      "type": "booking_confirmed | booking_cancelled | booking_requested | booking_completed | booking_no_show | booking_reminder | reschedule_proposed | reschedule_confirmed | reschedule_rejected | follow_request | follow_approved | post_reaction | listing_published | listing_shared | system",
      "title": "string",
      "body": "string",
      "read": false,
      "data": {
        "resourceType": "booking | profile | post | listing | null",
        "resourceId": "uuid | null"
      },
      "createdAt": "ISO8601"
    }
  ],
  "unreadCount": 0,
  "total": 0,
  "page": 1,
  "pageSize": 20
}
```

---

### `POST /notifications/{notificationId}/read`

Marca una notificacion como leida.

**Auth:** requerida

**Command dispatched:** `MarkNotificationReadCommand`

**Response:** `204 No Content`

---

### `POST /notifications/read-all`

Marca todas las notificaciones del usuario como leidas.

**Auth:** requerida

**Command dispatched:** `MarkAllNotificationsReadCommand`

**Response:** `204 No Content`

---

### `DELETE /notifications/{notificationId}`

Elimina una notificacion de la bandeja del usuario.

**Auth:** requerida

**Response:** `204 No Content`

---

## Notification Types

| Type | Trigger Event | Source module | Destinatario |
|------|--------------|---------------|-------------|
| `booking_confirmed` | `BookingConfirmed` | Booking | Attendee |
| `booking_cancelled` | `BookingCancelled` | Booking | Attendee + Provider |
| `booking_requested` | `BookingRequested` | Booking | Provider |
| `booking_reminder` | Scheduled job | Booking | Attendee + Provider |
| `booking_completed` | `BookingCompleted` | Booking | Attendee |
| `booking_no_show` | `BookingNoShow` | Booking | Attendee |
| `reschedule_proposed` | `RescheduleProposed` | Booking | Actor opuesto al proponente |
| `reschedule_confirmed` | `RescheduleConfirmed` | Booking | Ambos actores |
| `reschedule_rejected` | `RescheduleRejected` | Booking | Proponente |
| `follow_request` | `FollowRequestedDomainEvent` | Social | Followee (target del request) |
| `follow_approved` | `FollowAcceptedDomainEvent` | Social | Follower (quien envio el request) |
| `post_reaction` | `ReactionAddedDomainEvent` | Social | Author del Post |
| `listing_published` | `ListingPublished` | Listing | Seguidores del Profile author |
| `listing_shared` | `ListingShared` | Social | Owner del Listing |
| `system` | Interno | Any | Usuario |

> `follow_approved` reemplaza a `follow_accepted` para alinear con el termino canonico `Approve` usado en el dominio Social (`ApproveFollowCommand`, `FollowAcceptedDomainEvent`).

---

## Preferences

### `GET /notifications/preferences`

Retorna las preferencias de notificacion del usuario autenticado.

**Auth:** requerida

**Response:** `200 OK`
```json
{
  "email": {
    "enabled": true,
    "bookingUpdates": true,
    "reminders": true,
    "socialActivity": false,
    "marketing": false
  },
  "push": {
    "enabled": true,
    "bookingUpdates": true,
    "reminders": true,
    "socialActivity": true
  },
  "inApp": {
    "enabled": true,
    "all": true
  }
}
```

---

### `PATCH /notifications/preferences`

Actualiza las preferencias de notificacion. Campos no enviados no se modifican.

**Auth:** requerida

**Request (partial):**
```json
{
  "email": {
    "bookingUpdates": true,
    "reminders": false
  },
  "push": {
    "socialActivity": false
  }
}
```

**Command dispatched:** `UpdateNotificationPreferencesCommand`

**Response:** `200 OK` con las preferencias actualizadas (mismo shape que GET).

---

## Push Device Registration

### `POST /notifications/devices`

Registra un dispositivo para recibir push notifications.

**Auth:** requerida

**Request:**
```json
{
  "token": "string",
  "platform": "ios | android | web",
  "deviceId": "string"
}
```

**Response:** `201 Created`

---

### `DELETE /notifications/devices/{deviceId}`

Desregistra un dispositivo (al hacer logout o cambio de usuario).

**Auth:** requerida

**Response:** `204 No Content`

# API Contracts — Notifications

Base path: `/api/v1`
Module: `Vt.Modules.Communication` *(a definir — puede ser parte de Users o módulo propio)*

> Cubre notificaciones in-app, preferencias de canal (email, push) y registro de dispositivos.
> Los eventos que disparan notificaciones están documentados en `docs/vt-docs/public/01-domain-behavior/02-system-behaviors/notifications-reminders.md`.

---

## Notifications

### `GET /notifications` 🚧

Lista las notificaciones del usuario autenticado. Paginadas, más recientes primero.

**Query params:** `?unreadOnly=false&page=1&pageSize=20`

**Response:**
```json
{
  "items": [
    {
      "id": "uuid",
      "type": "booking_confirmed | booking_cancelled | follow_request | follow_accepted | post_reaction | reminder | system",
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

### `POST /notifications/:notificationId/read` 🚧

Marca una notificación como leída.

**Command dispatched:** `MarkNotificationReadCommand`

**Response:** `204 No Content`

---

### `POST /notifications/read-all` 🚧

Marca todas las notificaciones del usuario como leídas.

**Command dispatched:** `MarkAllNotificationsReadCommand`

**Response:** `204 No Content`

---

### `DELETE /notifications/:notificationId` 🚧

Elimina una notificación de la bandeja del usuario.

**Response:** `204 No Content`

---

## Notification Types

| Type | Trigger Event | Actor |
|------|--------------|-------|
| `booking_confirmed` | `BookingConfirmed` | Attendee |
| `booking_cancelled` | `BookingCancelled` | Attendee + Provider |
| `booking_requested` | `BookingRequested` | Provider |
| `booking_reminder` | Scheduled job | Attendee + Provider |
| `booking_completed` | `BookingCompleted` | Attendee |
| `booking_no_show` | `BookingNoShow` | Attendee |
| `follow_request` | `FollowRequested` | Target Profile |
| `follow_accepted` | `FollowAccepted` | Requester |
| `post_reaction` | `ReactionAdded` | Post owner |
| `listing_published` | `ListingPublished` | Followers del Profile |
| `system` | Internal | User |

---

## Preferences

### `GET /notifications/preferences` 🚧

Retorna las preferencias de notificación del usuario autenticado.

**Response:**
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

### `PATCH /notifications/preferences` 🚧

Actualiza las preferencias de notificación.

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

**Response:** `200 OK` con las preferencias actualizadas.

---

## Push Device Registration

### `POST /notifications/devices` 🚧

Registra un dispositivo para recibir push notifications.

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

### `DELETE /notifications/devices/:deviceId` 🚧

Desregistra un dispositivo (logout, cambio de usuario).

**Response:** `204 No Content`

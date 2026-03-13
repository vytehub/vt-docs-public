---
title: Notifications & Reminders (curated)
description: Eventos que deben disparar notificaciones y recordatorios en flows de booking y social.
---

# Notifications & Reminders (curated)

## Objetivo
Definir qué cambios del dominio generan mensajes a usuarios/negocios.

## Fuentes de eventos → notificación

### Booking
| Evento | Destinatario | Canal |
|---|---|---|
| `BookingRequested` | Provider | push + email |
| `BookingCreated` | Attendee + Provider | push + email |
| `BookingConfirmed` | Attendee | push + email |
| `BookingCancelled` | Attendee + Provider | push + email |
| `BookingCompleted` | Attendee + Provider | push |
| `BookingNoShow` | Attendee (penalidad) | push + email |
| `RescheduleProposed` | Actor opuesto al proponente | push + email |
| `RescheduleConfirmed` | Ambos actores | push |
| `RescheduleRejected` | Proponente | push |

### Social
| Evento | Destinatario | Canal |
|---|---|---|
| `FollowRequested` | Profile owner | push |
| `FollowApproved` | Solicitante | push |
| `PostPublished` | Seguidores del author (eligibles) | push (feed) |
| `ReactionAdded` | Author del Post o Listing | push |
| `ListingShared` | Owner del Listing | push |

> **Mute:** un follow en estado `MUTED` suprime la entrega de `PostPublished` para ese followee en el feed del follower. No genera notificación propia.

## Reminders (booking)
- Reminder previo a start: T-24h y T-2h **NEEDS-CLARIFICATION: configurable por Listing o fijo por plataforma**
- Recordatorio al proveedor si hay Bookings `Pending` sin confirmar (cadencia por definir)
- Recordatorio post-servicio para completar/dejar reseña **NEEDS-CLARIFICATION**

## Links
- Notifications overview: `00-core-domain/04-bounded-contexts/05-communication/02.notifications/01.overview.md`
- Reminders: `00-core-domain/04-bounded-contexts/05-communication/02.notifications/02.reminders.md`
- Flow 03 — Follow, Feed y Posts: `01-domain-behavior/01-core-flows/03-follow-and-feed.md`
- Flow 11 — Cancelar, Reagendar y Policies: `01-domain-behavior/01-core-flows/11-cancelar-reagendar-policies.md`

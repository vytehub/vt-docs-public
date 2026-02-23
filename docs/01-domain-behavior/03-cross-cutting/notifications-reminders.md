---
title: Notifications & Reminders (curated)
description: Eventos que deben disparar notificaciones y recordatorios en flows de booking y social.
---

# Notifications & Reminders (curated)

## Objetivo
Definir qué cambios del dominio generan mensajes a usuarios/negocios.

## Fuentes típicas (events)
- `BookingCreated` / `BookingConfirmed`
- `BookingCancelled`
- `FollowRequested` / `FollowApproved`
- `PostCreated` (según surfaces)

## Reminders (booking)
- reminder previo a start (ej: T-24h / T-2h) **NEEDS CLARIFICATION**
- recordatorios al business si hay pendiente de aprobación (si aplica)

## Links
- Notifications overview: `docs/00-core-domain/04-bounded-contexts/05.Communication/02.notifications/01.overview.md`
- Reminders: `docs/00-core-domain/04-bounded-contexts/05.Communication/02.notifications/02.reminders.md`

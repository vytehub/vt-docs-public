---
title: Timezones & Place (curated)
description: Reglas de interpretación de fechas/horas al proyectar slots, mostrar UI y crear bookings.
---

# Timezones & Place (curated)

## Principio
VyteMerge gira alrededor de **Time + Place**:
- Timeline representa el tiempo del owner.
- Place define el **timezone de referencia** del evento/servicio.

## Reglas recomendadas (producto)
1. El usuario ve horarios en **su timezone**, pero siempre con indicador (ej: “GMT-3”).
2. La disponibilidad (slots) se proyecta en el timezone del **Place/Timeline** (definir prioridad).
3. El Booking debe persistir:
   - `start/end` con timezone explícito o normalizado
   - el `placeTimezone` para reproducción fiel
4. Nunca inferir timezone por “locale” sin Place.

## Links
- Place timezone: `docs/00-core-domain/04-bounded-contexts/02.Supply - Time & Place/02.places/04.timezone.md`
- Multi-lugar/timezone use case: `docs/02-product-variants/use-cases/multi-lugar-timezone.md`

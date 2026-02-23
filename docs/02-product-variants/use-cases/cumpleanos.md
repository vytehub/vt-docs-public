---
title: Cumpleaños invitaciones + formulario dinámico
---

# Cumpleaños: invitaciones + formulario dinámico

## TL;DR
Invitar a un cumple con una publicación “linda”, recolectar info por formulario y manejar RSVP/capacidad.

## Actores
- Madre (Organizer Profile)
- Invitados (Profiles)
- Niños (dependientes/implícitos)

## Objetivo
Centralizar invitación, RSVP y datos (acompañantes, alergias), con updates simples.

## Contexto
WhatsApp no resuelve confirmaciones ni formularios.

## Flujo principal
1. Crea **Event** “Cumpleaños”.
2. Publica **Post/Invite** con diseño.
3. Adjunta **Form** dinámico.
4. Invitados responden RSVP + form.
5. Organizer ve asistentes y respuestas.

## Requisitos / Reglas
- Form dinámico por evento.
- Attendees con estado + “party size”.
- Capacidad.
- Privacidad de datos sensibles.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Organizer + invitados. |
| **Timeline** | Timeline del organizer; opcional “guardar en agenda”. |
| **Service/Product/Catalog/Listing** | No central (social). |
| **Slot** | No aplica. |
| **Event** | Cumpleaños. |
| **Booking** | No aplica (futuro: entrada/pago). |
| **Attendees** | RSVP + acompañantes. |
| **Order** | No aplica (futuro). |
| **Agreements/Access** | Privacidad del evento/form. |
| **Feed/Chat/Channels** | Post compartible + chat para dudas. |

## KPIs
- tasa RSVP
- completitud de formularios
- ausentismo

## Edge cases
- Invitado sin cuenta: guest link?
- Edición de formulario con respuestas previas.
- Datos sensibles (salud): retención/acceso.

## Preguntas abiertas
- ¿Niños como entidad o metadata?
- ¿Guest attendees en MVP?

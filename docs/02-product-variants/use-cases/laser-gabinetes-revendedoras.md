---
title: Depilación láser - gabinetes alquilados a revendedoras + profesionales independientes
---

# Depilación láser: gabinetes alquilados a revendedoras + profesionales independientes

## TL;DR
Una estética con 3 gabinetes (2 con personal, 1 solo equipo) alquila por hora/día a revendedoras que revenden turnos. Las revendedoras deben coordinar con profesionales propios; profesionales independientes quieren publicar disponibilidad para ser contratados por varias estéticas.

## Actores
- Estética/Local (org Profile)
- Revendedoras (Profiles)
- Profesionales independientes (Profiles)
- Clientas finales (Profiles)

## Objetivo
Monetizar infraestructura (gabinetes/equipos) delegando la relación con clientas a revendedoras y coordinando recursos + profesionales.

## Contexto
La estética no quiere lidiar con clientas finales. Alquila gabinetes. Dos incluyen personal interno; uno requiere profesional externo. Profesionales trabajan con múltiples estéticas.

## Flujo principal
1. La estética define Services B2B: “Gabinete + Personal”, “Gabinete (solo equipo)”.
2. Publica Listings B2B (alquiler por hora/día).
3. Revendedora reserva un bloque (Booking) y revende turnos a clientas con sus propios Listings.
4. En gabinete sin profesional: revendedora coordina con profesional independiente (availability).
5. Profesional publica su disponibilidad para contratación.
6. Agreements + Access separan visibilidad y responsabilidades (la estética no ve clientas de revendedora).

## Requisitos / Reglas
- Alquiler B2B por bloques (hora/día).
- Gabinetes como recursos (timelines) con capacity.
- Diferenciar “con personal” vs “solo equipo”.
- La estética NO debe ver datos sensibles de clientas finales (owner = revendedora).
- Coordinación recurso (gabinete) + profesional (tiempo).
- Agreements partner/reseller con términos y revocación.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Estética + revendedoras + profesionales + clientas. |
| **Timeline** | Timeline por gabinete (recurso) + timeline del profesional (tiempo). |
| **Service** | B2B alquiler gabinete + B2C turnos depilación (revendedora/profesional). |
| **Listing** | B2B alquiler + B2C reventa de turnos. |
| **Slot** | Slots del gabinete y del profesional deben no-conflictar. |
| **Booking** | Booking del bloque (alquiler) + bookings finales (separados). |
| **Event** | Bloques de alquiler + turnos finales (separados por ownership). |
| **Agreements/Access** | Agreement estética↔revendedora + agreement revendedora↔profesional; Access restringe datos. |
| **Campaigns** | Promoción B2B (alquiler) y B2C (turnos) según perfil. |
| **Feed/Chat/Channels** | Feed + chat; canales partners. |

## KPIs
- ocupación de gabinetes (fill rate)
- ingresos por alquiler
- cancelaciones por falta de profesional
- reventa por bloque

## Edge cases
- gabinete libre pero profesional no (o viceversa)
- revendedora cancela tarde y pierde dinero
- profesional reservado en dos lugares
- disputa por daño del equipo
- aislamiento de datos: la estética no debe ver clientas de revendedora

## Preguntas abiertas
- ¿Alquiler B2B es un Service Listing estándar con Booking largo o requiere tipo especial?
- ¿Cómo “encadenamos” turnos dentro del bloque sin exponer datos?
- ¿Sub-bookings dentro del bloque o alcanza con conflict rules + ownership?
- ¿Qué términos mínimos cubren responsabilidad por equipo y no-show?

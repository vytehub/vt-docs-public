---
title: Multi-lugar de trabajo + zonas horarias
---

# Multi-lugar de trabajo + zonas horarias

## TL;DR
Un profesional (ej: médico) mantiene **un único Timeline personal**, pero atiende en **múltiples lugares de trabajo** (hospital, consultorio privado) incluso en **zonas horarias distintas**. Los bookings/events deben quedar bien ubicados en el lugar correcto y sin confusiones de horario.

## Actores
- Profesional (Profile)
- Organización (Profile) (ej: hospital/clinica)
- Pacientes (Profiles)
- Staff (Profiles) (opcional)

## Objetivo
- Reflejar la realidad: una persona trabaja en más de un lugar.
- Evitar confusión de horarios / timezones.
- Permitir listings por lugar (“turnos en hospital” vs “turnos en consultorio”).

## Contexto
El médico tiene su agenda (Timeline). Algunos eventos suceden en el hospital, otros en su consultorio. Puede atender en otra ciudad/país (TZ distinta).

## Flujo principal
1. El profesional crea/declara **Workplaces (Places)**: Hospital, Consultorio.
2. Cada Place define:
   - TimeZoneId
   - dirección / ubicación
   - fotos / metadata
3. El profesional crea **Service Listings** por Place:
   - “Consulta en Consultorio”
   - “Consulta en Hospital”
4. Un paciente reserva desde el listing del lugar correspondiente.
5. Se crea **Booking** + **Event** en el Timeline del profesional con:
   - `PlaceId`
   - horario definido en TZ del Place (fuente de verdad)
6. (Opcional) Staff opera turnos bajo Agreement, sin ser dueño del timeline.

## Requisitos / Reglas de negocio
- El profesional tiene **un solo Timeline**.
- Un **Place no es un Timeline**: es el “dónde ocurre” (metadatos + TZ + reglas).
- El horario de un booking/event se guarda en TZ del Place y se renderiza:
  - hora del lugar
  - (opcional) hora del usuario.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| Profile | Profesional + hospital + pacientes |
| Timeline | Único: el del profesional |
| Place (nuevo) | Hospital/consultorio con TZ + ubicación + media |
| Service | Consulta, controles |
| Listing | Listing por Place |
| Slot | Slots del profesional para ese listing |
| Booking | Booking referencia Place |
| Event | Event en timeline con PlaceId |
| Agreements/Access | Delegación staff y visibilidad |

## Edge cases
- Usuario viaja y cambia TZ: ¿cómo se renderizan futuros eventos?
- Horario de verano (DST).
- Conflictos: dos bookings al mismo horario en lugares distintos.
- Lugar “zona aproximada” (servicios móviles).

## Preguntas abiertas
- ¿Place puede ser “propiedad” de una organización y “referenciado” por profesionales?
- ¿Cómo representamos recursos físicos (consultorio 1, gabinete 2) sin crear “timelines”? (posible: Assets/Resources con agenda propia, separada del Timeline de la persona)

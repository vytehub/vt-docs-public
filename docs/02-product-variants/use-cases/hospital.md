---
title: Hospital - múltiples médicos + reprogramación + reportes
---

# Hospital: múltiples médicos + reprogramación + reportes

## TL;DR
El hospital quiere manejar agendas de médicos, reducir no-shows, reprogramar fácil, optimizar quirófanos y generar reportes de demanda insatisfecha.

## Actores
- Hospital (Profile organización)
- Médicos (Profiles)
- Staff administrativo (Profiles)
- Pacientes (Profiles)

## Objetivo
Menos ausentismo, reprogramación ágil, mejor uso de recursos y reporting para justificar contratación.

## Contexto
Ausentismo alto, comunicación mala. Cirugías se cancelan y se desperdicia quirófano.

## Flujo principal
1. Cada médico mantiene su **Timeline** (owner del tiempo).
2. Hospital firma **Agreements** con médicos y staff.
3. Hospital publica **Service Listings** por especialidad/médico.
4. **Bookings** + reminders + reprogramación.
5. Quirófano como recurso: **shared capacity**.
6. Reporte: intentos/búsquedas sin slots → demanda insatisfecha.

## Requisitos / Reglas
- Delegation: staff opera sin adueñarse de timelines.
- Reprogramación masiva (vacaciones/congresos).
- Cirugías con prioridad y duración larga.
- Privacidad de salud.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Hospital + médicos + pacientes + staff. |
| **Timeline** | Timeline médico + timeline quirófano/recurso. |
| **Service** | Consulta/cirugía/estudios. |
| **Listing** | Listings por médico/especialidad. |
| **Slot** | Slots por médico + recurso. |
| **Event** | Eventos derivados + bloqueos (vacaciones). |
| **Booking** | Turnos/cirugías + reprogramación. |
| **Attendees** | Paciente (acompañante futuro). |
| **Order** | Futuro: depósito/no-show. |
| **Agreements/Access** | Agreements staff↔hospital, hospital↔médicos; Access por privacidad. |
| **Campaigns** | Opcional/regulado. |
| **Feed/Chat/Channels** | Chat paciente↔staff; canales internos. |

## KPIs
- no-shows
- % reprogramaciones exitosas
- ocupación quirófano
- demanda insatisfecha

## Edge cases
- cancelación masiva por médico enfermo
- prioridades complejas
- compliance de salud

## Preguntas abiertas
- ¿Depósito/no-show en MVP?
- ¿Cómo medimos demanda insatisfecha (búsqueda vs intento fallido)?

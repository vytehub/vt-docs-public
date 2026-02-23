---
title: Marketplace de profesionales independientes (contratación ágil)
---

# Marketplace de profesionales independientes (contratación ágil)

## TL;DR
Un profesional independiente quiere **publicar su disponibilidad** para que organizaciones (clínicas/estéticas) lo contraten por hora/día. La organización quiere reservarlo rápido y con acuerdos claros, manteniendo la autonomía del profesional.

## Actores
- Profesional independiente (Profile)
- Organización contratante (Profile)
- Staff (Profiles)
- (Opcional) Cliente final (Profile) si luego se venden turnos B2C

## Objetivo
- Reducir fricción para contratar profesionales.
- Mantener el paradigma social: cada uno con su cuenta.
- Asegurar permisos y responsabilidades vía Agreement.

## Contexto
La organización tiene demanda variable. El profesional trabaja con varias organizaciones y quiere evitar coordinación manual.

## Flujo principal
1. El profesional configura su perfil y su **disponibilidad** (Timeline).
2. Publica un **Listing B2B**: “Disponibilidad profesional”
   - por hora / por bloque / por día
   - lugares aceptados (Places) o zona
3. La organización descubre al profesional (Feed/Buscador/Canal B2B).
4. La organización envía un **Agreement**:
   - scope: servicios/lugares
   - permisos: reservar bloques, ver disponibilidad, operar reprogramación
   - términos: cancelaciones, pagos, responsabilidades
5. Con el acuerdo activo, la organización reserva bloques (Bookings) en el timeline del profesional.
6. (Opcional) La organización crea listings B2C y vende turnos dentro del bloque ya reservado.

## Requisitos / Reglas de negocio
- El profesional mantiene su Timeline y autonomía.
- La organización no “posee” el timeline: opera bajo Agreement.
- Separación conceptual:
  - contratación B2B = reservar tiempo del profesional
  - venta B2C = turnos a clientes
- Privacidad: la organización ve lo necesario, no todo.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| Profile | Profesional + organización |
| Timeline | Timeline del profesional |
| Place (nuevo) | Lugares aceptados |
| Service | “Prestación profesional por hora” (B2B) |
| Listing | Listing B2B del profesional |
| Booking | Booking B2B por la organización |
| Event | Bloque reservado en el timeline |
| Agreements/Access | Agreement profesional↔organización |
| Channels | Canal B2B (futuro) |
| Order | Pago B2B (futuro u off-platform inicialmente) |

## Edge cases
- Doble contratación en el mismo horario.
- Cancelación tardía: penalizaciones por términos.
- Profesional móvil: requiere zona y buffers.
- Organización reserva bloques y no los usa (no-show B2B).

## Preguntas abiertas
- ¿El pago B2B va on-platform desde el inicio o se empieza “coordinando”?
- ¿Cómo auditamos “quién bloqueó qué” en el timeline del profesional?

---
title: Veterinario - consultorio + walk-in + cirugías + domicilios
---

# Veterinario: consultorio + walk-in + cirugías + domicilios

## TL;DR
Un veterinario quiere ordenar su agenda, mezclar **demanda espontánea (walk-in)** con **turnos**, reservar **cirugías** en días fijos y hacer **visitas domiciliarias** mejor pagas; además quiere impulsar demanda con promos y vender productos relacionados.

## Actores
- Veterinario (Profile)
- Clientes (dueños de mascotas) (Profiles)
- (Futuro) Asistente/Staff

## Objetivo
Optimizar agenda y aumentar pacientes: facilitar reservas, absorber walk-ins, planificar cirugías y vender/bonificar productos.

## Contexto
Atiende en consultorio, cirugías Mar/Jue y emergencias. Domicilios por margen. Quiere campañas y promos (ej: bolsa de alimento por X atenciones).

## Flujo principal
1. Crea **Services**: Consulta, Cirugía, Domicilio.
2. Crea **Listings**:
   - Consulta (reservable) + modo walk-in.
   - Cirugía (reservable, restringida a Mar/Jue).
   - Domicilio (reservable, requiere zona).
3. Publica en **Feed** y activa **Campaigns**.
4. Booking genera **Event** y bloquea **Timeline**.
5. Promo: venta/bonificación de **Product** asociada a la atención.

## Requisitos / Reglas
- Soportar **walk-in** además de appointment.
- Cirugías con prioridad alta; emergencia puede desplazar agenda.
- Domicilios: zona + (posible) buffer de traslado.
- Promos: producto gratis o descuento por frecuencia.
- Chat para coordinar.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Veterinario como Profile (persona). |
| **Timeline** | Timeline del veterinario; opcional timeline “consultorio” como recurso. |
| **Service** | Consulta / Cirugía / Domicilio (duración base). |
| **Product** | Bolsa de alimento / insumos asociados. |
| **Catalog** | Services + Products del veterinario. |
| **Listing** | Service Listings + Product Listing (si vende alimento). |
| **Slot** | Slots para consultas; cirugías solo en ventanas Mar/Jue. |
| **Event** | Eventos de turnos + eventos de emergencia. |
| **Booking** | Bookings normales + walk-in (ver preguntas). |
| **Attendees** | Normalmente 1:1. |
| **Order** | Compra/bonificación de producto (si aplica). |
| **Agreements/Access** | Delegación a asistente (futuro). |
| **Campaigns** | Promoción local para captar demanda. |
| **Feed/Chat/Channels** | Feed para discovery; chat para coordinación. |

## KPIs
- conversión feed→booking
- ocupación agenda
- no-shows
- ventas de productos por booking

## Edge cases
- Emergencia: ¿reprograma automáticamente o manual?
- Walk-in: ¿cómo se bloquea agenda cuando llega alguien sin booking?
- Domicilio: travel time aún no está modelado explícitamente.
- Promo gratis: ¿Order $0 o descuento?

## Preguntas abiertas
- ¿Walk-in se modela como Booking instantáneo creado por el veterinario?
- ¿Travel buffers entran al Core Model o módulo futuro?
- ¿Cómo definimos prioridades (emergencia vs reservas existentes)?

---
title: Incentivos anti no-show (devolución por compromiso)
---

# Incentivos anti no-show (devolución por compromiso)

## TL;DR
Para servicios con alta demanda y no-shows, VyteMerge ofrece un mecanismo opcional: si el usuario reserva con mucha anticipación y **asiste**, recibe una **devolución** (idealmente crédito interno en MVP). Si no asiste, pierde el monto comprometido. Una parte se destina al proveedor, otra a comisión y otra a un pool que financia devoluciones.

## Actores
- Cliente (Profile)
- Proveedor/Organización (Profile)
- VyteMerge (reglas del programa)

## Objetivo
- Incentivar reservas anticipadas.
- Reducir no-shows.
- Crear un mecanismo simple y transparente.

## Contexto
Servicios muy demandados suelen tener turnos a meses. Un incentivo ayuda a reservar antes y cumplir, evitando huecos.

## Flujo principal (MVP)
1. Un Listing habilita **Modo Compromiso** (opt-in).
2. El cliente hace Booking y paga un **monto comprometido** (stake/deposito del programa).
3. Regla: “si asistís, devolvemos X%”.
4. Asistencia MVP: **confirmación del proveedor** (simple).
5. Si asistió:
   - se acredita “Devolución” (recomendado MVP: crédito interno)
6. Si no asistió:
   - el monto se distribuye: proveedor + comisión + pool.

## Requisitos / Reglas de negocio
- Opt-in por listing.
- MVP: devolución como **crédito interno** (“Vyte Credits”).
- Ventanas claras:
  - cancelación sin penalidad
  - definición de no-show
- Mecanismo de asistencia simple y auditable.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| Listing | Política “Modo Compromiso” |
| Booking | Booking con compromiso asociado |
| Event | Evento a asistir |
| Order | Cobro del compromiso y/o del servicio |
| AttendanceProof (nuevo) | Confirmación de asistencia (MVP: proveedor) |
| RewardPool (nuevo) | Pool que financia devoluciones |
| Policies | Cancelación, no-show, devoluciones |

## Opciones de “devolución” (para decidir luego)
1. **Crédito interno** (recomendado MVP): simple, reutilizable, menos fricción.
2. Cashback real: más complejo.
3. Descuento futuro: intermedio.

## Edge cases
- Disputa asistencia (cliente vs proveedor).
- Cancelación por parte del proveedor: devolución total + compensación?
- Reprogramación: ¿se mantiene el compromiso?
- Abuso: proveedores marcando no-show de forma incorrecta.

## Preguntas abiertas
- ¿El compromiso es obligatorio para acceder a cierto precio/descuento?
- ¿Qué porcentaje va a pool vs proveedor vs comisión?
- ¿Cómo escalar “proof” sin invadir privacidad?

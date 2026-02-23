---
title: Plomero - agenda flexible + depósito/seguro + mediación
---

# Plomero: agenda flexible + depósito/seguro + mediación

## TL;DR
Servicios a domicilio con baja confiabilidad: depósito/seguro y reglas para reducir no-shows, más mediación ante disputas.

## Actores
- Plomero (Profile)
- Cliente (Profile)
- Plataforma (reglas/mediación)

## Objetivo
Crear confianza: reserva con depósito, penalización por incumplimiento y coordinación simple.

## Contexto
Clientes frustrados por “no apareció”; plomeros por cancelaciones tardías.

## Flujo principal
1. Listing “Visita diagnóstico” (precio fijo).
2. Booking (request-to-book o confirmación manual).
3. Cobro de depósito/seguro.
4. Si se concreta: descuento del diagnóstico.
5. Si falla: penalización.
6. (Futuro) disputa: mediación.

## Requisitos / Reglas
- Request-to-book / confirmación manual.
- Depósito/escrow y refunds claros.
- Zona y ventana horaria.
- Chat.

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Plomero + cliente. |
| **Timeline** | Timeline del plomero. |
| **Service** | Diagnóstico/reparación. |
| **Listing** | Listing con políticas de depósito. |
| **Slot** | Slots o ventanas. |
| **Event** | Visita a domicilio. |
| **Booking** | Con confirmación manual + depósito. |
| **Order** | Depósito/pagos (futuro). |
| **Campaigns** | Promoción local. |
| **Feed/Chat/Channels** | Feed + chat. |

## KPIs
- no-shows
- cancelaciones tardías
- conversión booking→trabajo

## Edge cases
- abuso en disputas
- travel time
- tolerancias por retraso

## Preguntas abiertas
- ¿Depósito = Order separado o parte del booking?
- ¿Mediación entra al MVP o módulo posterior?

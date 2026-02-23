---
title: Estética nails - skills + auto-asignación + duraciones
---

# Estética nails: skills + auto-asignación + duraciones

## TL;DR
3 servicios, 3 empleadas con skills distintos y duraciones distintas. Se necesita auto-asignación al staff compatible disponible.

## Actores
- Estética (org)
- Empleadas (Profiles)
- Clientes (Profiles)

## Objetivo
Maximizar ocupación sin dobles reservas y respetar skills.

## Flujo principal
1. Services: capping(90), manicura(60), gel(30).
2. Listings por servicio.
3. Skills por empleada.
4. Booking dispara auto-asignación.
5. Event aparece en timeline de la empleada asignada.

## Requisitos / Reglas
- matching por skills
- auto-asignación (simple: round-robin + disponibilidad)
- reasignación si falta staff
- shared capacity si hay sillas/equipos limitados

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Estética + empleadas + clientes. |
| **Timeline** | Timeline por empleada. |
| **Service** | Capping/Manicura/Gel. |
| **Listing** | Service Listings. |
| **Slot** | Slots derivados por staff. |
| **Booking** | 1:1. |
| **Event** | Turno asignado. |
| **Agreements/Access** | Delegación staff; control de visibilidad. |
| **Campaigns** | Promoción de listings. |
| **Feed/Chat** | Discovery + coordinación. |

## KPIs
- ocupación staff
- cancelaciones
- conversión listing→booking

## Edge cases
- preferencia por empleada
- ausencia y reasignación
- simultaneidad

## Preguntas abiertas
- ¿Skills como tags o catálogo formal?
- ¿Auto-asignación vive en booking o listing?

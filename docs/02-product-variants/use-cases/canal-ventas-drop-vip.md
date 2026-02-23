---
title: Canal de ventas - Drop VIP + ofertas segmentadas
---

# Canal de ventas: Drop VIP + ofertas segmentadas

## TL;DR
Una marca/estética quiere vender servicios y productos mediante un **canal VIP** (clientes frecuentes) con **early access**, **promos exclusivas** y publicaciones que solo ven miembros del canal. El feed muestra ofertas del canal a sus miembros y permite reservar/comprar en 1–2 pasos.

## Actores
- Marca / Estética / Seller (Profile)
- Clientes VIP (Profiles)
- (Opcional) Staff/Operator (Profiles)

## Objetivo
Aumentar conversión y recurrencia creando un “club”:
- vender más rápido (drops)
- dar beneficios por pertenencia (promos por canal)
- mantener comunicación directa y segmentada

## Contexto
El seller hoy anuncia promociones por Instagram/WhatsApp, pero:
- la audiencia no está bien segmentada
- las ofertas se pierden
- no hay trazabilidad ni métricas por canal

## Flujo principal
1. El seller crea un **Channel**: “VIP Club”.
2. Define una regla de ingreso (manual o automática, futuro):
   - “solo clientes con 3 compras/booking”
   - o invitación directa
3. Publica en el canal:
   - **Posts** (contenido)
   - **Listings** (Service Listings y Product Listings)
4. Los miembros ven el contenido en su feed/canal:
   - pueden **Reservar** (service) o **Comprar** (product)
5. El seller puede lanzar un **Drop**:
   - early access 24h solo para VIP
   - cupos limitados (capacity/minToConfirm cuando aplique)
6. (Opcional) Se activa **Campaign** pero segmentada al canal (futuro: “solo miembros”).

## Requisitos / Reglas de negocio
- Un **canal define audiencia** (quién lo ve).
- Un Listing puede publicarse con visibilidad:
  - Public
  - Followers
  - **Channel**
- **Promos por canal**:
  - precio especial
  - bundle/beneficio (futuro)
  - ventana temporal (ej: 24h)
- El canal debe soportar:
  - posts (contenido)
  - listings (venta)
  - chat (preguntas y soporte)
- Moderación básica (spam/abuso).

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Seller + miembros del canal. |
| **Timeline** | No es el centro, pero afecta a Service Listings (slots/booking). |
| **Service** | Ej: “Masaje”, “Depilación”, “Consulta”. |
| **Product** | Ej: “Kit cuidado”, “Crema”, “Merch”. |
| **Catalog** | Services + Products del seller. |
| **Listing** | Publicado con visibilidad “Channel”; puede tener promo exclusiva. |
| **Slot** | Para Service Listings: disponibilidad proyectada. |
| **Event** | Se crea por Booking (turno), o post informativo (no-event). |
| **Booking** | Reservas desde el canal (ej: cupos limitados). |
| **Attendees** | Si es un evento grupal (ej: clase), capacity + minToConfirm. |
| **Order** | Compras de productos desde el canal. |
| **Agreements/Access** | Access restringe contenido a miembros del canal; staff puede administrar el canal. |
| **Campaigns** | Promoción (futuro: targeting al canal y lookalikes). |
| **Feed/Chat/Channels** | El canal es la superficie principal: feed segmentado + chat. |

## KPIs
- conversion canal → booking/order
- % miembros activos (views, clicks)
- retención (repeat purchases)
- ingresos atribuibles al canal
- tasa de respuesta del chat (tiempo promedio)

## Edge cases (casos límite)
- Un usuario sale del canal: ¿pierde acceso a promociones anteriores y a históricos?
- Un listing publicado en canal se comparte por link: ¿qué ve un no-miembro?
- Reventa/partners: ¿pueden existir canales “B2B Partners” separados del canal VIP?
- Promos conflictivas: un usuario ve dos precios (público vs canal).
- Moderación: spam en chat, reporte, expulsión.

## Preguntas abiertas
- ¿Los canales soportan reglas automáticas de entrada (por comportamiento) o solo invitación en MVP?
- ¿Se permite “pagar membresía” (subscription) para acceder al canal? (futuro)
- ¿Campaigns pueden targetear canales directamente en MVP o solo feed general?

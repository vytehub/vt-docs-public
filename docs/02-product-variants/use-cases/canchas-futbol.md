---
title: Canchas de Fútbol
description: Reservas privadas con invitados, partidos públicos con cupos, MinToConfirm y split payments futuro.
---

# Use Case: Canchas de Fútbol

## TL;DR
Una cancha puede funcionar de dos maneras en VyteMerge:

1) **Reserva privada**: una persona reserva y paga, e invita a otros jugadores (attendees).  
2) **Partido público**: cualquiera puede sumarse hasta completar cupos; se confirma solo si se alcanza un mínimo (`MinToConfirm`).

Esto se resuelve con:
- **Service**: “Alquiler cancha fútbol 5”
- **Listing**: cómo se vende (privado / público, precio, reglas)
- **Booking**: la reserva
- **Attendees**: invitados/participantes
- **Capacity / MinToConfirm**: cupos y umbral mínimo para confirmar

---

## Contexto
Un club tiene **3 canchas**. Su mayor flujo es fines de semana, pero quiere:
- facilitar que la gente organice partidos (siempre falta alguien)
- ofrecer una promo “**partido de desconocidos**” (open booking) los lunes para llenar horarios muertos

Además, jugadores quieren:
- invitar a amigos y que quede agendado
- evitar que el partido se caiga por ausencias
- (futuro) dividir la cuenta automáticamente

---

## Actores
- **Club (Profile/Org)**: dueño de las canchas.
- **Organizer (Profile/Person)**: reserva privada y arma el partido.
- **Players (Profiles)**: invitados o participantes.
- **Strangers (Profiles)**: participantes que se suman en un partido público.
- (Opcional) **Operator/Staff**: gestiona administración (por Agreements).

---

## Objetivos del negocio
- llenar horarios de baja demanda
- reducir “no-shows”
- aumentar visibilidad del club (feed + campaigns)
- habilitar comunidad recurrente (channels)

---

## Modelo (mapping a Core Model)

### Service
- **Service**: `Alquiler Cancha Fútbol 5`
  - duración base: 60/90 min (según club)
  - descripción / reglas generales

### Place
- **Places**: `Cancha 1`, `Cancha 2`, `Cancha 3`
  - cada una puede ser un Place o un “asset” futuro
  - define zona horaria (TimeZoneId)

### Timeline
- El club tiene un **Timeline por recurso** (recomendado):
  - Timeline “Cancha 1”
  - Timeline “Cancha 2”
  - Timeline “Cancha 3”
  
> Alternativa: un timeline único + resources/availability avanzada (futuro). Para MVP, timeline por cancha es más simple.

### Listing (2 modalidades)
1) **Listing Privado (Private Booking)**
   - tipo: `Service Listing`
   - modo: **Private**
   - capacity: 1 (la reserva es una sola, aunque haya 10 players)
   - attendees: invitables
   - precio: estándar

2) **Listing Público (Open Booking)**
   - tipo: `Service Listing`
   - modo: **Open**
   - **Capacity**: 10 jugadores
   - **MinToConfirm**: 10 (o 8 si querés permitir jugar incompleto)
   - precio: promocional (más barato)
   - política: si no llega al mínimo, se cancela o queda “no confirmado” (según regla)

### Booking & Attendees
- **Booking**: bloquea el slot en el timeline de la cancha
- **Attendees**:
  - private: organizer invita a 9
  - open: jugadores se suman hasta capacity

---

## Flujos

### A) Reserva privada con invitados
1. Organizer elige fecha/hora (slot)
2. Hace **Booking** (paga él)
3. Invita 9 jugadores (attendees: Invited)
4. Los jugadores aceptan/declinan
5. El evento aparece en el timeline del organizer y puede aparecer como invitación en la agenda de cada jugador

**Reglas**
- Si un invitado declina: no afecta al booking (la cancha sigue reservada)
- Organizer puede reemplazar jugadores hasta X horas antes (política)

---

### B) Partido público con desconocidos (Open booking)
1. Club publica listing “Partido público lunes 21:00”
2. Los usuarios lo ven en feed / búsqueda / canal
3. Se suman como attendees (Joined) hasta `Capacity`
4. Si se alcanza `MinToConfirm` antes de un deadline:
   - estado: Confirmed
   - se crea Event asociado
5. Si NO se alcanza:
   - Cancelled (y se liberan slots) **o**
   - queda “No confirmado” (futuro)

**Reglas recomendadas MVP**
- Deadline de confirmación: por ej. 12 horas antes
- Si no llega al mínimo: cancelar automáticamente y notificar

---

## Pagos (MVP y futuro)

### MVP
- Private: paga el organizer
- Open: puede pagar el organizer (si existe) o pago individual (si lo implementás)

### Futuro (Split payments)
- cada attendee paga su parte
- si no paga antes del deadline, pierde su lugar
- el sistema reemplaza con waitlist

---

## Edge cases / Casos límite
- **Overbooking**: dos usuarios intentan reservar el mismo slot → revalidación al confirmar booking.
- **Organizer cancela**: se notifica a invitados; si hay split payments futuro → refunds.
- **MinToConfirm alcanzado y luego bajas**: definir política:
  - “una vez confirmado, no se cae” (recomendado) o
  - “si cae por debajo del mínimo, se cancela”
- **No-show de jugadores**: no afecta a la cancha (si se pagó), pero impacta reputación (futuro).
- **Bloqueos sociales**: si hay usuarios bloqueados, evitar que coincidan en un open booking (futuro).
- **Clima / cancelación por el club**: reprogramación o refund; notificación masiva.

---

## Canales de venta (opcional para este caso)
- Canal “Comunidad Fútbol 5 – Club X”
- Publicación automática de próximos partidos públicos
- Anuncios del club (promos, torneos)

---

## KPIs (de producto)
- tasa de ocupación por cancha / franja horaria
- % partidos públicos confirmados vs cancelados (MinToConfirm)
- tiempo promedio para completar cupos
- conversión feed → booking / join
- recurrencia: jugadores que vuelven al mismo club

---

## Gaps Checklist (resultado)
- Timeline: ✅ (por cancha)
- Place/Timezone: ✅ (cancha como lugar)
- Listing: ✅ (private/open + promos)
- Booking/Attendees: ✅ (capacity + minToConfirm)
- Order/Payments: ⚠️ (split payments futuro)
- Agreements/Access: ✅ (staff/operadores opcional)
- Notificaciones: ✅ (confirmación/cancelación)
- Privacidad/Consent: ✅ (public vs privado)
- Moderación/Disputas: ⚠️ (conflictos entre desconocidos futuro)
- Analytics/Attribution: ✅ (campañas/canales futuro)

---

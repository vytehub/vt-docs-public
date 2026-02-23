---
title: Core Model
description: Modelo mental central de VyteMerge (Profile, Timeline, Place, Catalog, Service, Product, Listing, Slot, Event, Booking, Attendees, Orders, Agreements, Access, Campaigns, Feed, Chat, Channels).
---

# Core Model (VyteMerge)

## TL;DR
VyteMerge se entiende con estas 3 reglas:

> **Service/Product** define *qué* se ofrece (definición reusable).  
> **Timeline** define *cuándo* (de quién es el tiempo / agenda).  
> **Listing (Listado)** define *cómo se vende/ofrece* algo del Catálogo.

Y una regla extra clave para que el mundo real cierre:

> **Place (Lugar)** define *dónde* ocurre (ubicación, contexto y zona horaria).

Sobre eso se apoyan:
- **Slots** (disponibilidad proyectada)
- **Events** (cosas que pasan en el tiempo)
- **Bookings** (reservas/confirmaciones)
- **Attendees** (quiénes participan)
- **Orders** (compras / pagos)
- **Agreements + Access** (relaciones y permisos/visibilidad)
- **Campaigns** (autopromoción pagada de listings)
- **Feed + Chat + Channels** (superficies de interacción)

> Nota: Casos específicos (vouchers, canchas públicas, marketplace B2B, etc.) viven en *Use Cases*, no en este documento.

---

## Objetivo
Este documento define el **modelo mental** común para producto/UX/dev.
No describe implementación, endpoints ni detalles técnicos.

---

## 1) Identidad social

### Profile
Identidad social en VyteMerge: persona u organización.

- Persona: médico, docente, creador, profesional.
- Organización: clínica, colegio, club, estética.

Un Profile puede:
- publicar y socializar
- vender (Services/Products vía Listings)
- operar Timelines y Places
- firmar Agreements con otros Profiles

---

## 2) Tiempo y agenda

### Timeline
Representa **tiempo** (agenda) y su ocupación.

Regla clave:
> El owner de un Timeline es quien pone el tiempo (persona) o controla un recurso/agenda (organización).

Ejemplos:
- Dr. X (persona) es owner de su agenda.
- Colegio (org) es owner del timeline “Salita 3B”.
- Club (org) es owner del timeline “Cancha 1” (si modelás la cancha como agenda propia).

En un Timeline viven:
- Events (manuales o derivados)
- Bookings (que bloquean tiempo)
- **Conflict Rules** (qué cuenta como conflicto y qué acción tomar)

---

## 3) Lugares (dónde ocurre)

### Place (Lugar / Workplace)
Un **Place** representa un lugar de trabajo o ubicación donde ocurren eventos y servicios.

Incluye típicamente:
- nombre, descripción, fotos/media
- dirección / ubicación (o “zona”)
- **TimeZoneId** (muy importante)
- (opcional) metadata operativa (horarios del lugar, instrucciones, etc.)

Regla clave:
> **Place no es un Timeline.**  
> Un médico puede tener **un único Timeline** y **múltiples Places** (hospital + consultorio).

#### ¿Cómo se usa Place?
- Un **Listing** puede estar asociado a un Place (“consulta en consultorio”).
- Un **Booking/Event** referencia `PlaceId` para saber dónde ocurre.
- El horario real se interpreta con la **zona horaria del Place** (fuente de verdad del “horario local del lugar”).

Regla de zona horaria (propuesta):
- Si un Booking/Event tiene `PlaceId`, la fuente de verdad del “horario local” es `Place.TimeZoneId`.
- Si no hay Place, usar `Timeline.TimeZoneId` como fallback (si existe).
- Los usuarios visualizan en su timezone personal, pero el “horario del evento” se ancla al lugar/timeline.

> Extensión futura (no requerida ahora): “Assets/Resources” dentro de un Place (gabinete 1, cancha 2, quirófano), cuando necesites capacidad física o agenda del recurso.

---

## 4) Catálogo: qué se vende

### Service
Define **qué es** un servicio, de forma reusable y estable.

Incluye típicamente:
- nombre, descripción, categoría/tags
- **duración base** (lo “normal”)
- (opcional) términos del servicio

Ejemplos: “Masaje”, “Consulta”, “Corte de pelo”, “Alquiler cancha”.

**Service no vende y no agenda**: es el “qué”.

---

### Product (Producto)
Define un bien físico o digital que se puede vender.

Incluye típicamente:
- nombre, descripción, categoría/tags
- precio base (y luego promos)
- variantes (talle/color/pack), si aplica
- (opcional) inventario/stock y logística (futuro)

Ejemplos: “Shampoo”, “Kit uñas”, “Crema facial”, “Merch”.

> Product es el “qué” para bienes.  
> Se vende mediante un **Listing** igual que un Service.

---

### Catalog (Catálogo)
El Catálogo es el conjunto de “cosas vendibles” que un Profile ofrece:
- **Services**
- **Products**

El Catálogo permite:
- reutilizar definiciones (un mismo Service puede tener múltiples Listings)
- ordenar y buscar por categorías/tags
- habilitar *upsells* (productos asociados a un service) en el futuro

---

## 5) Cómo se vende: Listing

### Listing (Listado)
Define **cómo se ofrece/vende** algo del Catálogo (un Service o un Product).

Un Listing normalmente:
- referencia a un **Service** o **Product**
- define reglas comerciales: precio, promos, políticas, canal (público/privado/partner)
- define reglas de presentación: fotos, descripción, highlights
- define reglas de distribución: elegible para **Campaigns** (promoción)
- (opcional) define **Place** (dónde se presta / se retira / ocurre)

Tipos comunes:
- **Service Listing** (reservable): usa un **Timeline** como fuente de disponibilidad.
- **Product Listing** (comprable): puede incluir stock/inventario (si aplica).
- (futuro) **Bundle/Pack**: Service + Product (ej: “corte + shampoo”) o packs.

En **Service Listings**, además:
- define reglas de scheduling (duración efectiva, buffers, ventana, etc.)
- define **Capacity**, **MinToConfirm** y **Attendees** cuando corresponda
- define si el booking es **Private** u **Open**
- define si requiere **confirmación manual** (futuro)

**Listing es el objeto “promocionable”** en el marketplace/feed.

Regla clave:
> Un Listing **usa** un Timeline como fuente de disponibilidad, pero **no posee** el tiempo.  
> El tiempo (y sus conflictos) siempre pertenece al Timeline y su owner.

> Diferencia útil: un Post es social; un Listing es comercial (reservable o comprable).

---

## 6) Disponibilidad proyectada

### Slot
Un **Slot** es un “hueco reservable” *proyectado*.

> Slot = proyección de disponibilidad a partir de Timeline + reglas del Listing + conflictos.

Un Slot:
- aparece en búsquedas/booking UI
- puede caducar/cambiar con el tiempo
- no es la fuente de verdad (la fuente es Timeline + reglas + eventos)

---

## 7) Qué pasa en el tiempo: Events

### Event
Algo que ocurre en un Timeline (manual o derivado).

- **Manual / no comercial**: acto escolar, partido, viaje, reunión.
- **Derivado / comercial**: turno/cita generado por un Booking.

Un Event puede:
- existir sin Booking
- estar asociado a un Service (para semántica)
- estar asociado a un Booking (si nace de una reserva)
- estar asociado a un Place (dónde ocurre)

---

## 8) Reservas: Bookings

### Booking
Representa una **reserva/confirmación** de un Listing.
- bloquea tiempo en un Timeline
- suele generar o asociarse a un Event
- puede referenciar un Place
- tiene estado (pendiente/confirmado/cancelado)
- tiene responsable (quien reserva/paga)

Regla clave (consistencia):
> Un Event puede existir sin Booking, pero un Booking **confirmado** no debería existir sin un Event asociado en el Timeline target.  
> (Para *pending*, puede no haber Event o haber un Event tentativo, según política.)

En VyteMerge, un Booking puede ser:
- **Private**: el responsable invita a otros (ej: “reservé la cancha e invité a 9”)
- **Open**: cualquiera puede sumarse hasta completar cupos (ej: “partido público”)

Por eso, un Booking puede incluir:
- **Capacity**: cupos máximos (ej: 10 jugadores)
- **MinToConfirm**: mínimo para confirmar (ej: 8 jugadores); si no se alcanza, puede cancelarse o quedar sin confirmar (según política)
- **Attendees**: personas que participan (ver concepto siguiente)

---

### Attendees (Asistentes)
**Attendees** representa quiénes participan en un Event/Booking, aunque **no** sean quienes pagaron o reservaron.

Cada attendee típicamente tiene:
- rol (Organizer / Participant)
- estado (Invited / Joined / Declined / Cancelled / Waitlist)

Esto permite:
- reservas privadas con invitados (1 Booking, N asistentes)
- bookings públicos con desconocidos (N asistentes hasta `Capacity`, confirmación con `MinToConfirm`)

---

## 9) Compras, pagos e incentivos

### Order (Orden de compra)
Representa una compra y/o el compromiso económico asociado a un Listing.
- contiene uno o más ítems (line items)
- registra total, descuentos/promos y estado (pago/fulfillment en el futuro)

Regla simple (ajustada):
- **Booking** representa el compromiso de tiempo (Service Listings).
- **Order** representa el compromiso económico (pagos) para Product Listings **y** para Service Listings cuando hay pago/deposito/prepago.
- Un Booking puede tener `OrderId` (opcional) cuando hay pago asociado.

---

### Devolución por Compromiso (concepto básico)
Para reducir no-shows y fomentar reservas anticipadas, un Listing puede habilitar un modo opcional:

- el usuario reserva con anticipación
- paga un **monto comprometido**
- si asiste → recibe una **devolución**
- si no asiste → pierde el monto (según política)

**MVP recomendado (por simplicidad):**
- devolución como **crédito interno**
- asistencia confirmada por el proveedor

> Este mecanismo vive como “política del Listing” y se ejecuta vía Booking + Order.

---

## 10) Conflict Rules (en lugar de “availability rules”)

En VyteMerge, la clave no es solo “cuándo estoy disponible”, sino:

> **qué considero un conflicto** y **qué hago cuando ocurre**.

### Qué es un conflicto
Un conflicto es una condición que hace que:
- un Slot no sea válido, o
- un Booking requiera revisión, o
- un Event deba bloquear a otro.

Ejemplos:
- “Si tengo un evento ‘Acto Escolar’, no quiero crear slots ese día.”
- “Si tengo ‘Consulta’, no permitir otro booking en la misma franja.”
- “Si tengo un evento ‘Viaje’, bloquear toda la semana.”
- “Si el evento es ‘Personal’, bloquear disponibilidad pública.”

### Dónde viven las reglas
- Las **Conflict Rules** se configuran en el Timeline (porque afectan el tiempo).
- Los Listings pueden traer restricciones (buffers, ventanas), pero la decisión “conflicto” vive en Timeline.

### Qué producen (salidas)
- Bloqueo de slots (no mostrar / no permitir reservar)
- Degradación de visibilidad (mostrar “ocupado”)
- Priorización (qué gana si chocan cosas)
- Requerir confirmación manual (futuro)

---

## 11) Capacity y Shared Capacity

### Capacity (por Listing)
La capacidad (cupos) depende de “cómo lo vendo”, no del Service.
Por eso:

> **Capacity vive en Listing** (ej: 1:1, grupo de 10, pareja de 2).

### Shared Capacity (pool compartido)
A veces varios Listings consumen el mismo “inventario”:
- la misma persona (un profesional) no puede atender dos cosas a la vez
- un consultorio/sala/cancha puede ser un recurso físico compartido
- un equipo comparte cupos

Esto se modela como un **Capacity Pool** (SharedCapacityGroup):
- varios Listings apuntan al mismo pool
- si el pool está al límite, el slot entra en conflicto/no disponible

Orden mental de validación de disponibilidad (propuesta):
1) Conflict Rules del Timeline (eventos/ocupación/visibilidad)
2) Reglas del Listing (buffers, ventanas, duración efectiva)
3) Capacity (por Listing) y Shared Capacity Pool (si aplica)
4) Políticas (manual confirm, cancelación, minToConfirm)

---

## 12) Assignment (auto-asignación)
VyteMerge puede asignar automáticamente Events/Bookings a responsables, sin que el staff cree Listings.

Ejemplo (estética nails, 3 empleadas):
- el negocio vende el Listing
- al crearse un Booking, se aplica una regla de auto-asignación
- el Event aparece asignado en el Timeline de la empleada

Esto habilita:
- operación diaria sin dar control comercial a staff
- trazabilidad de quién atiende qué

Regla (a decidir y documentar):
- **Opción A (recomendada):** Booking se crea contra el Timeline del negocio/recurso y el Event se asigna (o se espeja) al Timeline del staff.
- **Opción B:** Booking se crea directamente contra el Timeline del staff (el business actúa como broker).

---

## 13) Agreements y Access (relaciones + enforcement)

### Agreements (visión general)
Relaciones formales entre Profiles para:
- **Sharing** (compartir visibilidad)
- **Delegation** (operación/staff)
- **Partner/Reseller** (relación comercial)

Un Agreement define:
- scope (Profile / Timeline / Listing / etc.)
- permissions
- terms (Platform Terms + Custom Terms)
- revocación inmediata

### Access (visión general)
Access aplica enforcement de:
- visibilidad (qué se puede ver)
- permisos (qué se puede hacer)

Concepto clave:
- **VisibilityMode** (None / BusyOnly / BusyOnlyDetails) para controlar cuánto detalle se revela.

---

## 14) Campaigns y Publicidad

### Campaigns (concepto básico)
**Campaigns** permiten autopromoción dentro de VyteMerge (ingreso por publicidad).

Idea central:
- una Campaign **promociona uno o más Listings** (no Slots).
- el anuncio puede mostrar “próxima disponibilidad” como metadata (derivada de Slots).

Una Campaign típicamente define:
- objetivo (más bookings / más compras)
- presupuesto y fechas
- audiencia/segmentación (básica al inicio)
- creatividades (texto/imagen)
- métricas (impresiones, clicks, conversiones)

#### Licitaciones (bidding) para promoción
Las Campaigns pueden operar con subasta:
- el seller define presupuesto y puja por audiencia/posición
- el sistema decide ranking según puja + relevancia/calidad (futuro)

### Recomendaciones contextuales (idea / futuro)
VyteMerge podría ofrecer recomendaciones “de pasada” (ej: ofertas cercanas a un lugar cuando vas a un turno), **solo si el usuario lo permite**.

Principios (para documentar sin comprometer implementación):
- requiere **consentimiento explícito** (opt-in)
- minimización de datos (no revelar detalles sensibles)
- “por qué veo esto” (explicabilidad)
- revocable en cualquier momento

> Esta sección queda planteada para análisis posterior.

---

## 15) Feed, Chat, Channels e Integraciones

### Feed (scroll principal)
VyteMerge necesita un **feed** donde conviven diferentes “cards”:
- **Posts** (social)
- **Listings** (comercial: reservar o comprar)
- (opcional) **Events** destacados
- **Sponsored Listings** (Campaigns)

Objetivo del feed:
- descubrimiento rápido
- acciones simples en 1–2 pasos: Reservar, Comprar, Guardar, Seguir, Escribir

### Chat (mensajería)
El Chat habilita:
- coordinación previa a una reserva
- seguimiento post-venta/servicio
- coordinación entre attendees (ej: partido público)

### Grupos o Canales
Canales/grupos permiten:
- segmentar audiencias (ej: “Padres Salita 3B”, “Comunidad fútbol 5”, “Clientes VIP”)
- publicar posts/listings a una audiencia específica
- aplicar beneficios/promos por canal (futuro)

### Integraciones (ej: Instagram)
Integración típica (por etapas):
1) Importar media (fotos/videos) para Listings/Posts.
2) Linking: links de reserva/compra para compartir afuera.
3) (futuro) Sincronización de catálogo/publicaciones según permisos/limitaciones de la plataforma.

> Importante: no asumir capacidades externas; diseñar integración por capas.

---

## Resumen del modelo
- **Profile**: identidad social
- **Timeline**: tiempo y conflictos
- **Place**: dónde ocurre (ubicación + zona horaria)
- **Service**: qué es (servicio)
- **Product**: qué es (producto)
- **Catalog**: conjunto de Services y Products
- **Listing**: cómo se ofrece/vende (reservable o comprable; promocionable; puede depender de Place)
- **Slot**: proyección reservable
- **Event**: ocurre en Timeline (y puede tener Place)
- **Booking**: reserva/confirmación (Private/Open)
- **Attendees**: participantes (invitar/unirse, cupos, MinToConfirm)
- **Order**: compras/pagos (y futuros combos)
- **Devolución por Compromiso**: incentivo opcional ligado a Booking+Order
- **Agreements**: relación formal (sharing/delegation/partner)
- **Access**: enforcement de permisos/visibilidad
- **Campaigns**: promoción pagada de Listings (con bidding)
- **Feed**: descubrimiento (posts + listings + sponsored)
- **Chat**: coordinación y soporte
- **Channels**: audiencias segmentadas


## VyteMerge

> Placeholder (Jarvis): este término aparece 35 veces en módulos pero no está definido en `concepts`.

Definición:
- (completar)

Relaciones:
- (completar)



## Timeline Conflict Rules

> Placeholder (Jarvis): este término aparece 9 veces en módulos pero no está definido en `concepts`.

Definición:
- (completar)

Relaciones:
- (completar)


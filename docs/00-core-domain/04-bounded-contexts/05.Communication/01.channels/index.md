---
title: Channels
description: Comunidades/audiencias (tipo grupos) para discovery, conversación y distribución de contenido (posts, listings, events).
---

# Módulo Channels

## TL;DR
Un **Channel** es una **comunidad/audiencia** dentro de VyteMerge (similar a un grupo de Facebook o un subreddit) donde usuarios se reúnen por interés, ubicación o pertenencia.

Los Channels sirven para:
- **Discovery**: ver publicaciones y listings relevantes por contexto (ej: “Mascotas”)
- **Conversación**: chat y comentarios alrededor del contenido
- **Distribución**: publicar listings/posts a una audiencia específica
- **Gobernanza**: reglas de acceso (public/private), moderación y roles

> Un Channel puede funcionar como “canal de venta”, pero el Channel es primero una **comunidad**.  
> La “venta” es una función de distribución de Listings.

---

## Contexto / Objetivo
VyteMerge mezcla red social + marketplace + booking.

El problema:
- si todo vive en un feed global, el contenido se pierde
- si todo es privado, no hay descubrimiento

Channels resuelven esto creando “lugares” con contexto donde:
- la gente descubre ofertas y servicios relevantes
- los vendedores encuentran audiencias naturales
- se coordina (chat) sin salir de la plataforma

---

## Qué es un Channel
Un Channel es una entidad social con:
- **Identidad**: nombre, descripción, cover, tags
- **Reglas**: público/privado, join policy
- **Miembros**: quién pertenece
- **Roles**: owner/mods/members
- **Superficies**: feed del channel, chat del channel, highlights
- **Contenido**: posts, listings, y opcionalmente events

Ejemplo:
- “Mascotas” → veterinarios, paseadores, tiendas de alimento, dueños de mascotas
- “Padres Salita 3B” → anuncios del colegio, eventos escolares, fotos
- “Fútbol 5 Zona Norte” → partidos abiertos, organización de equipos, canchas

---

## Tipos de Channel (orientados a negocio)

### Public
- cualquiera puede ver
- cualquiera puede unirse (o seguir)
- ideal para comunidades temáticas: “Mascotas”, “Cocina saludable”, “Fitness”

### Private
- solo miembros ven contenido
- ingreso por invitación o aprobación
- ideal para grupos cerrados: “Padres Salita 3B”, “Equipo del trabajo”

### Read-only / Announcements
- muchos pueden ver, pocos publican (mods/admins)
- ideal para instituciones: colegio, club, municipalidad
- reduce spam y mejora precisión de comunicación

> Estos tipos pueden implementarse como configuraciones: `Visibility` + `JoinPolicy` + `PostingPolicy`.

---

## Roles y permisos (modelo simple)

| Rol | Qué puede hacer |
|---|---|
| Owner | configura el channel, nombra mods, define reglas, archiva |
| Moderator | modera contenido/miembros, aprueba requests, borra spam |
| Member | ve el contenido, publica (si está permitido), comenta |
| Guest | (solo en Public) puede ver preview, pero no postea |

Reglas recomendadas:
- Un channel siempre tiene **1 owner**
- Moderators pueden ser varios
- Channels privados: sin membership no hay lectura

---

## Membership (cómo entra la gente)

### JoinPolicy (política de entrada)
- `Open`: cualquiera se une
- `Request`: el usuario pide, un mod aprueba
- `InviteOnly`: solo por invitación

### Estado de membresía
- `Active`
- `Pending`
- `Banned`
- `Left`

> La moderación y anti-spam son parte esencial si los users pueden crear channels libremente.

---

## Qué se publica dentro de un Channel

### Content Types (MVP sugerido)
- **Posts** (social)
- **Listings** (comercial: reservar/comprar)
- (opcional) **Events** (eventos comunitarios)

### Política de publicación (PostingPolicy)
- `MembersCanPost`: sí/no
- `OnlyModsCanPost`: sí/no
- `ListingsAllowed`: sí/no
- `EventsAllowed`: sí/no

Ejemplo:
- “Padres Salita 3B” → OnlyModsCanPost + EventsAllowed + PostsAllowed (con moderación)
- “Mascotas” → MembersCanPost + ListingsAllowed

---

## Discovery y Feed dentro del Channel
Un Channel típicamente tiene un feed propio:
- posts recientes (social)
- listings relevantes (comercial)
- “destacados” (pinned)

Beneficios:
- el usuario entra con un “contexto mental”
- el marketplace se vuelve “curado por intereses”
- se reduce el esfuerzo de búsqueda

---

## Chat del Channel (opcional)
Un Channel puede tener chat para:
- coordinar (ej: partido público)
- consultar a vendedores (si se permite)
- generar comunidad (enganche)

Regla recomendada:
- chat habilitable por channel
- moderación: reportes + mutear/bloquear

---

## Integración con Listings (conceptual)
Un Listing puede declarar:
- `Visibility = Public | Private | FollowersOnly | Restricted`
- `ChannelIds[]` (si se distribuye a channels específicos)

Entonces:
- el Channel es la comunidad
- el Listing se “publica” dentro del Channel como surface de discovery

> La decisión final de visibilidad se aplica por Access (bloqueos/privacidad) y por el propio Channel (private membership).

---

## Integración con Access / Agreements
Channels no reemplaza Access:
- si hay bloqueo entre usuarios → Access puede ocultar contenido
- si un channel es private → membership controla lectura
- agreements pueden habilitar contenido especial (ej: partner-only) dentro de channels específicos (futuro)

---

## KPIs (para producto)
- Channels creados por semana
- miembros activos por channel
- publicaciones por channel (posts/listings)
- conversión de listings publicados en channels (views → bookings)
- ratio de reportes/spam (salud del ecosistema)

---

## Riesgos & Open Questions
- spam si cualquiera crea channels sin límites
- moderación: reporting, bans, y escalado
- solapamiento: “channel” vs “tags” vs “search”
- canales temáticos globales (curados por la plataforma) vs user-generated

---

## Checklist / Próximos pasos
- [ ] Definir tipos: public/private/read-only
- [ ] Definir roles: owner/mod/member
- [ ] Definir membership flow: open/request/invite
- [ ] Definir content policy: posts/listings/events
- [ ] Definir surfaces: channel feed + (opcional) chat
- [ ] Integración con Listings: `ChannelIds[]` + `Visibility`
- [ ] Integración con Access: bloqueos y privacy

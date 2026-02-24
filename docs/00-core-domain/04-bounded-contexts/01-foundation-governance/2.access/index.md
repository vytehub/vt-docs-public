---
title: Access
description: Árbitro central de permisos y visibilidad. Decide qué puede ver/hacer un actor sobre un recurso.
---

# Módulo Access

## TL;DR
**Access** es el “árbitro” que decide **qué puede ver o hacer** un usuario (actor) sobre un recurso (profile, timeline, listing, etc.), de forma **consistente** en toda la plataforma.

- No almacena contenido de negocio.
- No devuelve datos (perfiles, timelines, bookings).
- Devuelve una **decisión**: permitido/denegado + nivel de visibilidad.

> Si querés evitar bugs de privacidad y lógica duplicada, Access debe ser el único lugar donde se “decide”.

---

## Contexto / Objetivo
En VyteMerge, los recursos se cruzan con reglas sociales y comerciales:

- cuentas públicas/privadas
- followers aceptados o no
- usuarios bloqueados
- contenido compartido manualmente
- delegación operativa (staff)
- partners/resellers (agreements)
- recursos “en organizaciones” (ej. consultorio con múltiples profesionales)

Sin un módulo central, estas reglas tienden a:
- duplicarse en endpoints
- divergir entre módulos
- romperse cuando agregás features (channels, partners, campañas, etc.)

**Access existe para concentrar esas decisiones.**

---

## Qué problemas resuelve
Access responde preguntas como:

- ¿Puedo ver este Profile? ¿con qué detalle?
- ¿Puedo ver este Timeline? ¿solo ocupado o con detalle?
- ¿Estoy bloqueado por este usuario/organización?
- ¿Tengo un Agreement que me habilita (staff/partner/sharing)?
- ¿Puedo gestionar (operar) un recurso o solo verlo?

Y lo hace sin que Users/Timelines/Listings tengan que conocer reglas sociales o de agreements.

---

## Concepto clave: AccessDecision + VisibilityMode
Access no decide “qué endpoint llamar”.
Decide **cuánta información es visible** y **qué acciones están permitidas**.

### VisibilityMode
- **None**  
  No hay acceso. El recurso se comporta como inexistente (o “no disponible”).

- **BusyOnly**  
  Vista mínima. El recurso existe, pero sin detalles sensibles.  
  Ejemplo: “ocupado / hay disponibilidad”, sin horarios exactos ni datos privados.

- **BusyOnlyDetails**  
  Vista completa dentro de lo permitido por la plataforma.  
  Ejemplo: detalles completos del perfil o listing; horarios exactos si corresponde.

> Regla mental: `VisibilityMode` controla “cuánto detalle se revela”, no “si existe”.

---

## Access no devuelve datos
Access devuelve una **decisión**:

- `Allowed` (bool)
- `VisibilityMode`
- `Reason` (auditoría / debugging / soporte)

Cada endpoint representa esa decisión en su respuesta:
- ocultar campos
- degradar información
- esconder CTAs (reservar/contactar)
- devolver 404/403 según política UX

---

## Responsabilidades por módulo (separación clara)

- **Users**  
  Define el Profile/usuario y su privacidad (público/privado).

- **Social**  
  Define relaciones: follow, block.

- **Agreements**  
  Define relaciones formales: sharing, staff, partner/reseller + términos + permisos.

- **Timelines**  
  Define el timeline (tiempo/agenda).

- **Catalog / Listings**  
  Define qué se vende y cómo se vende.

- **Access**  
  Decide si un actor puede ver/usar esos recursos, combinando:
  - privacy
  - follow/block
  - agreements/permisos
  - reglas de cada recurso (por tipo)

---

## Cómo opera Access (modelo mental)
Access evalúa por **tipo de recurso** usando evaluators.

Ejemplos:
- `ProfileAccessEvaluator`
- `TimelineAccessEvaluator`
- (futuro) `ListingAccessEvaluator`, `BookingAccessEvaluator`, etc.

Cada evaluator consulta las **Public APIs** necesarias:
- UsersPublicApi (existencia + privacy del profile)
- TimelinesPublicApi (existencia + owner + privacy)
- Social (follow status + block)
- Agreements (si aplica)

> Access “orquesta” reglas; los datos viven en los módulos dueños.

---

## Ejemplos de negocio

### Perfil tipo Instagram
- Perfil público → `BusyOnlyDetails`
- Perfil privado + follower aceptado → `BusyOnlyDetails`
- Perfil privado + no follower → `BusyOnly`
- Bloqueado → `None`

### Staff interno (estética nails)
- Cliente ve listing: “Atiende Estética X” (no ve empleada)
- Internamente el booking se asigna a staff (agreement)
- Staff ve el evento en su timeline (según permisos)
- Cliente **no** ve identidad del staff aunque exista staff agreement

### Agenda compartida
- Pareja/familia (sharing) → puede ver “ocupado” o detalle según permiso
- Staff (delegation) → puede gestionar agenda (si se le otorgó `ManageSchedule`)
- Usuario bloqueado → `None`

---

## Beneficios del enfoque
- reglas consistentes en toda la plataforma
- menos bugs de privacidad
- menos lógica duplicada en endpoints
- fácil agregar nuevos recursos (nuevo evaluator)
- mejor auditabilidad (reasons + agreements)

---

## Checklist / Próximos pasos
- [ ] Definir qué recursos tienen evaluator en MVP (Profile, Timeline, Listing)
- [ ] Mapear cada `VisibilityMode` a una política concreta por endpoint
- [ ] Asegurar que ningún endpoint “decida” privacidad por su cuenta
- [ ] Documentar razones estándar (Blocked, Owner, FollowerAccess, Agreement, etc.)

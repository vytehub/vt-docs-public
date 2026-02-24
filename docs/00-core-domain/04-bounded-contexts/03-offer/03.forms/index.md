---
title: Forms
description: Formularios dinámicos (Form Templates) para capturar información. Se usan en Bookings, Events y Channels, y guardan versiones/snapshots.
position: 7
---

# Forms

## TL;DR
**Forms** permite crear **formularios dinámicos** reutilizables para capturar información y respuestas.

- Un **Listing** puede requerir un Form para reservar.
- Un **Booking** guarda un **snapshot** (qué versión del form se respondió y cuáles fueron las respuestas).
- En el futuro, Forms puede crecer hacia:
  - fichas de clientes (CRM liviano)
  - encuestas en Channels
  - integraciones (export/import) con herramientas externas

> Importante: aunque “se usa en Listings”, Forms se diseña como **módulo aparte** para poder reutilizarlo en más superficies sin duplicar lógica.

---

## Contexto / Objetivo
Muchas reservas necesitan información adicional para poder prestarse correctamente:

- **Cumpleaños**: acompañantes, alergias, confirmación.
- **Veterinaria**: especie, peso, síntomas.
- **Peluquería/estética**: preferencias, historial, observaciones.
- **Partidos abiertos**: posición, nivel, disponibilidad, confirmación.

Forms busca:
- reducir fricción en el booking
- evitar mensajes sueltos por chat
- estandarizar datos (mejor operación, mejores reportes)

---

## Conceptos principales

### Form Template
Un **Form Template** es una “plantilla” reusable.

Incluye:
- Título y descripción
- Lista de campos (Fields)
- Reglas de obligatoriedad
- (opcional) lógica simple: mostrar/ocultar campos según respuestas (futuro)

### Submission
Una **Submission** es una respuesta concreta a un form.

Incluye:
- respuestas a los campos
- quién lo completó (si aplica)
- contexto de uso (ej: BookingId / EventId / ChannelId)

---

## Uso en Booking (Intake Form)
El caso más común es el **Intake Form** al reservar.

### Datos base mínimos (recomendación)
Incluso sin form personalizado, para cualquier booking normalmente necesitás:
- `Name`
- `LastName`
- `Email`

> Estos campos pueden ser autocompletados si el usuario está logueado, pero el sistema debe poder pedirlos si no lo está.

### Campos adicionales
El owner del listing puede agregar campos extra:
- `Phone`
- “¿Viene con acompañante?”
- “¿Es celíaco?”
- “Observaciones”

MVP recomendado:
- soportar solo `Text` / `Input` (como tu nota original)
- luego sumar `Select`, `Checkbox`, `Number`, `Date`, etc.

---

## Regla de integración (para no complicar Listings)
- **Forms vive como módulo propio.**
- **Listings solo referencian un Form** (si lo requieren).
- **Bookings guardan el snapshot** de respuestas.

Por qué:
- el mismo form puede usarse en otros contextos (Events, Channels, CRM)
- permite versionado y auditoría sin acoplarlo a Listings

---

## Versionado y Snapshots (regla clave)
Los forms cambian con el tiempo (se agregan preguntas, se modifican opciones).

Para evitar problemas:
- cuando un Booking usa un Form, se guarda:
  - `FormId`
  - `FormVersion` (o `PublishedAt`)
  - `SubmissionId`
  - snapshot de preguntas relevantes (si hace falta para auditoría)

Esto permite:
- que un booking de hace 3 meses siga “teniendo sentido”
- que el owner no rompa reservas pasadas al editar un formulario

---

## Privacidad (visión simple)
Las respuestas pueden ser sensibles (salud, alergias, contacto).

Reglas recomendadas:
- solo el owner/operadores autorizados pueden ver submissions asociadas a sus bookings
- el usuario que respondió puede ver su propia submission
- no se publican en feed/channels salvo que sea explícitamente una encuesta pública

---

## Evolución futura (sin comprometer implementación)
Forms puede crecer a:

### 1) Encuestas (Channels)
- Un channel puede publicar una encuesta (form)
- miembros responden
- se ven resultados agregados (futuro)

### 2) Ficha de clientes (CRM liviano)
- un form reusable “Ficha de cliente”
- cada visita/reserva puede agregar una nueva submission
- se arma historial por cliente (con permisos)

### 3) Integraciones externas
- exportar submissions (ej: CSV/Sheets)
- importar un form como template (según capacidades externas)

> Se documenta como dirección futura, no como capacidad garantizada.

---

## Checklist (MVP)
- [ ] Booking requiere datos base (Name/LastName/Email) si no hay identidad
- [ ] Listing puede referenciar `FormId` opcional
- [ ] Soportar fields simples `Text/Input`
- [ ] Guardar `Submission` y snapshot/version en Booking
- [ ] Restringir visibilidad de respuestas (privacy)

---

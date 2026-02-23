---
title: Agreements
description: Relaciones formales entre Profiles para compartir, delegar operación (staff) y habilitar partnerships/resellers, con permisos, scope, términos y revocación inmediata.
---

# Módulo Agreements

## TL;DR
**Agreements** define relaciones formales entre **Profiles** (personas u organizaciones) para habilitar:

- **Sharing**: compartir visibilidad (ej. calendario escolar, calendario familiar)
- **Delegation (Staff)**: delegar operación sin perder control (ej. secretarias, docentes, asistentes)
- **Partner / Reseller**: habilitar distribución/comercialización por terceros (ej. revendedores)

Un Agreement combina:
- **Permisos**
- **Scope** (qué recursos cubre)
- **Términos** (Platform + Custom)
- **Revocación inmediata**
- **Auditoría** (quién aceptó qué y cuándo)

El módulo **Access** usa Agreements para decidir *quién puede ver o hacer qué* sobre recursos como **Timelines**, **Catalog (Services/Products)**, **Listings**, **Bookings** y, en el futuro, **Orders**.

---

## Contexto / Objetivo
VyteMerge es una red social donde **cada persona u organización mantiene su propia cuenta (Profile)**.

En la práctica, muchos escenarios requieren relaciones más formales que un simple “follow”:

- Compartir un calendario familiar.
- Un colegio comparte un timeline escolar con padres.
- Un hospital delega operación diaria a su staff (secretarias, asistentes).
- Un proveedor permite que un partner revenda sus servicios bajo reglas comerciales.

Agreements existe para **modelar estas relaciones de forma clara, auditable y extensible**, sin romper el paradigma social del producto.

---

## Alcance del módulo

### Incluido (MVP)
- Crear Agreement (Draft)
- Enviar invitación
- Aceptar / rechazar
- Revocación inmediata por el Owner
- Definir **Scope** (Profile / Timeline / Catalog / Listing)
- Definir permisos (lenguaje de negocio)
- Incluir **Platform Terms + Custom Terms**
- Exponer vistas UX derivadas (Staff, Shared Calendars, Partners)

### Fuera de alcance (por ahora)
- Firma electrónica
- Adjuntos PDF
- Automatización de pagos/revenue share automático
- Resolución formal de disputas legales

---

## Conceptos clave

### Agreement
Un **Agreement** es una relación formal entre:
- **Owner**: Profile que inicia y controla el acuerdo
- **Participant**: Profile invitado que acepta o rechaza
- **Type**: tipo de relación (Sharing / Delegation / Partner)
- **Scope**: recursos a los que aplica
- **Permissions**: acciones permitidas
- **Terms**: condiciones y disclaimers (Platform + Custom)
- **Status**: Draft / Invited / Accepted / Rejected / Revoked / Expired (conceptual)

> Los Agreements son conceptos de dominio.  
> En UX se muestran como “Staff”, “Calendarios compartidos” o “Partners”, no como “contratos”.

---

## Agreement Types (tipos)

| Tipo | Qué habilita | Ejemplo |
|---|---|---|
| **Sharing** | Compartir visibilidad/lectura | Colegio → padres (timeline escolar) |
| **Delegation (Staff)** | Delegación operativa controlada | Hospital → secretarias (gestión turnos) |
| **Partner / Reseller** | Relación comercial con condiciones | Proveedor → revendedor |

---

## Scope (alcance)
El Scope define “qué recursos cubre este Agreement”.

| Scope Type | Significado |
|---|---|
| **Profile** | Aplica a la cuenta del Owner (visión general, administración parcial, etc.) |
| **Timeline** | Aplica a un timeline específico (agenda/visibilidad/gestión) |
| **Catalog** | Aplica al catálogo del Owner (services/products; más común para partners) |
| **Listing** | Aplica a una oferta puntual (una promo, un canal, un reseller específico) |

> Nota: en la documentación anterior aparecía “Offering / AllOfferings”.  
> En VyteMerge eso migra a **Catalog / Listing**.

---

## Permissions (lenguaje de negocio)
Los permisos deberían ser **pocos, claros y componibles**.

Ejemplos comunes:
- `ViewBusyOnly` (ver ocupado / mínimo)
- `ViewDetails` (ver detalle)
- `ManageSchedule` (gestionar agenda: bloquear, mover, disponibilidad)
- `ManageBookings` (confirmar/cancelar/reprogramar bookings)
- `ManageListings` (publicar/archivar/editar listings)
- `DeriveListing` (crear listing derivado si es partner)
- `Resell` (revender / distribuir)

### Regla clave (control)
- **Solo el Owner** puede invitar y revocar acuerdos de tipo **Delegation (Staff)** sobre sus recursos.

---

## Staff (Delegation) ≠ Provider público
Un Agreement de **Staff** sirve para operación interna (capacidad, assignment, agenda), pero **no obliga** a exponer públicamente quién atiende.

Ejemplo estética nails:
- Las empleadas existen como staff para asignación y agenda.
- El cliente ve “Atiende: Estética X” (marca), **no** “Atiende: Empleada 2”.

Esto se logra porque:
- Staff se modela como Agreement (interno)
- La visibilidad de “quién atiende” se controla desde el **Listing** (configuración de presentación/privacidad)

---

## Caso clave: Organizaciones + profesionales con agenda propia (centro médico)
Escenario: la organización vende el servicio, pero el profesional quiere controlar su agenda.

### Modelo recomendado (separación “venta” vs “tiempo”)
- El **Centro Médico** controla el **Listing** (precio, publicación, promos, canales).
- El **Médico** controla su **Timeline** (su tiempo real: hospital + consultorio privado, etc.).
- El Listing del centro puede usar el timeline del médico como fuente de disponibilidad (según permisos).

### Agreements involucrados (mínimos)
1) **Delegation (Staff)**: Centro → Secretarias  
   - Scope: Profile/Centro o Listings específicos  
   - Permisos: `ManageBookings`, `ManageSchedule` (si corresponde), `ManageListings` (si aplica)

2) **Delegation o Sharing**: Médico → Centro (y/o secretarias)  
   - Scope: Timeline del médico  
   - Permisos: `ManageSchedule` (si secretaría agenda) o `ViewDetails` (si solo lectura)

> Regla importante: si el médico revoca el agreement, el acceso operativo a su agenda se corta inmediatamente.

### Beneficios
- El médico mantiene control real sobre su tiempo.
- El centro opera comercialmente el servicio.
- Staff trabaja solo con autorización explícita.
- Auditoría y revocación clara ante cambios laborales.

---

## Terms & Disclosures (términos y disclaimers)

### Por qué existen términos
Incluso acuerdos simples (familia, colegio) pueden generar:
- confusión sobre responsabilidad
- problemas de privacidad
- mal uso de información
- conflictos comerciales

Los términos brindan claridad y protección.

---

### Capas de términos en Agreements
Cada Agreement incluye dos capas:

#### 1) Platform Terms (VyteMerge)
Siempre incluidos y no opcionales. Versionados (`v1`, `v2`, ...).

Incluyen típicamente:
- disclaimer de responsabilidad de la plataforma
- privacy & acceptable use
- políticas de abuso/fraude
- marco legal general
- data handling disclosure

#### 2) Custom Terms (del Owner)
Definidos por el Owner según relación.

Ejemplos:
- Sharing: “uso personal únicamente”
- Delegation: responsabilidades operativas, límites, confidencialidad
- Partner/Reseller: attribution, reglas de marca, canales permitidos, condiciones comerciales

---

### Aceptación y auditoría
Al aceptar un Agreement se registra:
- versión de Platform Terms
- snapshot de Custom Terms
- timestamp
- Profile que aceptó

Esto evita ambigüedades futuras.

---

### Revocación
- El Owner puede revocar en cualquier momento.
- Efecto inmediato.
- El historial se conserva para auditoría.

---

## Integración con Access
Access utiliza Agreements para enforcement:
- verifica si existe un Agreement activo
- aplica permisos y visibilidad
- combina con privacidad/bloqueos (social)

Implementación MVP:
- Access consulta Agreements vía **Public API** del módulo Agreements.

---

## KPIs (para producto)
*(Opcional, pero útil para saber si la feature “vive”)*

- agreements activos por profile
- acuerdos por tipo (sharing vs staff vs partner)
- revocaciones (y motivos/tiempo)
- adopción de staff en organizaciones
- listings derivados por partner agreements (si aplica)

---

## Riesgos & Consideraciones
- Evitar oversharing por defaults incorrectos
- Mantener permisos simples al inicio (no explotar en 30 flags)
- Versionar correctamente Platform Terms
- UX: mostrar “Staff / Calendarios compartidos / Partners” sin jerga legal

---

## Próximos pasos
- [ ] Renombrar scopes antiguos (Offering/AllOfferings) → Catalog/Listing
- [ ] Definir set final de permissions MVP
- [ ] Definir cómo un listing controla si muestra staff (BusinessOnly vs Visible)
- [ ] Integrar evaluators de Access que consulten Agreements

# Módulos/features de negocio a revisar (pendientes)

> Objetivo: **solo listar** qué falta cubrir a nivel **lógica de negocio** (no formato de documentación), para revisarlo más adelante.

---

## P0 — Para que el marketplace/booking cierre en producción

> Los cuatro contextos P0 ya tienen documentación inicial en sus propios directorios:

| Contexto | Directorio |
|---|---|
| Policies (Cancelación / Reprogramación / No-show) | `07-policies/index.md` |
| Commerce (Payments / Refunds / Disputes / Payouts) | `06-commerce/index.md` |
| Trust & Safety (Report/Block/Moderation/Reviews) | `09-trust-safety/index.md` |
| Org & Roles (Teams/Staff/Delegation) | `08-org-roles/index.md` |

Cada doc incluye entidades, comandos, eventos, integraciones y preguntas abiertas a resolver antes de implementar.

---

## P1 — Para escalar oferta/demanda (growth y calidad)

### 5) Supply onboarding & Quality (alta y control de catálogo)
**Qué falta definir:**
- Flujo de onboarding para business/place/listing (pasos y requisitos).
- Verificación mínima (opcional): identidad, place, propiedad.
- Reglas de calidad: qué listings se publican, cómo se corrigen, cómo se despublican.

---

### 6) Pricing & Promotions (Campaigns “de verdad”)
**Qué falta definir:**
- Reglas de precio: tiers, peak/off-peak, dinámico (aunque sea “later”).
- Promos/cupones: elegibilidad, stacking, expiración.
- Atribución y métricas: qué cuenta como conversión de una Campaign.

**Encaje en modelo:** Campaign + Listing + Order (+ attribution).

---

### 7) Discovery & Ranking (Search/Suggest/Feed con objetivos)
**Qué falta definir:**
- Objetivo principal: relevancia vs conversión vs calidad.
- Señales: follows, bookings, reviews, disponibilidad, distancia, precio.
- Controles de fairness/anti-spam (no “pay-to-win” sin disclosure).

---

### 8) Inventory avanzado (recursos, capacidades, buffers)
**Qué falta definir:**
- Recursos: canchas/salas/equipos/staff como “capacity constraints”.
- Buffers (setup/cleanup), capacidades simultáneas, reglas de conflicto.
- Waitlist, overbooking controlado (si aplica).

**Encaje en modelo:** Slot generation + Place/Resources + Booking constraints.

---

## P2 — Operación, cumplimiento y “empresa”

### 9) Support/CRM y herramientas internas (Admin/Ops)
**Qué falta definir:**
- Gestión de casos: disputes, refunds, moderación, accesos.
- SLAs, macros, escalamiento, logging/auditoría.

---

### 10) Billing / Invoices / Tax (según país/vertical)
**Qué falta definir:**
- Facturación, impuestos/retenciones, perfiles fiscales.
- Reglas por tipo de proveedor y jurisdicción (si aplica).

---

### 11) Compliance & Data retention (mensajes, privacidad, borrado)
**Qué falta definir:**
- Retención de mensajes (Channels/Chat), export/borrado.
- Políticas de privacidad por tipo de dato y actor (Profile/Business/Staff).

---

### 12) Integrations (calendarios, pagos, mapas)
**Qué falta definir:**
- Calendar sync (in/out), conflictos y prioridad.
- PSP (si hay pagos): provider-agnostic rules y fallback manual.
- Map/geo y distancia para discovery.

---

## Checklist rápido de “qué decidir” (para cuando retomes)
- ¿Phase 1 tiene pagos? (sí/no)
- Si sí: definir **Policies + Commerce + Disputes + Payout** (aunque sea MVP manual).
- Definir **Trust & Safety MVP** antes de crecer socialmente.
- Definir **Roles/Org** si hay B2B (clínicas, canchas, teams).
- Elegir cuáles de P1 van en la próxima iteración (Campaigns / Ranking / Inventory).


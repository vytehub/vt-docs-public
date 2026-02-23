# Mdulos/features de negocio a revisar (pendientes)

> Objetivo: **solo listar** qué falta cubrir a nivel **lógica de negocio** (no formato de documentación), para revisarlo más adelante.

---

## P0 — Para que el marketplace/booking cierre en producción

### 1) Policies (Cancelación / Reprogramación / No-show)
**Por qué es crítico:** es donde se rompen los casos reales (soporte, conflictos, confianza, conversión).  
**Reglas mínimas a definir (v1):**
- Ventanas de cancelación y reprogramación (por Listing/Service).
- Penalidades: fee fijo o porcentaje; quién cobra y a quién afecta.
- No-show: cómo se determina “asistencia” y cómo se resuelve (prueba, apelación).
- Impacto en Booking lifecycle y (si hay pagos) en Order.

**Encaje en modelo:** Listing.Policy + Booking lifecycle + Order adjustments (si aplica).

---

### 2) Commerce end-to-end (Payments / Refunds / Disputes / Payouts)
**Por qué es crítico:** sin settlement/payout/refunds el “Order” queda incompleto para marketplace real.  
**Reglas mínimas a definir (v1):**
- Estados de Order: created → authorized/paid → refunded (parcial/total).
- Regla de payout: cuándo se libera dinero (al confirmar vs al completar vs con hold).
- Disputas/chargebacks: quién arbitra, plazos, evidencias, resultado.
- Relación Booking ↔ Order: qué acciones del Booking mutan el Order (cancel/reschedule/no-show).

**Encaje en modelo:** Order (+ futuros objetos: Payout/Refund/Dispute) ↔ Booking/Event.

---

### 3) Trust & Safety (Report/Block/Moderation/Reviews)
**Por qué es crítico:** social graph + marketplace escala abuso, spam, fraude; reviews sostienen confianza.  
**Reglas mínimas a definir (v1):**
- Report/Block para Profile, Listing, Post, Channel.
- Acciones de moderación: hide, suspend, takedown, ban.
- Reviews/ratings: elegibilidad (ej: solo Booking=Completed), anti-abuso básico.
- Guidelines de contenido y enforcement.

**Encaje en modelo:** Profile/Follow/Post/Listing/Channel + reglas de elegibilidad ligadas a Booking.

---

### 4) Org & Roles (Teams/Staff/Delegation operativa)
**Por qué es crítico:** B2B real necesita roles y auditoría; sin esto, operación manual.  
**Reglas mínimas a definir (v1):**
- Roles: owner / manager / staff.
- Permisos por acción: publicar Listing, confirmar Booking, operar refunds, ver datos.
- Delegación y auditoría (audit log) para acciones sensibles.
- Multi-Profile / multi-Place / multi-Timeline (si aplica por vertical).

**Encaje en modelo:** Agreements + Access + (Profile como Org) + ownership/delegación sobre Timelines/Listings.

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


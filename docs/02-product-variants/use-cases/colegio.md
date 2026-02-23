---
title: Colegio timelines privados + comunicación + pagos + media
---

# Colegio: timelines privados + comunicación + pagos + media

## TL;DR
El colegio quiere compartir eventos con padres por curso con **privacidad**, comunicar cambios (cancelaciones) y permitir **pagos** de salidas; los padres quieren integrar esos eventos con su vida diaria y aplicar conflictos para no solapar.

## Actores
- Colegio (Profile organización)
- Docentes/Staff (Profiles)
- Padres (Profiles)
- Alumnos (dependientes/implícitos)

## Objetivo
Comunicación confiable y privada: calendario escolar por curso, pagos simples, media post-evento y notificaciones de cambios.

## Contexto
Hoy usan WhatsApp/papel. Baja precisión ante lluvia/cortes de luz. Quieren centralizar y mantener privacidad.

## Flujo principal
1. Colegio crea **Timeline** por curso (ej “Salita 3B”).
2. Crea **Channel/Group** “Padres Salita 3B”.
3. Publica **Events**: actos, salidas, cancelaciones.
4. Padres consumen canal y aplican **Conflict Rules** personales.
5. Para salidas: **Order** (pago) asociado al evento.
6. Media: posts con fotos/videos en el canal.

## Requisitos / Reglas
- Privacidad fuerte: solo miembros autorizados.
- Cambios de evento: estados confiables (cancelado/reprogramado).
- Pagos: recibo/estado claro.
- Moderación y consentimiento (menores).

## Mapping al Core Model
| Concepto | En este caso |
|---|---|
| **Profile** | Colegio + padres + staff. |
| **Timeline** | Timeline por curso. |
| **Service** | Opcional: “Salida escolar” (si se estandariza). |
| **Product** | Opcional: merchandising/material (futuro). |
| **Catalog** | No central (salvo ventas). |
| **Listing** | Opcional: “pago de salida” como listing/producto. |
| **Slot** | No central. |
| **Event** | Eventos escolares + cambios de estado. |
| **Booking** | Opcional: cupos para excursión (futuro). |
| **Attendees** | Alumnos como asistentes (futuro). |
| **Order** | Pago de salidas/actividades. |
| **Agreements/Access** | Access por canal + acuerdos de staff. |
| **Campaigns** | No central. |
| **Feed/Chat/Channels** | Canales por curso + feed de anuncios. |

## KPIs
- % padres activos
- tiempo de reacción ante cambios
- tasa de pagos completados
- reducción de consultas repetidas

## Edge cases
- Padres separados/tutores: permisos por alumno.
- Alta/baja alumnos: membership dinámico.
- Cancelación: ¿requiere “ack” leído?
- Reembolsos.

## Preguntas abiertas
- ¿Alumnos como entidad en MVP?
- Consentimiento media: ¿cómo se registra?
- ¿Pago de salida es Order puro o Booking con cupos?

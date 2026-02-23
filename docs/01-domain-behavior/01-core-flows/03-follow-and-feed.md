---
title: Flow 03 - Follow, aprobación y Feed
description: Un usuario sigue un perfil, puede requerir aprobación, y luego consume posts y listings en feed.
---

# Flow 03 — Follow, aprobación y Feed

## Resumen
- **Goal:** que un usuario siga a un perfil (persona/negocio), con reglas de privacidad, y luego vea posts/listings en superficies (feed/discovery).
- **Actores:** Usuario A, Perfil B (usuario/negocio), Sistema VyteMerge.
- **Contextos:** Social (profiles, graph, posts), Governance (privacy/access), Discovery/Suggest (feed surfaces), Communication (notifications).

## Preconditions
- Perfiles A y B existen.
- Reglas de privacidad de B conocidas (público/privado/aprobación).

## Main Flow (paso a paso)
1. Usuario A visita Profile B.
2. Usuario A ejecuta Follow.
3. Sistema evalúa política:
   - si B es público: follow se activa inmediatamente
   - si B requiere aprobación: queda pendiente
4. (Si aplica) B aprueba o rechaza.
5. Sistema actualiza Social Graph y recalcula elegibilidad del Feed.
6. Usuario A ve en su Feed:
   - posts de B
   - listings de B (si visible y published)
7. Notificaciones:
   - B recibe request/confirmación de follow
   - A recibe resultado (aprobado/rechazado)

## Domain Trace (Command → Aggregate → Event)
- **Command:** `RequestFollow`
  - **Aggregate:** `SocialGraph` / `FollowRelation`
  - **Events:** `FollowRequested`
- **Command:** `ApproveFollow` / `RejectFollow` (si privado)
  - **Aggregate:** `FollowRelation`
  - **Events:** `FollowApproved` / `FollowRejected`
- **Projections:**
  - `FeedEligibilityUpdated`
  - `SuggestInputsUpdated` (si aplica)
  - `NotificationsEnqueued`

## Edge Cases
- Bloqueos/privacidad: follow puede estar prohibido por Access rules.
- Unfollow: debe retirar contenido del feed (¿inmediato o eventual?). **NEEDS CLARIFICATION**
- Rate limits/abuse: moderación. (documentar luego)

## Links (fuente de verdad)
- Social overview: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/01.overview.md`
- Profiles: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/02.profiles.md`
- Social graph: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/03.social-graph.md`
- Privacy: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/06.privacy-moderation.md`
- Discovery/Feed: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/04.search_discovery/04.discovery.md`

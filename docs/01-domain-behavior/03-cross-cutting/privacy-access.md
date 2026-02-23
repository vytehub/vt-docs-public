---
title: Privacy & Access Enforcement (curated)
description: Cómo se aplican reglas de privacidad/visibilidad en perfiles, timelines, listings y discovery.
---

# Privacy & Access Enforcement (curated)

## Objetivo
Centralizar **cómo** se decide si un usuario puede:
- ver un Profile/Post/Listing
- ver Slots proyectados
- ejecutar Booking
- recibir recomendaciones (Suggest)

## Capas de control (orden recomendado)
1. **Account/Profile privacy** (quién puede ver/seguir)
2. **Agreements/Sharing** (delegación/compartición)
3. **Timeline visibility** (qué partes del tiempo se exponen)
4. **Listing visibility + channels** (public/unlisted/private)
5. **Discovery eligibility** (reglas de index + feed)
6. **Per-field redaction** (ej: ocultar datos sensibles)

## Puntos donde se aplica
- **Open Profile** → puede requerir follow/approval.
- **Open Listing** → listing visible pero slots pueden estar redacted.
- **Search/Discovery** → filtrado previo al render.
- **Booking** → revalidación final (no confiar en UI).

## Links
- Users privacy/visibility: `docs/00-core-domain/04-bounded-contexts/01.Foundation & Governance/1.users/02.privacy_visibility.md`
- Social privacy/moderation: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/06.privacy-moderation.md`
- Discovery eligibility/privacy: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/04.search_discovery/05.eligibility-privacy.md`
- Listing privacy: `docs/00-core-domain/04-bounded-contexts/03.Offer - Catalog & Listings/02.listing/12.privacy.md`

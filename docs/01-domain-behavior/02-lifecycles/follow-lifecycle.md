---
title: Follow Lifecycle (draft)
description: Estados para relaciones de follow según privacidad y moderación.
---

# Follow Lifecycle (draft)

## Estados
- **Requested** (si requiere aprobación)
- **Active**
- **Rejected**
- **Blocked** (moderación/abuse)
- **Removed** (unfollow)

```mermaid
stateDiagram-v2
  [*] --> Requested
  Requested --> Active: ApproveFollow
  Requested --> Rejected: RejectFollow
  Active --> Removed: Unfollow
  Active --> Blocked: Block
  Requested --> Blocked: Block
```

## Links
- Social graph: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/03.social-graph.md`
- Privacy/moderation: `docs/00-core-domain/04-bounded-contexts/04.Social & Discovery/01.social/06.privacy-moderation.md`

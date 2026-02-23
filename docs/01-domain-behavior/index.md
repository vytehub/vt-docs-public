# Domain Behavior

Esta sección **curará comportamiento de dominio** (end-to-end) a partir de la documentación del Core Domain (Bounded Contexts).

## Cómo usar esta sección
- Si querés entender **"qué pasa"** en el sistema (paso a paso), empezá por **Core Flows**.
- Si querés entender **estados / transiciones**, mirá **Lifecycles**.
- Si querés entender reglas transversales (privacidad, timezones, notificaciones), mirá **Cross-cutting**.

> Regla: acá documentamos **comportamiento** (Commands → Aggregates → Events → Outcomes).  
> Los detalles de “qué es” cada entidad viven en **Core Domain**.

## Notación
- **Actor**: humano/sistema que inicia.
- **Command**: intención/acción solicitada.
- **Aggregate**: entidad que aplica invariants.
- **Domain Event**: hecho ocurrido (fuente de verdad para integraciones).
- **Read Models / Projections**: vistas para UI / search / feed.

# Product Skeleton v1 — VyteMerge

## Estado

Draft v1

## Propósito

Este documento define la **forma visible del producto**.

No describe detalles de implementación backend ni un flow puntual.
Su objetivo es fijar una visión compartida de:

- qué es VyteMerge para el usuario
- qué representa cada surface principal
- qué debe ver un usuario al entrar
- qué debe vivir en el shell
- qué puede vivir en los MFEs
- cómo cerrar la experiencia general antes de seguir agregando features

Este documento existe para evitar que la implementación derive en una suma de features sin forma de producto.

---

# 1. Product statement

## Definición corta

**VyteMerge es una red social comercial centrada en tiempo.**

El usuario entra a un ecosistema donde descubre actividad, personas, servicios, productos y oportunidades relevantes, y desde ahí puede:

- seguir
- mirar actividad
- reservar
- comprar (a futuro, cuando aplique)
- coordinar esas acciones con su tiempo, disponibilidad, slots y contexto

## Idea fuerza

VyteMerge no es:

- solo una red social
- solo un marketplace
- solo una agenda

VyteMerge une:

- descubrimiento
- visibilidad
- reputación
- comercialización
- coordinación del tiempo

## Promesa base

> descubrir cosas útiles y convertirlas en acciones coordinadas en el tiempo

---

# 2. Principio central del producto

## “Todo se conecta con todo”

VyteMerge debe sentirse como un sistema donde:

- la actividad social influye en el descubrimiento
- el descubrimiento puede llevar a seguir perfiles o abrir offerings
- las offerings pueden llevar a reservar
- las reservas y compromisos se coordinan en el tiempo
- el timeline conecta la capa social/comercial con la capa operativa

## Implicancia UX

Las surfaces no deben sentirse como silos aislados.

Pero tampoco deben mezclarse sin propósito.

La experiencia debe dar una sensación de continuidad entre:

- ver
- interesarse
- seguir
- abrir
- reservar
- coordinar

---

# 3. Top-level surfaces

## 3.1 Home

### Propósito

**Descubrimiento personalizado y activación**

Home es la entrada viva al ecosistema.

No es solo feed.
No es solo marketplace.
No es solo publicidad.

Es una surface curada donde el usuario ve una mezcla relevante de “cosas”.

### Qué puede mostrar

- posts
- promoted placements / publicidad paga
- cards de listings reservables
- perfiles recomendados
- recomendaciones cerca de mí
- bloques o carruseles temáticos
- actividad relevante
- highlights del ecosistema

### Qué debe lograr

Cuando el usuario entra, debe sentir:

- “acá pasan cosas”
- “esto me entiende”
- “puedo explorar, seguir o reservar”
- “este producto está vivo”

### CTA principales

- Reservar
- Seguir
- Ver más
- Abrir perfil
- Abrir listing

### Regla clave

**Home no es Feed puro.**

Home es una surface shell-owned de descubrimiento comercial-social.

---

## 3.2 Explore

### Propósito

**Descubrimiento explícito**

Explore es donde el usuario entra a explorar de forma más intencional.

Si Home lo expone a cosas interesantes,
Explore le permite navegar y descubrir con más control.

### Qué puede mostrar

- grid o stream de descubrimiento
- categorías
- cerca de mí
- popular
- nuevos
- perfiles sugeridos
- listings
- ofertas destacadas
- filtros suaves

### CTA principales

- Explorar categoría
- Abrir listing
- Abrir perfil
- Seguir
- Ver más

### Relación con Home

Explore no debe competir con Home.

Home = mezcla curada/personalizada  
Explore = descubrimiento más intencional y navegable

### Decisión

Explore puede existir como sub-surface dentro del dominio discover/social,
pero no necesariamente como un concepto top-level separado para el usuario.

---

## 3.3 Timeline

### Propósito

**Coordinación del tiempo**

Timeline es la capa operativa del producto.

No es:
- un feed
- una surface de exploración
- un calendario genérico

Es la surface donde el usuario ve y coordina:

- bookings
- slots
- eventos privados
- compromisos
- acuerdos
- interacciones con dimensión temporal

### Qué debe lograr

Cuando el usuario entra a Timeline, debe sentir:

- “acá organizo mi tiempo”
- “acá veo reservas, compromisos y disponibilidad”
- “esto es operativo”

### CTA principales

- Ver disponibilidad
- Gestionar bookings
- Confirmar / rechazar / mover
- Crear evento privado
- Revisar agenda operativa

### Regla clave

**Timeline no debe empujar descubrimiento como identidad principal.**

Puede conectar con otras surfaces, pero su propósito no es “explorar”.

---

## 3.4 Studio

### Propósito

**Creación y gestión de oferta**

Studio es el lado creator/provider del sistema.

### Qué puede contener

- crear servicios
- crear/publicar listings
- configurar precios
- configurar disponibilidad
- reglas y políticas
- solicitudes pendientes
- estado de la oferta
- datos operativos del lado provider

### Qué debe lograr

Cuando el usuario entra a Studio, debe sentir:

- “acá construyo y administro mi oferta”
- “este es mi espacio de trabajo comercial”

### CTA principales

- Crear
- Editar
- Publicar
- Ver solicitudes
- Configurar

### Regla clave

Studio no debe sentirse como un panel técnico o administrativo genérico.

Debe sentirse como:
- mi negocio
- mi oferta
- mi lado provider

---

## 3.5 Profile

### Propósito

**Identidad + reputación + actividad + oferta**

Profile no debe ser solo un hub de links ni una página de settings.

Debe mostrar:

- quién es esta persona o negocio
- qué publica
- qué ofrece
- qué actividad tiene
- qué señales de reputación o confianza transmite
- cómo se conecta con otros

### CTA principales

- Seguir
- Ver actividad
- Ver offerings/listings
- Reservar (si aplica)
- Interactuar

### Regla clave

Profile debe funcionar tanto para:
- mi propio perfil
- perfil ajeno accesible por URL

---

# 4. Qué ve un usuario nuevo vs un usuario activo

## 4.1 Usuario nuevo

Un usuario nuevo:

- no sigue a nadie
- no tiene grafo social
- no entiende todavía el ecosistema
- necesita activación, no vacío

### Decisión

Para usuario nuevo o cold-start:

**Home debe comportarse como Explore-first.**

### Qué debería ver

- contenido popular
- perfiles sugeridos
- listings o servicios destacados
- recomendaciones cerca de mí
- promoted placements relevantes
- bloques que expliquen implícitamente el ecosistema

### Objetivo

- activar curiosidad
- mostrar que hay vida
- ayudar a entender el producto
- dar primeros caminos de acción

---

## 4.2 Usuario activo

Un usuario activo:

- ya tiene follows
- ya tiene señales de interés
- tal vez ya reservó o publicó
- necesita relevancia y continuidad

### Decisión

Para usuario activo:

**Home puede comportarse como Feed/For You-first**, pero manteniendo mezcla de contenido.

### Qué debería ver

- actividad relevante
- contenido de seguidos
- servicios/listings relacionados
- señales cercanas o contextuales
- placements promovidos coherentes

### Objetivo

- sostener engagement
- reforzar relevancia
- facilitar acción rápida

---

# 5. Qué pertenece al Shell y qué a los MFEs

## 5.1 El Shell debe ser dueño de

- navegación top-level
- orden y labels de surfaces
- layout global
- lógica de entrada
- decisión cold-start vs personalized
- experiencia general de Home
- empty states top-level
- product skeleton
- mocked data inicial del sitio para validar la experiencia

## 5.2 Los MFEs deben ser dueños de

- implementación concreta de cada surface
- componentes y estados de la surface
- rendering interno
- data-fetching específico
- interacciones locales de dominio

## Regla general

**El shell cuenta la historia.  
El MFE implementa la surface.**

---

# 6. Navegación top-level

## Decisión general

Mantener la estructura base:

- Home
- Timeline
- Studio
- Profile vía avatar

## Top nav

- logo / marca
- entrada a búsqueda
- notificaciones
- avatar → profile

## Tabs principales

- Home
- Timeline
- Studio

## Decisión sobre Discover

No conviene exponer al usuario final una superposición confusa entre:

- Home
- Discover
- Explore

La recomendación es:

- Home como surface principal
- Explore como sub-surface o vista clara dentro del dominio discover/social
- Discover puede sobrevivir como naming técnico interno, pero no como concepto top-level dominante para el usuario

---

# 7. Home v1 — estructura propuesta

## Principio

Home no debe abrir a una pantalla vacía.

Debe sentirse como:
- viva
- diversa
- relevante
- útil

## Estructura sugerida

### 7.1 Header ligero

- búsqueda
- greeting/contexto ligero
- chips rápidos o atajos:
  - Cerca de mí
  - Populares
  - Esta semana
  - Seguidos

### 7.2 Featured / hero block

- promoted placement
- listing destacado
- campaña contextual
- recomendación editorial/personalizada

### 7.3 Main mixed stream

Intercalado de:
- post
- listing card
- suggested profile
- promoted placement
- activity card

### 7.4 Nearby / local relevance

- servicios
- lugares
- perfiles
- oportunidades cercanas

### 7.5 Recommendations

- perfiles
- categories
- offerings
- suggested follow targets

## Regla

Home puede ser infinite scroll,
pero los primeros bloques deben contar mejor la historia que un feed flat vacío.

---

# 8. Explore v1 — estructura propuesta

Explore debe servir para descubrimiento más explícito.

Puede mostrar:

- grid de listings/perfiles
- filtros
- categorías
- popular
- nuevos
- cerca de mí
- tags o intereses

## Feeling deseado

“Quiero explorar algo más específico”  
no simplemente “seguir scrolleando Home”.

---

# 9. Empty state strategy

## 9.1 Home

### Regla
Home no debe mostrar vacío real como primera experiencia.

Si no hay suficiente data real:
- usar cold-start curation
- usar popular
- usar nearby
- usar recomendaciones mockeadas o del sistema

### No hacer
- “Tu feed está vacío” como experiencia dominante

### Sí hacer
- ofrecer contenido y caminos de entrada

---

## 9.2 Timeline

### Regla
Timeline puede estar vacío, pero el mensaje debe reforzar su identidad temporal.

### Buen mensaje implícito
- “Cuando tengas bookings, eventos o compromisos, los vas a ver acá”
- “Acá coordinás tu tiempo”

### Evitar
- CTAs principales que conviertan Timeline en marketplace/discovery surface

---

## 9.3 Studio

### Regla
Studio sí puede usar empty state invitando a crear la primera oferta.

Eso es consistente con su propósito.

---

## 9.4 Profile

Profile no debería sentirse “vacío” en el sentido de producto débil.

Incluso mockeado, debería tener:
- identidad
- actividad
- offerings
- señales sociales mínimas

---

# 10. Mocked data strategy

## Propósito

Antes de cerrar features reales, el producto necesita una representación visual creíble.

La mock data no debe ser genérica ni azarosa.
Debe contar la historia correcta.

## 10.1 Home mock data

- 2–3 promoted cards
- 4–6 posts
- 4 listings reservables
- 3–4 perfiles sugeridos
- 1 bloque “cerca de mí”
- 1 bloque “esta semana”
- 1 bloque de actividad relevante

## 10.2 Timeline mock data

- 2 bookings
- 1 evento privado
- 1 hueco libre / slot
- 1 reminder/compromiso
- 1 día con poco movimiento

## 10.3 Studio mock data

- 1 servicio
- 1 listing draft
- 1 listing published
- 1 booking request pending

## 10.4 Profile mock data

- bio
- 3 posts
- 2 offerings/listings
- followers/following básicos
- actividad reciente

---

# 11. Qué NO se debe priorizar antes de cerrar este skeleton

No conviene priorizar todavía:

- moderation
- report/block/mute
- wiring fino de feed
- algoritmos reales complejos
- contratos completos de data real
- búsqueda avanzada final
- integración profunda de todas las surfaces

Primero hay que cerrar:
- la forma del producto
- la navegación
- la identidad de cada surface
- la experiencia visible del sitio

---

# 12. Decisiones v1 propuestas

## Decisión 1
**Home no es Feed puro.**

## Decisión 2
**Home debe ser discovery-first para usuario nuevo.**

## Decisión 3
**Explore no debe competir como top-level separado del usuario si genera solapamiento con Home.**

## Decisión 4
**Timeline es la capa operativa del tiempo, no una surface de discovery.**

## Decisión 5
**Shell es dueño de la experiencia top-level y del product skeleton.**

## Decisión 6
**El siguiente gran paso debe ser un shell-first mocked skeleton cycle.**

---

# 13. Próximo deliverable recomendado

El próximo paso después de este documento es crear una spec ejecutable, por ejemplo:

```text
docs/vt-docs/specs/DRAFT-shell-product-skeleton/
```

Con:

- `requirements.md`
- `design.md`
- `tasks.md`
- `acceptance.md`

Esa spec debería traducir este skeleton a trabajo concreto para Claude Code.

---

# 14. Criterio de éxito

Este skeleton se considera exitoso cuando el usuario puede entrar a VyteMerge y sentir, en menos de 30 segundos:

- qué es Home
- qué es Timeline
- qué es Studio
- qué es Profile
- que el producto está vivo
- que hay cosas que descubrir
- que puede seguir, reservar y coordinar
- que todo pertenece al mismo sistema

---

# 15. Frase síntesis del producto

**VyteMerge es una red social comercial centrada en tiempo, donde el usuario descubre cosas relevantes y convierte ese interés en acciones coordinadas en el mundo real.**

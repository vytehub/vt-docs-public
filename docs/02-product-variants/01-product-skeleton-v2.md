# Product Skeleton v2 — VyteMerge

## Estado

Draft v2

## Propósito

Este documento define la **forma visible y conceptual** del producto VyteMerge en su estado actual de entendimiento.

No describe backend ni contratos técnicos detallados.
Su objetivo es fijar una visión compartida de:

- qué es VyteMerge para el usuario
- qué representa cada surface principal
- cómo se relacionan descubrimiento, vida y tiempo
- qué debe vivir en el shell
- qué debe vivir en los MFEs
- cómo cerrar primero la forma del producto antes de normalizar componentes o conectar backend
- cómo seguir trabajando en modo UI/UX-only sin perder foco

Este documento existe para evitar:
- que el producto derive en una suma de features sin forma
- que Timeline se reduzca a agenda genérica
- que Home se convierta en feed vacío o collage sin concepto
- que Explore se vuelva un panel rígido
- que se fuerce toolkit/componentización antes de cerrar la experiencia

---

# 1. Product statement

## Definición corta

**VyteMerge es una red social comercial centrada en tiempo.**

El usuario entra a un ecosistema donde descubre:
- personas
- negocios
- servicios
- productos
- oportunidades
- momentos compartidos
- contextos relevantes

Y desde ahí puede:
- seguir
- mirar actividad
- reservar
- guardar
- agregar cosas a su línea de vida
- coordinar decisiones reales en el tiempo

## Idea fuerza

VyteMerge no es:
- solo una red social
- solo un marketplace
- solo una agenda
- solo un booking system

VyteMerge une:
- descubrimiento
- identidad
- actividad social
- comercialización
- tiempo
- conflictos
- coordinación real

## Promesa base

> descubrir cosas relevantes y conectarlas con mi vida, mi tiempo y mis decisiones reales

---

# 2. Principio central del producto

## “Todo se conecta con todo”

VyteMerge debe sentirse como un sistema donde:

- la actividad social influye en el descubrimiento
- el descubrimiento puede llevar a seguir perfiles, abrir ofertas o reservar
- los momentos compartidos pueden aparecer como fragmentos del graph
- las ofertas/listings consumen o proyectan tiempo
- los recursos tienen tiempo
- las personas tienen tiempo
- los lugares tienen tiempo
- los acuerdos conectan tiempos
- los conflictos revelan decisiones
- el Timeline conecta la capa social/comercial con la capa operativa

## Implicancia UX

Las surfaces no deben sentirse como islas.

Pero tampoco deben mezclarse sin propósito.

La experiencia debe dar una sensación de continuidad entre:

- ver
- interesarse
- seguir
- abrir
- reservar
- sumar a mi vida
- coordinar
- decidir

---

# 3. Principio de esta etapa

## UI/UX-only mode

Durante esta etapa, la prioridad NO es:

- cerrar backend
- conectar APIs reales
- normalizar componentes
- forzar reutilización temprana
- optimizar contratos técnicos

La prioridad es:

- cerrar la forma del producto
- cerrar la navegación
- cerrar Home / Explore / Timeline / Studio / Profile como surfaces
- dar vida al sistema con mock data
- encontrar el lenguaje visual del Timeline
- validar qué se siente correcto antes de endurecer implementación

## Regla complementaria

# **Page-first, toolkit-later**

En esta etapa:
- primero se diseñan y construyen páginas/surfaces
- después se normalizan patrones
- después se componentiza
- después se alinea toolkit

Esto no significa abandonar el stack actual.
Significa no dejar que el toolkit condicione el concepto antes de tiempo.

## Regla técnica

Seguir usando:
- la misma tecnología base
- el mismo stack
- los mismos repos donde ya trabajás
- la misma arquitectura general

No introducir tecnologías nuevas innecesarias en esta etapa.

---

# 4. Qué es cada surface

## 4.1 Home

### Propósito

**Mi línea + las cosas que se conectan con ella**

Home ya no debe pensarse solo como:
- feed
- mezcla de cosas
- discovery surface genérica

Home debe convertirse en la surface donde el usuario siente:

- esta es mi vida / mi tiempo
- esto es lo próximo
- esto se conecta conmigo
- estas son oportunidades relevantes
- esto vale mi atención ahora

### Qué muestra

Home puede mostrar:
- mi mini línea / timeline glimpse
- próximos nodos relevantes
- cosas que encajan con mi tiempo/contexto
- actividad relevante
- perfiles, negocios o recursos conectables
- promoted placements relevantes
- listings reservables
- fragmentos públicos o compartidos del graph
- sugerencias cercanas o contextuales

### Qué debe lograr

Cuando el usuario entra, debe sentir:

- “esto está vivo”
- “esto habla de mi tiempo”
- “esto me muestra cosas que podrían entrar a mi vida”
- “puedo explorar, guardar, reservar o sumar algo”
- “no estoy viendo un feed vacío ni una home genérica”

### CTA principales

- Reservar
- Agregar a mi timeline
- Seguir
- Guardar
- Abrir perfil/listing/contexto
- Ver Timeline completo

### Regla clave

**Home no es Feed puro.**  
**Home no es Timeline completo.**

Home es:
# **mi línea + connected discovery**

---

## 4.2 Explore

### Propósito

**Búsqueda y descubrimiento explícito**

Explore no debe competir con Home con una estructura rígida.

Debe sentirse como:
- simple
- natural
- search-first
- progresivamente refinable

### Qué muestra

Explore debe priorizar:
- un entrypoint de búsqueda claro
- resultados en “Todo” por defecto
- personas / empresas / servicios según corresponda
- refinamiento progresivo con tags o filtros suaves
- hints temporales donde sea útil
- disponibilidad pública o fragmentos temporales si agregan valor

### Qué debe lograr

Cuando el usuario entra a Explore, debe sentir:

- “puedo buscar algo”
- “puedo descubrir sin que me obliguen a elegir demasiadas cosas antes”
- “esto no es un dashboard raro”

### CTA principales

- Buscar
- Refinar
- Abrir resultado
- Reservar
- Agregar a mi timeline
- Guardar

### Regla clave

**Explore es search-first, no graph-first.**  
**Explore no debe ser rígido ni sobre-segmentado.**

---

## 4.3 Timeline

### Propósito

**Línea de vida/tiempo + coordinación + graph emergente**

Timeline ya no debe definirse solo como “capa operativa”.

Timeline es:
- una línea de vida/tiempo
- hecha de nodos temporales
- donde el usuario ve su tiempo
- donde aparecen convergencias
- donde se revelan conflictos
- donde el graph emerge cuando corresponde

### Qué muestra

Timeline puede mostrar:
- mi línea personal
- slots
- eventos
- shared events
- recursos
- related timelines
- conflictos
- merge points
- intersections con privacidad preservada
- vida densa o vida sparse

### Qué debe lograr

Cuando el usuario entra a Timeline, debe sentir:

- “acá veo mi vida en el tiempo”
- “acá entiendo qué viene”
- “acá veo qué se cruza conmigo”
- “acá coordino decisiones reales”
- “esto no es un calendario genérico”

### CTA principales

- Ver detalle del nodo
- Resolver conflicto
- Cancelar / reprogramar / aceptar / rechazar
- Ver merged view
- Prender/apagar timelines relacionados
- Abrir origen del momento
- Crear evento privado
- Volver a Home si quiero discovery

### Regla clave

**Timeline es timeline-first, graph-capable.**  
**El graph no se impone de entrada; emerge cuando hay relaciones relevantes.**

---

## 4.4 Studio

### Propósito

**Workspace comercial / provider side**

Studio es el espacio donde el usuario construye, administra y opera su lado comercial.

### Qué muestra

- creación de oferta
- publicación
- disponibilidad
- reglas
- solicitudes
- estado del negocio/oferta
- métricas operativas si aportan

### Qué debe lograr

Cuando el usuario entra, debe sentir:

- “este es mi espacio de trabajo”
- “acá organizo mi lado provider”
- “esto no es un admin panel genérico”

### CTA principales

- Crear
- Editar
- Publicar
- Ver solicitudes
- Configurar disponibilidad/reglas

### Regla clave

Studio no debe contaminar Home ni Timeline con vocabulario o CTAs fuera de lugar.

---

## 4.5 Profile

### Propósito

**Identidad + reputación + actividad + oferta**

Profile debe ser:
- una identidad visible
- una surface de relación
- una forma de entender a una persona o negocio
- una puerta a actividad, oferta y contexto

### Qué muestra

- quién es esta persona/negocio
- qué publica
- qué ofrece
- qué actividad tiene
- señales de reputación / confianza
- timeline fragments si aportan
- availability hints cuando aplique
- followers/following/relaciones cuando tengan sentido

### Qué debe lograr

Cuando el usuario entra a un Profile, debe sentir:

- “entiendo quién es”
- “entiendo qué hace”
- “entiendo por qué debería seguirlo, reservarle o explorar más”

### CTA principales

- Seguir
- Ver actividad
- Ver offerings/listings
- Reservar
- Abrir timeline/contexto si aplica

### Regla clave

Profile no debe ser un hub vacío ni una simple página de links.

---

# 5. Qué ve un usuario nuevo vs un usuario activo

## 5.1 Usuario nuevo

Un usuario nuevo:
- no sigue a nadie
- no tiene grafo personal
- no tiene historial fuerte
- no entiende todavía el sistema

### Decisión

Para usuario nuevo:
**Home debe comportarse como discovery-first**, pero sin perder la idea de línea de vida.

### Qué debería ver

- contenido relevante y vivo
- sugerencias
- oportunidades cerca
- perfiles/negocios/servicios destacados
- fragmentos públicos o compartidos del graph
- hints temporales suficientes para entender que este producto trata de vida + tiempo

### Objetivo

- activar curiosidad
- hacer sentir producto vivo
- introducir la promesa temporal sin abrumar
- invitar a seguir, reservar o explorar

---

## 5.2 Usuario activo

Un usuario activo:
- ya tiene follows
- ya tiene contexto
- ya tiene historial
- puede tener bookings, conflictos y relaciones temporales

### Decisión

Para usuario activo:
**Home puede volverse más personal y más timeline-connected.**

### Qué debería ver

- mi mini línea
- mis próximos nodos
- cosas que se conectan con mi tiempo
- actividad relevante
- oportunidades compatibles
- fragmentos de graph útiles
- señales de conflicto o coordinación cuando sea relevante

### Objetivo

- continuidad
- relevancia
- engagement
- decisión
- coordinación

---

# 6. Qué pertenece al Shell y qué a los MFEs

## 6.1 El Shell debe ser dueño de

- navegación top-level
- labels de surfaces
- layout global
- relación entre Home / Explore / Timeline / Studio / Profile
- composición macro de Home
- lógica de entrada del usuario (new vs active)
- product skeleton
- story of the product
- empty states top-level
- shell-level mock data orchestration si hace falta
- branded 404 / estructura general

## 6.2 Los MFEs deben ser dueños de

- rendering interno de sus surfaces
- páginas específicas
- composición local
- mock data local cuando convenga
- interacciones internas
- refinamientos de experiencia propios del dominio

## Regla general

**El shell cuenta la historia.  
El MFE implementa la surface.**

---

# 7. Navegación top-level

## Estructura base

Mantener:

- Home
- Timeline
- Studio
- Profile vía avatar

## Top nav

- logo / marca
- entrada a búsqueda
- notificaciones
- avatar → Profile
- eventualmente un acceso contextual al Timeline si se justifica

## Tabs principales

- Home
- Timeline
- Studio

## Discover / Explore

No exponer:
- Home
- Discover
- Explore

como tres conceptos top-level simultáneos.

### Decisión

- Home = top-level principal
- Explore = sub-surface clara / search-driven
- Discover puede quedar como naming técnico interno si hace falta

---

# 8. Home v2 — estructura propuesta

## Principio

Home debe sentirse como:
# **mi línea + cosas que se conectan con ella**

No solo una columna de cards.

## Estructura sugerida

### 8.1 Timeline glimpse / life glimpse
- mini línea
- próximos nodos
- moving avatar si suma
- una o dos conexiones entrantes relevantes

### 8.2 Search/discovery entry
- barra de búsqueda o entrada clara a Explore
- chips simples/contextuales si realmente ayudan

### 8.3 Connected opportunities
- listings / eventos / momentos / perfiles que encajan con mi contexto

### 8.4 Activity / public graph fragments
- momentos públicos
- cosas compartidas
- actividad relevante
- oportunidades cercanas o conectables

### 8.5 Soft recommendations
- perfiles
- negocios
- contextos
- lugares
- cosas “para seguir” o explorar

## Regla

Home debe sentirse:
- vivo
- personal
- conectado
- no sobre-estructurado
- no vacío
- no artificial

---

# 9. Explore v2 — estructura propuesta

## Principio

Explore debe ayudar a:
- buscar primero
- refinar después
- descubrir sin rigidez

## Estructura sugerida

### 9.1 Search-first entry
- input fuerte y claro
- resultados en “Todo” por defecto

### 9.2 Resultados simples
- personas / empresas / servicios / oportunidades
- hints temporales cuando agregan valor

### 9.3 Refinamiento progresivo
- tags
- filtros suaves
- no demasiada estructura upfront

## Regla

Explore no debe ser:
- dashboard
- panel rígido
- mini sistema inventado

Debe apoyarse en patrones conocidos y mejorados.

---

# 10. Timeline v2 — lectura funcional dentro del skeleton

Timeline debe quedar ya explícitamente alineado con el bloque documental nuevo.

## Timeline es:

- línea de vida/tiempo
- nodos como protagonistas
- sparse time compressed
- slot vs event claramente distintos
- shared moments
- resources as lines
- privacy-aware intersections
- graph emergent
- conflict as decision signal

## Home relation

Home puede mostrar:
- un vistazo de timeline
- cosas que se conectan a él

Timeline es donde:
- se entiende el tiempo a fondo
- se gestiona
- se compara
- se decide

---

# 11. Temporal entities as part of the skeleton

El skeleton v2 debe incorporar explícitamente esta verdad:

## El tiempo no pertenece solo a usuarios

También pueden ser entidades temporales:
- personas
- recursos
- lugares
- organizaciones
- family/group contexts
- listing-backed offerings cuando proyectan tiempo
- temporal containers

## Implicancia

Esto significa que:
- una cancha puede tener línea
- una sala puede tener línea
- una clínica puede coordinar múltiples líneas
- una banda puede converger en eventos
- una familia puede afectar una línea profesional
- un listing puede proyectar tiempo sin “ser” el dueño del tiempo

---

# 12. Mocked data strategy v2

## Propósito

La mock data sigue siendo la estrategia principal en esta etapa.

Pero ahora debe contar una historia mejor y más conectada.

## 12.1 Home mock data
- mini línea personal
- nodos próximos
- oportunidades conectadas
- actividad relevante
- fragmentos de graph públicos o compartidos
- contenido contextual (cerca, esta semana, etc.)

## 12.2 Explore mock data
- resultados realistas
- búsqueda simple
- hints temporales
- personas / negocios / ofertas / momentos

## 12.3 Timeline mock data
- sparse case
- dense case
- slot vs event
- shared event
- conflict
- resource line
- related timelines simple

## 12.4 Profile mock data
- identidad
- actividad
- oferta
- fragmentos temporales cuando sumen

## 12.5 Studio mock data
- oferta creada o en draft
- solicitudes
- reglas/disponibilidad básicas

---

# 13. Qué NO priorizar todavía

No conviene priorizar todavía:

- backend real
- wiring profundo de APIs
- algoritmos complejos
- normalización de toolkit
- design system perfecto
- micro-optimización técnica
- features secundarias no esenciales
- moderación profunda
- resolver toda la complejidad institucional en UI final

Primero hay que cerrar:
- navegación
- pages
- concept
- shells
- surfaces
- timeline language
- interaction model básico

---

# 14. Decisiones v2 propuestas

## Decisión 1
**Home = my line + connected things.**

## Decisión 2
**Explore = search-first, progressive refinement, no rigid dashboard.**

## Decisión 3
**Timeline = line of life/time, graph emergent.**

## Decisión 4
**Temporal entities are part of the core product truth.**

## Decisión 5
**The shell owns the narrative and top-level composition.**

## Decisión 6
**Page-first, toolkit-later in this stage.**

## Decisión 7
**Same stack, no unnecessary new technology.**

## Decisión 8
**Mock-first remains mandatory until the product shape is truly closed.**

---

# 15. Próximo deliverable recomendado

Después de este skeleton v2, el siguiente paso no debería ser backend.

El siguiente paso debería ser:

## A. Shell / layout / surface map
- layout general
- menús
- navegación
- top-level pages
- relación entre surfaces

## B. Home / Explore / Timeline page work
- pages necesarias
- relaciones
- entry flows
- placeholders vivos
- prototypes navegables

## C. recién después
- normalización
- toolkit alignment
- backend wiring

---

# 16. Criterio de éxito

Este skeleton v2 se considera exitoso cuando el usuario puede entrar a VyteMerge y sentir, en menos de 30 segundos:

- qué es Home
- qué es Explore
- qué es Timeline
- qué es Studio
- qué es Profile
- que el producto está vivo
- que descubre cosas relevantes
- que esas cosas pueden conectarse con su vida y tiempo
- que Timeline no es una agenda genérica
- que todo pertenece al mismo sistema

---

# 17. Frase síntesis del producto

**VyteMerge es una red social comercial centrada en tiempo, donde el usuario descubre cosas relevantes, las conecta con su línea de vida y coordina decisiones reales a través de entidades temporales, momentos compartidos y conflictos visibles.**

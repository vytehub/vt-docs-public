---
title: Tags
description: Identidad semántica para discovery, filtros y organización (global, public-by-profile y private).
---

# Tags

## TL;DR
**Tags** son el lenguaje semántico de VyteMerge para:
- **descubrimiento** (search, feed, marketplace)
- **filtro y organización** (listings, services, products, events)
- **contexto social** (channels, intereses)

> Los tags no son “strings sueltos”: se resuelven a un `TagId` para consistencia, analytics y evolución.

---

## Contexto / Objetivo
A medida que VyteMerge crece (servicios, productos, listings, channels), sin tags se vuelve difícil:
- encontrar lo que buscás
- filtrar resultados por necesidad
- mantener consistencia de categorías (“uñas”, “nails”, “manicura”…)
- construir recomendaciones en el futuro

Tags resuelve esto de forma simple y transversal.

---

## Alcance del módulo Tags (MVP)

### Incluye
- Definir Tags con identidad (`TagId`, `Slug`, `DisplayName`)
- Resolver entradas a tags (“resolve”)
- Soportar tags en distintos alcances (ver abajo)
- Asociar tags a entidades del sistema (Service/Product/Listing/Event/Channel/Profile)

### No incluye
- ❌ reglas de scheduling
- ❌ conflictos del timeline
- ❌ assignment de staff
- ❌ privacy/access
- ❌ ranking de marketplace (aunque se alimenta de tags)

---

## Conceptos clave

### Tag
Un Tag representa un concepto semántico.
Ejemplos:
- `#veterinaria`, `#masajes`, `#manicura`
- `#guardia`, `#a-domicilio`
- `#futbol5`, `#padres`, `#colegio`

Un Tag tiene dos caras:
- **Slug**: identificador estable y normalizado (técnico)
- **DisplayName**: nombre visible (editable)

---

## Tipos de Tags (los “3 tipos” que querías)

En VyteMerge conviene pensar los tags por **alcance**:

### 1) Global Tags (Platform)
- Curados o aprobados por la plataforma.
- Usados para marketplace y discovery amplio.
- Evitan explosión de sinónimos en búsqueda global.

Ej: `#veterinaria`, `#manicura`, `#fisioterapia`

**Uso típico**
- clasificación general de Services/Products/Listings
- filtros en search/marketplace

---

### 2) Public Tags (Profile-owned)
- Los crea un Profile y son **visibles públicamente** dentro de ese perfil/catálogo.
- Permiten que un vendedor use su propio “lenguaje” sin contaminar lo global.

Ej:
- Una estética crea `#promo-invierno`, `#vip`, `#nails-team`
- Un club crea `#torneo-lunes`, `#partido-abierto`

**Uso típico**
- filtros dentro del profile (catálogo del vendedor)
- campañas y comunicación interna de ese vendedor
- organización pública propia

> Esto es lo que en tu doc viejo llamabas “tags públicos propios”.

---

### 3) Private Tags (Personal)
- Tags privados del usuario (solo visibles para quien los crea).
- Sirven para organización personal y filtros privados.

Ej:
- `#no-reservar`, `#clientes-problema` (solo para mí)
- `#mejores-amigos` (si lo usás como segmentación personal)
- `#pendiente` (organización)

**Uso típico**
- filtrar eventos/bookings personales
- organizar contactos o notas internas (futuro)
- clasificar “a mi manera” sin exponerlo

> Esto es lo que en tu doc viejo llamabas “tags privados”.

---

## ¿Dónde se aplican los tags? (super importante)
Tags pueden asociarse a:

- **Service**: clasificación semántica del servicio (“masaje”, “consulta”)
- **Product**: clasificación del producto (“shampoo”, “alimento mascotas”)
- **Listing**: etiquetas específicas de esa oferta (“2x1”, “promo”, “a domicilio”)
- **Event**: etiquetas operativas (“acto escolar”, “guardia”, “viaje”)
- **Profile**: intereses o rubros del perfil (descubrimiento)
- **Channel**: temática del channel (“mascotas”, “futbol5”)

Regla útil:
- **Service/Product tags** = “qué es”
- **Listing tags** = “cómo lo vendo”
- **Event tags** = “qué está pasando”
- **Channel tags** = “de qué trata esta comunidad”

---

## Resolución (Resolve) vs Creación (Create)

### Principio
El flujo principal es **resolver**, no crear.

Cuando el usuario escribe “manicura”:
1) intentamos mapearlo a un tag existente (global o del profile según contexto)
2) si no existe, se crea **en el scope correcto** (según política)

Esto evita:
- duplicados
- tags similares con distinta ortografía
- caos en marketplace

---

## Reglas simples de UX (MVP)
- Autocomplete al escribir tags (según contexto)
- En marketplace: sugerir primero **Global**
- En profile: sugerir primero **Profile-owned**
- Para tags privados: creación rápida sin aprobación

---

## Relación con otros módulos
| Módulo | Cómo usa Tags |
|---|---|
| Catalog (Service/Product) | clasificación y búsqueda |
| Listings | filtros, promos, distribución (por contexto) |
| Search/Feed | ranking y relevancia (futuro) |
| Channels | tema/comunidad, discovery |
| Events/Bookings | organización y reporting |

---

## KPIs (para producto)
- top tags globales por semana (tendencias)
- tags más usados por categoría (services/products)
- ratio de “resolve vs create” (salud del sistema)
- uso de tags privados (retención/organización)

---

## Riesgos & Open Questions
- Moderación de tags globales (sinónimos, contenido inapropiado)
- ¿Cómo evitar spam en profile-owned tags?
- Multilenguaje: slug único vs display por idioma (futuro)
- Recomendaciones: cómo usar tags sin sesgos (futuro)

---

## Checklist / Próximos pasos
- [ ] Confirmar los 3 scopes: Global / Profile-owned / Private
- [ ] Definir autocomplete por contexto (marketplace vs profile vs privado)
- [ ] Definir dónde se almacenan tags por entidad (Service/Product/Listing/Event/Channel/Profile)
- [ ] Definir política de creación (quién puede crear qué y dónde)

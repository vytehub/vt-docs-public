---
title: Flow 00 — Registro y Primer Login
description: >
  El usuario crea una cuenta en VyteMerge via Keycloak (email+password o SSO).
  En el primer API call autenticado el backend crea el User en PostgreSQL,
  le asigna permisos básicos, y auto-genera las entidades fundacionales:
  Profile, Timeline default, Attendee y Customer.
status: draft
version: 1
---

# Flow 00 — Registro y Primer Login

## 1. Resumen
- **Goal:** que un nuevo usuario quede completamente registrado en VyteMerge con todas
  las entidades fundacionales listas para operar (Profile, Timeline, Attendee, Customer).
- **Actores:**
  - **Primary:** Nuevo usuario (futuro Business o Attendee).
  - **Secondary:** Keycloak (Identity Provider), JWT Middleware (backend).
- **Surfaces:** Página de registro/login de Keycloak → Shell (vt-shell) → Dashboard.

---

## 2. Domain Context

### Separación Identity / User / Profile

| Capa | Entidad | Propietario | Responsabilidad |
|------|---------|-------------|-----------------|
| **Identity** | Realm User (Keycloak) | Keycloak | AuthN/AuthZ, sesión, credenciales |
| **Account** | `User` | BC Foundation | Identidad persistente en PostgreSQL; permisos base (`Role.Member`) |
| **Social Identity** | `Profile` | BC Social-Discovery | Cómo el usuario se presenta al mundo; puede tener N Profiles |

Reglas clave:
- **1 Keycloak user → 1 User** en PostgreSQL (linked via `IdentityId = Keycloak.sub`).
- **1 User → 1 Profile personal** (auto-creado, privacy = Private).
- El usuario puede crear Profiles adicionales (organizaciones) después del registro.
- El `User.privacy = Private` por defecto; el Follow Request está bloqueado hasta que el usuario configure su Profile.

### Entidades auto-creadas (via `UserRegisteredIntegrationEvent`)

| Entidad | Módulo | Trigger | Descripción |
|---------|--------|---------|-------------|
| `User` | `Users` | JWT middleware (primer API call) | Identity persistente; `Role.Member`; `Privacy.Private` |
| `Profile` | `Social` | `UserRegisteredIntegrationEvent` | Profile personal por defecto; `name = firstName + lastName`; `privacy = Private` |
| `Timeline` | `Timelines` | `UserRegisteredIntegrationEvent` | Timeline personal por defecto; `isDefault = true`; `name = firstName + lastName` |
| `Attendee` | `Attendance` | `UserRegisteredIntegrationEvent` | Permite al usuario ser participante en Bookings/Events |
| `Customer` | `Ticketing` | `UserRegisteredIntegrationEvent` | Permite al usuario realizar compras/órdenes |

---

## 3. Preconditions
- No existe un `User` en PostgreSQL con el `IdentityId` del Keycloak sub.
- Keycloak tiene configurado el realm de VyteMerge y el cliente de vt-shell.
- El email del usuario no está registrado previamente (o está confirmado si el email ya existía).

---

## 4. Trigger

**Opción A — Email + Password:**
El usuario completa el formulario de registro de Keycloak y confirma su email.
En el primer API call desde vt-shell (con JWT válido), el middleware detecta que
el `IdentityId` no existe en PostgreSQL y dispara el registro.

**Opción B — SSO (Google, Apple — futuro):**
El usuario autoriza el proveedor SSO en Keycloak. Mismo comportamiento posterior:
primer API call → middleware → registro.

---

## 5. Main Flow

### Fase 1 — Registro en Keycloak (sin backend propio)

1. Usuario abre la pantalla de registro/login provista por Keycloak.
2. Completa: `email`, `password`, `firstName`, `lastName` (o autoriza SSO).
3. Keycloak crea un **Realm User** con un `sub` (Keycloak user ID único).
4. Keycloak emite el JWT con claims: `sub`, `email`, `given_name`, `family_name`.
5. vt-shell recibe el token y realiza el primer API call autenticado al backend.

### Fase 2 — Creación del User en PostgreSQL (JWT Middleware)

6. El JWT middleware del backend valida el token.
7. Middleware consulta PostgreSQL: ¿existe un `User` con `IdentityId = sub`? → **No**.
8. Middleware ejecuta el comando `RegisterUser` con los claims del JWT:
   - `email`, `firstName`, `lastName` del JWT
   - `identityId = Keycloak.sub`
   - `timeZoneId` del JWT claim (si existe) o `null` (se configura después)
9. El domain crea el `User`:
   - `Privacy = Private` (default)
   - `Role.Member` asignado
   - Raises: `UserRegisteredDomainEvent`
10. El dominio persiste el `User` en PostgreSQL (tabla `users.users`).
11. El outbox pattern publica el `UserRegisteredIntegrationEvent` al bus.

### Fase 3 — Auto-creación de entidades fundacionales (event-driven)

12. Los siguientes handlers reciben el `UserRegisteredIntegrationEvent` en paralelo:

    **Handler: Timelines module**
    - Ejecuta `CreateTimelineCommand(userId, fullName, "", isDefault: true)`
    - Persiste `Timeline` (schema: `timelines`) vinculada al `userId`

    **Handler: Social module** *(nuevo — a implementar)*
    - Ejecuta `CreateDefaultProfileCommand(userId, fullName, privacy: Private)`
    - Persiste `Profile` (schema: `social`) vinculada al `userId`

    **Handler: Attendance module**
    - Ejecuta `CreateAttendeeCommand(userId, email, firstName, lastName)`
    - Persiste `Attendee` (schema: `attendance`)

    **Handler: Ticketing module**
    - Ejecuta `CreateCustomerCommand(userId, email, firstName, lastName)`
    - Persiste `Customer` (schema: `ticketing`)

13. vt-shell recibe la respuesta del API call inicial y redirige al Dashboard.

---

## 6. Alternate / Edge Cases

| Caso | Comportamiento |
|------|---------------|
| Email ya registrado en Keycloak | Keycloak rechaza el registro; el backend no se involucra |
| JWT inválido o expirado | Middleware rechaza con 401; no se crea el User |
| User ya existe (`IdentityId` match) | Middleware detecta que ya existe → flujo normal de login; no re-crea nada |
| Fallo en la creación del User (DB error) | Transacción rollback; no se publican eventos; el usuario puede reintentar al próximo API call |
| Fallo parcial de un handler (ej: Timeline se crea, Profile no) | Cada handler reintenta de forma independiente (outbox pattern); los handlers son idempotentes |
| Usuario cancela SSO (OAuth authorization denied) | Keycloak maneja el error; el backend no recibe token |
| `timeZoneId` ausente en el JWT | `User.TimeZoneId = null`; se solicita al usuario en el perfil (NEEDS-CLARIFICATION: ¿obligatorio antes del primer use?) |
| Email no confirmado (si Keycloak lo requiere) | Keycloak bloquea el login hasta confirmar; el backend no recibe JWT válido |

---

## 7. Data Model (v1 minimal)

```
User {
  id:            UUID
  identityId:    string          -- Keycloak sub; único
  email:         string
  firstName:     string
  lastName:      string
  timeZoneId:    string?         -- IANA timezone; puede ser null en el registro
  privacy:       Private | Public -- default: Private
  roles:         Role[]          -- default: [Member]
  createdAt:     DateTime
}

Profile {                        -- auto-creado; 1 por User en v1
  id:            UUID
  userId:        UUID            -- FK User (owner)
  name:          string          -- default: firstName + lastName; editable
  bio:           string?
  avatarUrl:     string?
  privacy:       Private | Public -- default: Private (hereda de User)
  createdAt:     DateTime
  -- type: Personal (v1 only; Organization es un Profile adicional, no en registro)
}

Timeline {                       -- auto-creado; 1 por User
  id:            UUID
  userId:        UUID            -- FK User (owner)
  name:          string          -- default: firstName + lastName
  isDefault:     bool            -- true para el Timeline personal
  timeZoneId:    string?         -- hereda de User.timeZoneId si existe
  createdAt:     DateTime
}

Attendee {                       -- auto-creado; 1 por User
  userId:        UUID
  email:         string
  firstName:     string
  lastName:      string
}

Customer {                       -- auto-creado; 1 por User
  userId:        UUID
  email:         string
  firstName:     string
  lastName:      string
}
```

---

## 8. Commands

| Command | Aggregate / Módulo | Trigger |
|---------|--------------------|---------|
| `RegisterUser` | `User` / `Users` | JWT middleware (primer API call con `IdentityId` nuevo) |
| `CreateDefaultTimeline` | `Timeline` / `Timelines` | `UserRegisteredIntegrationEvent` |
| `CreateDefaultProfile` | `Profile` / `Social` | `UserRegisteredIntegrationEvent` *(nuevo)* |
| `CreateAttendee` | `Attendee` / `Attendance` | `UserRegisteredIntegrationEvent` |
| `CreateCustomer` | `Customer` / `Ticketing` | `UserRegisteredIntegrationEvent` |

---

## 9. Events

| Event | Tipo | Disparado por |
|-------|------|--------------|
| `UserRegisteredDomainEvent` | Domain | `User.Create()` en el aggregate |
| `UserRegisteredIntegrationEvent` | Integration | Outbox → publicado después del commit del `User` |
| `TimelineCreated` | Domain | `CreateDefaultTimeline` handler |
| `ProfileCreated` | Domain | `CreateDefaultProfile` handler *(nuevo)* |
| `AttendeeCreated` | Domain | `CreateAttendee` handler |
| `CustomerCreated` | Domain | `CreateCustomer` handler |

---

## 10. Invariants

1. Un `User` tiene exactamente un `IdentityId` (Keycloak sub); debe ser único en PostgreSQL.
2. Un `User` tiene exactamente un `email`; debe ser único.
3. El `Role.Member` se asigna siempre en la creación; no puede estar vacío.
4. `User.privacy = Private` en la creación; no puede omitirse.
5. Cada `User` tiene exactamente un Timeline con `isDefault = true`.
6. El `ProfileCreated` handler debe ser idempotente: si el Profile ya existe para el `userId`, no crea duplicado.
7. Los handlers de `UserRegisteredIntegrationEvent` son **independientes**: el fallo de uno no cancela los demás.
8. Un `User` no puede crearse sin `IdentityId` válido.
9. El middleware no re-ejecuta `RegisterUser` si el `User` ya existe para ese `IdentityId`.

---

## 11. Outputs

Al completar el flujo, el nuevo usuario tiene:
- `User` en PostgreSQL con `Role.Member` y `Privacy.Private`.
- `Profile` personal (privacy = Private; nombre = fullName).
- `Timeline` default (personal, isDefault = true).
- `Attendee` record (puede ser invitado a Events/Bookings).
- `Customer` record (puede realizar órdenes).
- Sesión activa en vt-shell con JWT válido.
- Redirección al Dashboard.

---

## 12. Technical Mapping (Draft)

### Backend

#### JWT Middleware — User Registration
- **Módulo:** `Users`
- **Behavior:** `JwtUserRegistrationBehavior` o `UserRegistrationMiddleware`
  - Extrae `sub`, `email`, `given_name`, `family_name` del JWT.
  - Consulta `UsersDbContext` por `IdentityId = sub`.
  - Si no existe → envía `RegisterUser` command via MediatR.
  - Si existe → no hace nada; el request continúa normalmente.
- **Comando:** `RegisterUserCommand(identityId, email, firstName, lastName, timeZoneId?)`
- **Handler:** `RegisterUserCommandHandler` → `User.Create()` → persiste → outbox event

#### Módulo: `Users`
- **DbContext:** `UsersDbContext` (schema: `users`)
- **Tabla:** `users.users`
- **Endpoints:**
  - `GET /users/me` → detalle del usuario autenticado

#### Módulo: `Social` *(handler nuevo a implementar)*
- **Archivo a crear:** `UserRegisteredIntegrationEventHandler.cs` en `Vt.Modules.Social.Presentation`
- **Comando:** `CreateDefaultProfileCommand(userId, name, privacy: Private)`

### Frontend (vt-shell)
- Keycloak SDK integrado para OIDC/OAuth2 (login, token refresh, logout).
- En el callback post-login: el shell realiza el primer API call autenticado → trigger del middleware.
- Si el backend retorna 401 (token inválido): redirigir a login.
- Una vez registrado: redirigir a Dashboard.
- **Pantalla de onboarding compleja (B2B onboarding):** fuera de scope v1 — ver `00-pending/index.md`.

---

## 13. Acceptance Criteria

- [ ] Un usuario puede registrarse con email + password vía Keycloak.
- [ ] En el primer API call autenticado, el backend crea el `User` en PostgreSQL.
- [ ] El `User` es creado con `Role.Member` y `Privacy.Private` por defecto.
- [ ] No se crea un `User` duplicado si el `IdentityId` ya existe.
- [ ] Al registrarse, se crea automáticamente un `Timeline` con `isDefault = true`.
- [ ] Al registrarse, se crea automáticamente un `Profile` personal con `privacy = Private`.
- [ ] Al registrarse, se crea automáticamente un `Attendee` record.
- [ ] El fallo de un handler de `UserRegisteredIntegrationEvent` no impide la creación de los demás.
- [ ] El usuario queda redirigido al Dashboard tras el primer login exitoso.
- [ ] `GET /users/me` retorna los datos del usuario autenticado.

---

## 14. NEEDS-CLARIFICATION

- **`timeZoneId` en el registro:** ¿es obligatorio antes del primer uso? ¿O se solicita lazily
  (la primera vez que el usuario intenta crear un Slot o Booking sin timezone configurado)?
- **Confirmación de email:** ¿Keycloak requiere confirmación de email antes de permitir login?
  ¿Es configurable por entorno (dev sin confirmación, prod con confirmación)?
- **Profile.name en v1:** ¿el default `firstName + lastName` es suficiente, o el usuario debe
  confirmar/editar el nombre del Profile antes de acceder al Dashboard?
- **SSO (Google/Apple):** ¿en scope v1 o solo email+password? Impacta la pantalla de registro.
- **B2B Onboarding post-registro:** después del primer login, ¿hay una pantalla de "¿eres un negocio?"
  que inicie el flujo de creación de Place/Service/Listing? Ver `00-pending/index.md`.
- **`UserRegisteredIntegrationEvent` estructura:** verificar que el evento incluye `FirstName` y
  `LastName` (ya confirmado en el handler de Timelines y Attendance), además de `UserId` y `Email`.

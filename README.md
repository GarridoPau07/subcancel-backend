# 💳 SubCancel

> **Plataforma multi-dispositivo para el control de suscripciones y gastos recurrentes, con alertas preventivas antes de cada cobro.**

![Java](https://img.shields.io/badge/Java-25-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-4.1.1-6DB33F?logo=springboot&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-1.9+-7F52FF?logo=kotlin&logoColor=white)
![Jetpack Compose](https://img.shields.io/badge/Jetpack_Compose-Material_3-4285F4?logo=jetpackcompose&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-4169E1?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![Postman](https://img.shields.io/badge/Postman-Collection-FF6C37?logo=postman&logoColor=white)
![OpenAPI](https://img.shields.io/badge/OpenAPI-3.0-85EA2D?logo=swagger&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## 📑 Tabla de Contenidos

1. [📌 Descripción del Proyecto](#-descripción-del-proyecto)
2. [🏛️ Arquitectura del Sistema](#️-arquitectura-del-sistema)
3. [🗄️ Diseño de Base de Datos](#️-diseño-de-base-de-datos)
4. [🔌 Especificación de la API REST](#-especificación-de-la-api-rest)
   - [4.1 Convenciones generales](#41-convenciones-generales)
   - [4.2 Módulo Auth](#42-módulo-auth--apiv1auth)
   - [4.3 Módulo Suscripciones](#43-módulo-suscripciones--apiv1subscriptions)
   - [4.4 Módulo Categorías](#44-módulo-categorías--apiv1categories)
5. [📂 Estructura del Proyecto y Repositorio](#-estructura-del-proyecto-y-repositorio)
6. [🚀 Guía de Instalación y Despliegue Local](#-guía-de-instalación-y-despliegue-local)
7. [🛣️ Roadmap (V2)](#️-roadmap-v2)
8. [📜 Licencia](#-licencia)

---

## 📌 Descripción del Proyecto

### Resumen ejecutivo

**SubCancel** es una solución integral que permite a los usuarios **centralizar, visualizar y controlar** todas sus suscripciones digitales (streaming, software, gimnasio, servicios en la nube, etc.) y gastos recurrentes. Su principal propuesta de valor es el **sistema de alertas preventivas**: la aplicación notifica al usuario *N* días antes de cada cobro, dándole la oportunidad de cancelar a tiempo y evitar cargos no deseados.

### 🎯 Objetivos

| Objetivo | Descripción |
|---|---|
| 🔍 **Visibilidad** | Vista unificada del gasto mensual y anual en suscripciones. |
| 🔔 **Prevención** | Recordatorios configurables antes de cada fecha de cobro. |
| 🗂️ **Organización** | Clasificación por categorías personalizables. |
| 📱 **Ubicuidad** | Acceso desde Android nativo y navegador web con datos sincronizados. |

### 🧩 Componentes de la plataforma

| Componente | Tecnología | Repositorio |
|---|---|---|
| 🖥️ **Backend API REST** | Java 25, Spring Boot 4.1.1, Spring Data JPA, Spring Security (JWT), OpenAPI | `subcancel-backend` |
| 📱 **App Móvil** | Kotlin, Jetpack Compose, Retrofit, WorkManager | `subcancel-android` |
| 🌐 **Cliente Web** | SPA con React (o HTML5 + JavaScript) | `subcancel-web` |
| 🗄️ **Base de Datos** | PostgreSQL 18 (alternativa: MySQL) | — |

### ✨ Funcionalidades principales

- ✅ Registro e inicio de sesión con autenticación **JWT**.
- ✅ CRUD completo de suscripciones y categorías.
- ✅ Cálculo automático de la **próxima fecha de cobro** según el ciclo de facturación.
- ✅ Notificaciones locales programadas con **WorkManager** (Android).
- ✅ Resumen de gasto mensual/anual agrupado por categoría.
- ✅ Documentación interactiva de la API con **Swagger UI**.

---

## 🏛️ Arquitectura del Sistema

El sistema sigue un **modelo cliente-servidor** donde los clientes (Android y Web) consumen una única **API RESTful** *stateless* protegida mediante tokens JWT.

### 📐 Diagrama de arquitectura

```text
┌────────────────────────────────────────────────────────────────────────────┐
│                               CAPA DE CLIENTES                             │
│                                                                            │
│   ┌──────────────────────────────┐      ┌──────────────────────────────┐   │
│   │      📱 APP ANDROID          │      │       🌐 CLIENTE WEB         │   │
│   │  Kotlin + Jetpack Compose    │      │   SPA (React / HTML5 + JS)   │   │
│   │  ┌────────────────────────┐  │      │                              │   │
│   │  │ ViewModel (MVVM)       │  │      │                              │   │
│   │  │ Retrofit + OkHttp      │  │      │   fetch / axios              │   │
│   │  │ WorkManager (alertas)  │  │      │                              │   │
│   │  └────────────────────────┘  │      │                              │   │
│   └──────────────┬───────────────┘      └───────────────┬──────────────┘   │
└──────────────────┼──────────────────────────────────────┼──────────────────┘
                   │         HTTPS  ·  JSON  ·  REST      │
                   │      Authorization: Bearer <JWT>     │
                   ▼                                      ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                           🖥️ BACKEND (Spring Boot)                         │
│                                                                            │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │  Security Filter Chain  →  JwtAuthenticationFilter  →  Validación  │   │
│   └───────────────────────────────┬────────────────────────────────────┘   │
│                                   ▼                                        │
│   ┌─────────────┐   ┌─────────────┐   ┌──────────────┐   ┌─────────────┐   │
│   │ Controller  │ → │   Service   │ → │  Repository  │ → │ Entity/Model│   │
│   │ (REST/DTO)  │   │ (Negocio)   │   │ (Spring Data)│   │   (JPA)     │   │
│   └─────────────┘   └─────────────┘   └──────┬───────┘   └─────────────┘   │
│                                              │                             │
│   ┌────────────────────────┐                 │      ┌───────────────────┐  │
│   │ Swagger UI / OpenAPI   │                 │      │ GlobalException   │  │
│   │ /swagger-ui.html       │                 │      │ Handler           │  │
│   └────────────────────────┘                 │      └───────────────────┘  │
└──────────────────────────────────────────────┼─────────────────────────────┘
                                               │  JDBC / Hibernate
                                               ▼
                              ┌──────────────────────────────────┐
                              │      🗄️ PostgreSQL / MySQL       │
                              │   users · categories ·           │
                              │   subscriptions                  │
                              └──────────────────────────────────┘
```

### 🔄 Flujo de datos

1. **Autenticación:** el cliente envía credenciales a `POST /api/v1/auth/login`. El backend valida el usuario, compara el hash de la contraseña (BCrypt) y devuelve un **access token** (corta duración) y un **refresh token** (larga duración).
2. **Peticiones protegidas:** el cliente adjunta el access token en la cabecera `Authorization: Bearer <token>`.
3. **Filtro de seguridad:** `JwtAuthenticationFilter` intercepta cada petición, valida firma y expiración, y carga el `SecurityContext` con la identidad del usuario.
4. **Procesamiento:** el `Controller` valida el DTO de entrada (`@Valid`), delega en el `Service` y este usa el `Repository` para persistir o consultar. **Toda consulta se filtra por el `user_id` del token**, garantizando el aislamiento de datos entre usuarios.
5. **Respuesta:** el `Service` devuelve entidades que se mapean a DTOs de salida; los errores se centralizan en un `@RestControllerAdvice`.
6. **Alertas (Android):** tras sincronizar, `WorkManager` programa tareas periódicas que calculan qué suscripciones vencen dentro de su ventana de aviso y lanzan notificaciones locales.

### 🔐 Modelo de seguridad (JWT)

| Aspecto | Decisión de diseño |
|---|---|
| **Algoritmo de firma** | HS256 (secreto ≥ 256 bits) o RS256 en producción. |
| **Access token** | Vida corta: 15 min. Contiene `sub` (userId), `email`, `iat`, `exp`. |
| **Refresh token** | Vida larga: 7 días. Permite renovar el access token sin reautenticar. |
| **Almacenamiento (Android)** | `EncryptedSharedPreferences` / Android Keystore. |
| **Almacenamiento (Web)** | Memoria + cookie `HttpOnly; Secure; SameSite=Strict` (recomendado). |
| **Contraseñas** | Hash con **BCrypt** (factor de coste 10–12). Nunca se almacenan en claro. |
| **Sesión** | `SessionCreationPolicy.STATELESS`; CSRF deshabilitado al usar Bearer tokens. |
| **Transporte** | HTTPS obligatorio fuera de entorno local. |
| **CORS** | Lista blanca de orígenes configurable por propiedades. |
| **Autorización** | Verificación de propiedad del recurso (`resource.userId == token.userId`). |

---

## 🗄️ Diseño de Base de Datos

### 🧭 Diagrama Entidad-Relación

```text
┌──────────────────┐         ┌──────────────────────┐         ┌────────────────────────┐
│      users       │ 1     N │      categories      │ 1     N │     subscriptions      │
├──────────────────┤─────────├──────────────────────┤─────────├────────────────────────┤
│ PK id            │         │ PK id                │         │ PK id                  │
│    email (UQ)    │         │ FK user_id           │         │ FK user_id             │
│    password_hash │         │    name              │         │ FK category_id         │
│    full_name     │         │    color_hex         │         │    name                │
│    currency      │         │    icon              │         │    amount              │
│    created_at    │         │    created_at        │         │    billing_cycle       │
│    updated_at    │         └──────────────────────┘         │    next_billing_date   │
└────────┬─────────┘                                          │    reminder_days_before│
         │ 1                                                  │    status              │
         └──────────────────────────────────── N ─────────────┤    ...                 │
                                                              └────────────────────────┘
```

### 👤 Tabla `users`

| Campo | Tipo de dato | Llave | Constraints | Descripción |
|---|---|---|---|---|
| `id` | `BIGSERIAL` / `BIGINT AUTO_INCREMENT` | 🔑 **PK** | `NOT NULL` | Identificador único del usuario. |
| `email` | `VARCHAR(150)` | — | `NOT NULL`, `UNIQUE` | Correo electrónico (login). |
| `password_hash` | `VARCHAR(255)` | — | `NOT NULL` | Hash BCrypt de la contraseña. |
| `full_name` | `VARCHAR(120)` | — | `NOT NULL` | Nombre completo. |
| `currency` | `CHAR(3)` | — | `NOT NULL`, `DEFAULT 'EUR'` | Moneda preferida (ISO 4217). |
| `role` | `VARCHAR(20)` | — | `NOT NULL`, `DEFAULT 'USER'` | Rol de seguridad (`USER`, `ADMIN`). |
| `created_at` | `TIMESTAMP` | — | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP` | Fecha de creación. |
| `updated_at` | `TIMESTAMP` | — | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP` | Última modificación. |

### 🏷️ Tabla `categories`

| Campo | Tipo de dato | Llave | Constraints | Descripción |
|---|---|---|---|---|
| `id` | `BIGSERIAL` / `BIGINT AUTO_INCREMENT` | 🔑 **PK** | `NOT NULL` | Identificador de la categoría. |
| `user_id` | `BIGINT` | 🔗 **FK** → `users.id` | `NOT NULL`, `ON DELETE CASCADE` | Propietario de la categoría. |
| `name` | `VARCHAR(60)` | — | `NOT NULL` | Nombre (ej. *Streaming*). |
| `color_hex` | `CHAR(7)` | — | `NOT NULL`, `DEFAULT '#6200EE'` | Color en formato `#RRGGBB`. |
| `icon` | `VARCHAR(40)` | — | `NULL` | Identificador de icono. |
| `created_at` | `TIMESTAMP` | — | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP` | Fecha de creación. |
| — | — | — | `UNIQUE (user_id, name)` | Evita categorías duplicadas por usuario. |

### 🔁 Tabla `subscriptions`

| Campo | Tipo de dato | Llave | Constraints | Descripción |
|---|---|---|---|---|
| `id` | `BIGSERIAL` / `BIGINT AUTO_INCREMENT` | 🔑 **PK** | `NOT NULL` | Identificador de la suscripción. |
| `user_id` | `BIGINT` | 🔗 **FK** → `users.id` | `NOT NULL`, `ON DELETE CASCADE` | Propietario. |
| `category_id` | `BIGINT` | 🔗 **FK** → `categories.id` | `NULL`, `ON DELETE SET NULL` | Categoría asignada. |
| `name` | `VARCHAR(100)` | — | `NOT NULL` | Nombre del servicio (ej. *Netflix*). |
| `description` | `VARCHAR(255)` | — | `NULL` | Notas adicionales. |
| `amount` | `NUMERIC(10,2)` | — | `NOT NULL`, `CHECK (amount > 0)` | Importe por ciclo. |
| `currency` | `CHAR(3)` | — | `NOT NULL`, `DEFAULT 'EUR'` | Moneda del cobro. |
| `billing_cycle` | `VARCHAR(15)` | — | `NOT NULL`, `CHECK IN ('WEEKLY','MONTHLY','QUARTERLY','YEARLY')` | Periodicidad. |
| `start_date` | `DATE` | — | `NOT NULL` | Fecha de inicio de la suscripción. |
| `next_billing_date` | `DATE` | — | `NOT NULL` | Próxima fecha de cobro. |
| `reminder_days_before` | `SMALLINT` | — | `NOT NULL`, `DEFAULT 3`, `CHECK (0..30)` | Días de antelación del aviso. |
| `status` | `VARCHAR(15)` | — | `NOT NULL`, `DEFAULT 'ACTIVE'`, `CHECK IN ('ACTIVE','PAUSED','CANCELLED')` | Estado actual. |
| `created_at` | `TIMESTAMP` | — | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP` | Fecha de creación. |
| `updated_at` | `TIMESTAMP` | — | `NOT NULL`, `DEFAULT CURRENT_TIMESTAMP` | Última modificación. |

**Índices recomendados**

```sql
CREATE INDEX idx_subscriptions_user        ON subscriptions (user_id);
CREATE INDEX idx_subscriptions_next_billing ON subscriptions (user_id, next_billing_date)
    WHERE status = 'ACTIVE';
CREATE INDEX idx_categories_user           ON categories (user_id);
```

### 🔗 Relaciones entre entidades

| Relación | Cardinalidad | Descripción |
|---|---|---|
| `users` → `categories` | **1 : N** | Un usuario define muchas categorías; cada categoría pertenece a un único usuario. |
| `users` → `subscriptions` | **1 : N** | Un usuario posee muchas suscripciones. |
| `categories` → `subscriptions` | **1 : N** | Una categoría agrupa muchas suscripciones; una suscripción tiene como máximo una categoría. |

> 📝 **Nota:** en el modelo actual **no existen relaciones N:M**. Si en el futuro se admitieran etiquetas múltiples por suscripción, se introduciría una tabla intermedia `subscription_tags`.

---

## 🔌 Especificación de la API REST

### 4.1 Convenciones generales

| Elemento | Valor |
|---|---|
| **Base URL (local)** | `http://localhost:8080` |
| **Prefijo** | `/api/v1` |
| **Formato** | `application/json; charset=UTF-8` |
| **Autenticación** | `Authorization: Bearer <access_token>` (excepto `/auth/register` y `/auth/login`) |
| **Fechas** | ISO 8601 (`yyyy-MM-dd` / `yyyy-MM-dd'T'HH:mm:ss'Z'`) |
| **Documentación interactiva** | `http://localhost:8080/swagger-ui.html` |

**Formato estándar de error**

```json
{
  "timestamp": "2026-10-17T10:15:30Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Descripción legible del error",
  "path": "/api/v1/subscriptions",
  "details": ["campo: motivo del error"]
}
```

**Códigos de estado utilizados**

| Código | Significado |
|---|---|
| `200 OK` | Operación exitosa. |
| `201 Created` | Recurso creado. |
| `204 No Content` | Eliminación exitosa. |
| `400 Bad Request` | Validación fallida o petición mal formada. |
| `401 Unauthorized` | Token ausente, inválido o expirado. |
| `403 Forbidden` | El recurso no pertenece al usuario. |
| `404 Not Found` | Recurso inexistente. |
| `409 Conflict` | Conflicto de unicidad (ej. email duplicado). |

---

### 4.2 Módulo Auth · `/api/v1/auth`

#### 🟢 `POST /api/v1/auth/register` — Registrar usuario

| Atributo | Valor |
|---|---|
| **Autenticación** | ❌ Pública |
| **Cabeceras** | `Content-Type: application/json` |
| **Parámetros** | Ninguno |

**Request Body**

```json
{
  "fullName": "Ana García",
  "email": "ana.garcia@example.com",
  "password": "S3gura!2026",
  "currency": "EUR"
}
```

**Response `201 Created`**

```json
{
  "id": 1,
  "fullName": "Ana García",
  "email": "ana.garcia@example.com",
  "currency": "EUR",
  "createdAt": "2026-10-17T10:15:30Z"
}
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:15:30Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Error de validación",
  "path": "/api/v1/auth/register",
  "details": [
    "email: debe ser una dirección de correo válida",
    "password: debe tener al menos 8 caracteres"
  ]
}
```

---

#### 🟢 `POST /api/v1/auth/login` — Iniciar sesión

| Atributo | Valor |
|---|---|
| **Autenticación** | ❌ Pública |
| **Cabeceras** | `Content-Type: application/json` |

**Request Body**

```json
{
  "email": "ana.garcia@example.com",
  "password": "S3gura!2026"
}
```

**Response `200 OK`**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.xxxxx",
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.yyyyy",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:16:02Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Credenciales inválidas",
  "path": "/api/v1/auth/login"
}
```

---

#### 🟢 `POST /api/v1/auth/refresh` — Renovar access token

| Atributo | Valor |
|---|---|
| **Autenticación** | ❌ Pública (requiere refresh token válido en el body) |

**Request Body**

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.yyyyy"
}
```

**Response `200 OK`**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIn0.zzzzz",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:20:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Refresh token inválido o expirado",
  "path": "/api/v1/auth/refresh"
}
```

---

#### 🔵 `GET /api/v1/auth/me` — Perfil del usuario autenticado

| Atributo | Valor |
|---|---|
| **Autenticación** | ✅ Bearer Token |
| **Cabeceras** | `Authorization: Bearer <accessToken>` |

**Response `200 OK`**

```json
{
  "id": 1,
  "fullName": "Ana García",
  "email": "ana.garcia@example.com",
  "currency": "EUR"
}
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:21:10Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token ausente o inválido",
  "path": "/api/v1/auth/me"
}
```

---

### 4.3 Módulo Suscripciones · `/api/v1/subscriptions`

> 🔒 Todos los endpoints requieren la cabecera `Authorization: Bearer <accessToken>`.

#### 📋 Resumen de endpoints

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/v1/subscriptions` | Lista paginada con filtros. |
| `GET` | `/api/v1/subscriptions/{id}` | Detalle de una suscripción. |
| `POST` | `/api/v1/subscriptions` | Crear suscripción. |
| `PUT` | `/api/v1/subscriptions/{id}` | Actualizar suscripción completa. |
| `PATCH` | `/api/v1/subscriptions/{id}/status` | Cambiar estado (activa/pausada/cancelada). |
| `DELETE` | `/api/v1/subscriptions/{id}` | Eliminar suscripción. |
| `GET` | `/api/v1/subscriptions/upcoming` | Cobros próximos en una ventana de días. |
| `GET` | `/api/v1/subscriptions/summary` | Resumen de gasto mensual/anual. |

---

#### 🔵 `GET /api/v1/subscriptions` — Listar suscripciones

| Atributo | Valor |
|---|---|
| **Cabeceras** | `Authorization: Bearer <accessToken>` |

**Query params**

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `status` | `string` | No | `ACTIVE`, `PAUSED`, `CANCELLED`. |
| `categoryId` | `long` | No | Filtra por categoría. |
| `page` | `int` | No | Página (por defecto `0`). |
| `size` | `int` | No | Tamaño de página (por defecto `20`). |
| `sort` | `string` | No | Ej. `nextBillingDate,asc`. |

**Ejemplo:** `GET /api/v1/subscriptions?status=ACTIVE&page=0&size=10&sort=nextBillingDate,asc`

**Response `200 OK`**

```json
{
  "content": [
    {
      "id": 12,
      "name": "Netflix",
      "description": "Plan Estándar",
      "amount": 13.99,
      "currency": "EUR",
      "billingCycle": "MONTHLY",
      "startDate": "2025-01-05",
      "nextBillingDate": "2026-11-05",
      "reminderDaysBefore": 3,
      "status": "ACTIVE",
      "category": { "id": 2, "name": "Streaming", "colorHex": "#E50914" }
    }
  ],
  "page": 0,
  "size": 10,
  "totalElements": 1,
  "totalPages": 1
}
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:30:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token expirado",
  "path": "/api/v1/subscriptions"
}
```

---

#### 🔵 `GET /api/v1/subscriptions/{id}` — Obtener detalle

| Atributo | Valor |
|---|---|
| **Cabeceras** | `Authorization: Bearer <accessToken>` |
| **Path param** | `id` (long) — identificador de la suscripción |

**Response `200 OK`**

```json
{
  "id": 12,
  "name": "Netflix",
  "description": "Plan Estándar",
  "amount": 13.99,
  "currency": "EUR",
  "billingCycle": "MONTHLY",
  "startDate": "2025-01-05",
  "nextBillingDate": "2026-11-05",
  "reminderDaysBefore": 3,
  "status": "ACTIVE",
  "category": { "id": 2, "name": "Streaming", "colorHex": "#E50914" },
  "createdAt": "2025-01-05T09:00:00Z",
  "updatedAt": "2026-10-05T09:00:00Z"
}
```

**Response `404 Not Found`**

```json
{
  "timestamp": "2026-10-17T10:31:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Suscripción con id 99 no encontrada",
  "path": "/api/v1/subscriptions/99"
}
```

---

#### 🟢 `POST /api/v1/subscriptions` — Crear suscripción

| Atributo | Valor |
|---|---|
| **Cabeceras** | `Authorization: Bearer <accessToken>`, `Content-Type: application/json` |

**Request Body**

```json
{
  "name": "Spotify Premium",
  "description": "Plan familiar",
  "amount": 17.99,
  "currency": "EUR",
  "billingCycle": "MONTHLY",
  "startDate": "2026-10-20",
  "reminderDaysBefore": 2,
  "categoryId": 3
}
```

**Response `201 Created`**

```json
{
  "id": 25,
  "name": "Spotify Premium",
  "description": "Plan familiar",
  "amount": 17.99,
  "currency": "EUR",
  "billingCycle": "MONTHLY",
  "startDate": "2026-10-20",
  "nextBillingDate": "2026-11-20",
  "reminderDaysBefore": 2,
  "status": "ACTIVE",
  "category": { "id": 3, "name": "Música", "colorHex": "#1DB954" }
}
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:32:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Error de validación",
  "path": "/api/v1/subscriptions",
  "details": [
    "amount: debe ser mayor que 0",
    "billingCycle: valor no permitido (WEEKLY, MONTHLY, QUARTERLY, YEARLY)"
  ]
}
```

---

#### 🟠 `PUT /api/v1/subscriptions/{id}` — Actualizar suscripción

| Atributo | Valor |
|---|---|
| **Cabeceras** | `Authorization: Bearer <accessToken>`, `Content-Type: application/json` |
| **Path param** | `id` (long) |

**Request Body**

```json
{
  "name": "Spotify Premium",
  "description": "Plan familiar (6 cuentas)",
  "amount": 19.99,
  "currency": "EUR",
  "billingCycle": "MONTHLY",
  "startDate": "2026-10-20",
  "reminderDaysBefore": 5,
  "categoryId": 3
}
```

**Response `200 OK`**

```json
{
  "id": 25,
  "name": "Spotify Premium",
  "description": "Plan familiar (6 cuentas)",
  "amount": 19.99,
  "currency": "EUR",
  "billingCycle": "MONTHLY",
  "nextBillingDate": "2026-11-20",
  "reminderDaysBefore": 5,
  "status": "ACTIVE",
  "category": { "id": 3, "name": "Música", "colorHex": "#1DB954" }
}
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:33:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Error de validación",
  "path": "/api/v1/subscriptions/25",
  "details": ["reminderDaysBefore: debe estar entre 0 y 30"]
}
```

---

#### 🟡 `PATCH /api/v1/subscriptions/{id}/status` — Cambiar estado

**Request Body**

```json
{
  "status": "CANCELLED"
}
```

**Response `200 OK`**

```json
{
  "id": 25,
  "status": "CANCELLED",
  "updatedAt": "2026-10-17T10:34:00Z"
}
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:34:30Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Estado no válido. Valores permitidos: ACTIVE, PAUSED, CANCELLED",
  "path": "/api/v1/subscriptions/25/status"
}
```

---

#### 🔴 `DELETE /api/v1/subscriptions/{id}` — Eliminar suscripción

| Atributo | Valor |
|---|---|
| **Cabeceras** | `Authorization: Bearer <accessToken>` |

**Response `204 No Content`** — *(sin cuerpo)*

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:35:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token ausente o inválido",
  "path": "/api/v1/subscriptions/25"
}
```

---

#### 🔵 `GET /api/v1/subscriptions/upcoming` — Próximos cobros

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `days` | `int` | No | Ventana de días hacia adelante (por defecto `7`, máx. `90`). |

**Ejemplo:** `GET /api/v1/subscriptions/upcoming?days=7`

**Response `200 OK`**

```json
[
  {
    "id": 12,
    "name": "Netflix",
    "amount": 13.99,
    "currency": "EUR",
    "nextBillingDate": "2026-10-20",
    "daysUntilCharge": 3
  }
]
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:36:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "El parámetro 'days' debe estar entre 1 y 90",
  "path": "/api/v1/subscriptions/upcoming"
}
```

---

#### 🔵 `GET /api/v1/subscriptions/summary` — Resumen de gasto

**Response `200 OK`**

```json
{
  "currency": "EUR",
  "activeSubscriptions": 8,
  "monthlyTotal": 86.45,
  "yearlyTotal": 1037.40,
  "byCategory": [
    { "category": "Streaming", "monthlyTotal": 38.97 },
    { "category": "Música",    "monthlyTotal": 17.99 },
    { "category": "Software",  "monthlyTotal": 29.49 }
  ]
}
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:37:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token expirado",
  "path": "/api/v1/subscriptions/summary"
}
```

---

### 4.4 Módulo Categorías · `/api/v1/categories`

> 🔒 Todos los endpoints requieren la cabecera `Authorization: Bearer <accessToken>`.

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/v1/categories` | Lista las categorías del usuario. |
| `GET` | `/api/v1/categories/{id}` | Detalle de una categoría. |
| `POST` | `/api/v1/categories` | Crear categoría. |
| `PUT` | `/api/v1/categories/{id}` | Actualizar categoría. |
| `DELETE` | `/api/v1/categories/{id}` | Eliminar categoría (las suscripciones quedan sin categoría). |

#### 🔵 `GET /api/v1/categories` — Listar categorías

**Response `200 OK`**

```json
[
  { "id": 2, "name": "Streaming", "colorHex": "#E50914", "icon": "movie" },
  { "id": 3, "name": "Música",    "colorHex": "#1DB954", "icon": "music_note" }
]
```

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:40:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token ausente o inválido",
  "path": "/api/v1/categories"
}
```

#### 🔵 `GET /api/v1/categories/{id}` — Obtener categoría

**Response `200 OK`**

```json
{ "id": 2, "name": "Streaming", "colorHex": "#E50914", "icon": "movie" }
```

**Response `404 Not Found`**

```json
{
  "timestamp": "2026-10-17T10:41:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "Categoría con id 50 no encontrada",
  "path": "/api/v1/categories/50"
}
```

#### 🟢 `POST /api/v1/categories` — Crear categoría

**Request Body**

```json
{
  "name": "Productividad",
  "colorHex": "#0078D4",
  "icon": "work"
}
```

**Response `201 Created`**

```json
{ "id": 7, "name": "Productividad", "colorHex": "#0078D4", "icon": "work" }
```

**Response `400 Bad Request`**

```json
{
  "timestamp": "2026-10-17T10:42:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Error de validación",
  "path": "/api/v1/categories",
  "details": ["colorHex: formato inválido, use #RRGGBB"]
}
```

#### 🟠 `PUT /api/v1/categories/{id}` — Actualizar categoría

**Request Body**

```json
{
  "name": "Productividad y Trabajo",
  "colorHex": "#0078D4",
  "icon": "business_center"
}
```

**Response `200 OK`**

```json
{ "id": 7, "name": "Productividad y Trabajo", "colorHex": "#0078D4", "icon": "business_center" }
```

**Response `409 Conflict`**

```json
{
  "timestamp": "2026-10-17T10:43:00Z",
  "status": 409,
  "error": "Conflict",
  "message": "Ya existe una categoría con el nombre 'Streaming'",
  "path": "/api/v1/categories/7"
}
```

#### 🔴 `DELETE /api/v1/categories/{id}` — Eliminar categoría

**Response `204 No Content`** — *(sin cuerpo)*

**Response `401 Unauthorized`**

```json
{
  "timestamp": "2026-10-17T10:44:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Token ausente o inválido",
  "path": "/api/v1/categories/7"
}
```

---

## 📂 Estructura del Proyecto y Repositorio

### 🖥️ Backend · `subcancel-backend` (Arquitectura en capas)

```text
subcancel-backend/
├── 📄 pom.xml
├── 📄 mvnw / mvnw.cmd
├── 📄 Dockerfile                          # (pendiente)
├── 📄 docker-compose.yml
├── 📄 .env.example                        # plantilla de variables de Docker
├── 📄 .gitignore
├── 📄 LICENSE
├── 📄 README.md
└── 📁 src/
    ├── 📁 main/
    │   ├── 📁 java/com/subcancel/
    │   │   ├── 📄 SubcancelbackendApplication.java
    │   │   │
    │   │   ├── 📁 config/
    │   │   │   ├── 📄 OpenApiConfig.java
    │   │   │   ├── 📄 CorsConfig.java
    │   │   │   └── 📄 AppProperties.java
    │   │   │
    │   │   ├── 📁 controller/
    │   │   │   ├── 📄 AuthController.java
    │   │   │   ├── 📄 SubscriptionController.java
    │   │   │   └── 📄 CategoryController.java
    │   │   │
    │   │   ├── 📁 service/
    │   │   │   ├── 📄 AuthService.java
    │   │   │   ├── 📄 SubscriptionService.java
    │   │   │   ├── 📄 CategoryService.java
    │   │   │   └── 📁 impl/
    │   │   │       ├── 📄 AuthServiceImpl.java
    │   │   │       ├── 📄 SubscriptionServiceImpl.java
    │   │   │       └── 📄 CategoryServiceImpl.java
    │   │   │
    │   │   ├── 📁 repository/
    │   │   │   ├── 📄 UserRepository.java
    │   │   │   ├── 📄 CategoryRepository.java
    │   │   │   └── 📄 SubscriptionRepository.java
    │   │   │
    │   │   ├── 📁 model/
    │   │   │   ├── 📄 User.java
    │   │   │   ├── 📄 Category.java
    │   │   │   ├── 📄 Subscription.java
    │   │   │   └── 📁 enums/
    │   │   │       ├── 📄 BillingCycle.java
    │   │   │       └── 📄 SubscriptionStatus.java
    │   │   │
    │   │   ├── 📁 dto/
    │   │   │   ├── 📁 request/
    │   │   │   │   ├── 📄 RegisterRequest.java
    │   │   │   │   ├── 📄 LoginRequest.java
    │   │   │   │   ├── 📄 SubscriptionRequest.java
    │   │   │   │   └── 📄 CategoryRequest.java
    │   │   │   └── 📁 response/
    │   │   │       ├── 📄 AuthResponse.java
    │   │   │       ├── 📄 SubscriptionResponse.java
    │   │   │       ├── 📄 CategoryResponse.java
    │   │   │       └── 📄 ErrorResponse.java
    │   │   │
    │   │   ├── 📁 mapper/
    │   │   │   └── 📄 SubscriptionMapper.java
    │   │   │
    │   │   ├── 📁 security/
    │   │   │   ├── 📄 SecurityConfig.java
    │   │   │   ├── 📄 JwtService.java
    │   │   │   ├── 📄 JwtAuthenticationFilter.java
    │   │   │   └── 📄 CustomUserDetailsService.java
    │   │   │
    │   │   └── 📁 exception/
    │   │       ├── 📄 GlobalExceptionHandler.java
    │   │       ├── 📄 ResourceNotFoundException.java
    │   │       └── 📄 DuplicateResourceException.java
    │   │
    │   └── 📁 resources/
    │       ├── 📄 application.yml
    │       ├── 📄 application-dev.yml.example   # plantilla (el real no se versiona)
    │       └── 📁 db/migration/          # Flyway (opcional)
    │           └── 📄 V1__init_schema.sql
    │
    └── 📁 test/java/com/subcancel/
        ├── 📁 controller/
        ├── 📁 service/
        └── 📁 repository/
```

### 📱 App Android · `subcancel-android` (Arquitectura MVVM)

```text
subcancel-android/
├── 📄 build.gradle.kts
├── 📄 settings.gradle.kts
└── 📁 app/
    ├── 📄 build.gradle.kts
    └── 📁 src/main/
        ├── 📄 AndroidManifest.xml
        └── 📁 java/com/subcancel/app/
            ├── 📄 SubCancelApp.kt                # Application class
            ├── 📄 MainActivity.kt
            │
            ├── 📁 data/                          # 💾 CAPA DE DATOS
            │   ├── 📁 remote/
            │   │   ├── 📄 ApiService.kt          # Interfaz Retrofit
            │   │   ├── 📄 RetrofitClient.kt
            │   │   ├── 📄 AuthInterceptor.kt     # Inyecta Bearer Token
            │   │   └── 📁 dto/
            │   │       ├── 📄 AuthDto.kt
            │   │       ├── 📄 SubscriptionDto.kt
            │   │       └── 📄 CategoryDto.kt
            │   ├── 📁 local/
            │   │   └── 📄 TokenStorage.kt        # EncryptedSharedPreferences
            │   ├── 📁 model/
            │   │   └── 📄 Subscription.kt        # Modelos de dominio
            │   └── 📁 repository/
            │       ├── 📄 AuthRepository.kt
            │       ├── 📄 SubscriptionRepository.kt
            │       └── 📄 CategoryRepository.kt
            │
            ├── 📁 ui/                            # 🎨 CAPA DE PRESENTACIÓN
            │   ├── 📁 theme/
            │   │   ├── 📄 Color.kt
            │   │   ├── 📄 Theme.kt
            │   │   └── 📄 Type.kt
            │   ├── 📁 navigation/
            │   │   └── 📄 NavGraph.kt
            │   ├── 📁 components/
            │   │   ├── 📄 SubscriptionCard.kt
            │   │   └── 📄 CategoryChip.kt
            │   ├── 📁 auth/
            │   │   ├── 📄 LoginScreen.kt
            │   │   ├── 📄 RegisterScreen.kt
            │   │   └── 📄 AuthViewModel.kt
            │   ├── 📁 subscriptions/
            │   │   ├── 📄 SubscriptionListScreen.kt
            │   │   ├── 📄 SubscriptionFormScreen.kt
            │   │   ├── 📄 SubscriptionDetailScreen.kt
            │   │   └── 📄 SubscriptionViewModel.kt
            │   └── 📁 dashboard/
            │       ├── 📄 DashboardScreen.kt
            │       └── 📄 DashboardViewModel.kt
            │
            ├── 📁 worker/                        # ⏰ TAREAS EN SEGUNDO PLANO
            │   ├── 📄 ReminderWorker.kt          # WorkManager: revisa próximos cobros
            │   ├── 📄 ReminderScheduler.kt
            │   └── 📄 NotificationHelper.kt
            │
            ├── 📁 di/                            # Inyección de dependencias (Hilt)
            │   └── 📄 AppModule.kt
            │
            └── 📁 util/
                ├── 📄 DateUtils.kt
                └── 📄 Resource.kt                # Sealed class: Success/Error/Loading
```

---

## 🚀 Guía de Instalación y Despliegue Local

### ✅ Requisitos previos

| Herramienta | Versión mínima | Verificación |
|---|---|---|
| ☕ **JDK** | 25 | `java -version` |
| 🐳 **Docker + Docker Compose** | 24.x / v2 | `docker --version` |
| 🤖 **Android Studio** | Koala (2024.1) o superior | — |
| 🟩 **Node.js** (cliente web) | 18 LTS | `node -v` |
| 🔧 **Git** | 2.30 | `git --version` |
| 📦 **Maven** | Incluido mediante el wrapper `mvnw` | `.\mvnw -version` |

### 1️⃣ Clonar los repositorios

```bash
git clone https://github.com/GarridoPau07/subcancel-backend.git
git clone https://github.com/GarridoPau07/subcancel-android.git
git clone https://github.com/GarridoPau07/subcancel-web.git
```

### 2️⃣ Configurar las variables de entorno

Los archivos con credenciales reales **no se versionan**. El repositorio incluye plantillas que debes copiar y editar:

```bash
# Linux / macOS / Git Bash
cp .env.example .env
cp src/main/resources/application-dev.yml.example src/main/resources/application-dev.yml
```

```powershell
# Windows PowerShell
Copy-Item .env.example .env
Copy-Item src\main\resources\application-dev.yml.example src\main\resources\application-dev.yml
```

Edita ambos archivos con tus credenciales. **Deben coincidir entre sí**: el usuario, la contraseña y el nombre de la base de datos del `.env` son los que debe usar la URL de `application-dev.yml`.

Archivo `.env.example`:

```env
POSTGRES_DB=subcancel_db
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password
```

### 3️⃣ Levantar la base de datos con Docker

Archivo `docker-compose.yml` (en la raíz de `subcancel-backend`):

```yaml
services:
  postgres:
    image: postgres:18-alpine
    container_name: subcancel-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - subcancel_pgdata:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  subcancel_pgdata:
```

```bash
docker compose up -d
docker compose ps        # Verifica que el contenedor esté "healthy"
```

> ⚠️ **PostgreSQL 18:** el volumen se monta en `/var/lib/postgresql` (no en `/var/lib/postgresql/data`, como en versiones anteriores).
>
> ⚠️ Las variables `POSTGRES_*` solo se aplican la **primera vez** que se crea el volumen. Si cambias usuario, contraseña o nombre de la base de datos, recrea el volumen con `docker compose down -v` (esto borra los datos).

Para comprobar la conexión:

```bash
docker exec -it subcancel-db psql -U <tu_usuario> -d subcancel_db
```

### 4️⃣ Configurar y ejecutar el Backend

Archivo `src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: subcancel-backend
  profiles:
    active: dev
```

Archivo `src/main/resources/application-dev.yml` (plantilla en `application-dev.yml.example`):

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/subcancel_db
    username: your_user
    password: your_password
  jpa:
    hibernate:
      ddl-auto: update        # En producción usar "validate" + Flyway
    properties:
      hibernate:
        format_sql: true
    open-in-view: false
```

Al implementar la autenticación JWT y Swagger se añadirá además esta configuración:

```yaml
app:
  jwt:
    secret: ${JWT_SECRET}
    access-expiration-ms: 900000        # 15 minutos
    refresh-expiration-ms: 604800000    # 7 días

springdoc:
  swagger-ui:
    path: /swagger-ui.html
```

Ejecución (con la base de datos de Docker levantada):

```bash
# Linux / macOS / Git Bash
export JWT_SECRET="un-secreto-largo-y-aleatorio-de-al-menos-32-caracteres"
./mvnw spring-boot:run
```

```powershell
# Windows PowerShell
$env:JWT_SECRET = "un-secreto-largo-y-aleatorio-de-al-menos-32-caracteres"
.\mvnw spring-boot:run
```

✅ Verificación:

- En el log aparece `The following 1 profile is active: "dev"` y `HikariPool-1 - Start completed`.
- API disponible en 👉 `http://localhost:8080`
- Swagger UI en 👉 `http://localhost:8080/swagger-ui.html` *(disponible cuando se añada OpenAPI)*

### 5️⃣ Ejecutar la App Android

1. Abre **Android Studio** → `File > Open` → selecciona la carpeta `subcancel-android`.
2. Espera a que finalice la sincronización de Gradle.
3. Configura la URL base del backend en `app/build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        // 10.0.2.2 apunta al localhost de tu máquina desde el emulador Android
        buildConfigField("String", "BASE_URL", "\"http://10.0.2.2:8080/api/v1/\"")
    }
}
```

4. Selecciona un emulador (API 26+) o un dispositivo físico y pulsa ▶️ **Run**.

> ⚠️ **Dispositivo físico:** sustituye `10.0.2.2` por la IP local de tu equipo (ej. `192.168.1.50`) y asegúrate de estar en la misma red. Para tráfico HTTP en desarrollo, habilita `usesCleartextTraffic` solo en el build de *debug*.

### 6️⃣ Ejecutar el Cliente Web *(opcional)*

```bash
cd subcancel-web
npm install
echo "VITE_API_URL=http://localhost:8080/api/v1" > .env.local
npm run dev
```

👉 Disponible en `http://localhost:5173`

### 🧪 Verificación rápida con `curl`

```bash
# Registro
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"fullName":"Ana García","email":"ana@example.com","password":"S3gura!2026","currency":"EUR"}'

# Login
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ana@example.com","password":"S3gura!2026"}'
```

---

## 🛣️ Roadmap (V2)

| Prioridad | Característica | Descripción |
|---|---|---|
| 🥇 | 📧 **Detección automática de suscripciones** | Lectura de correos de facturación (OAuth con Gmail/Outlook) y extractos bancarios para detectar cobros recurrentes sin entrada manual. |
| 🥈 | 🔔 **Notificaciones push multicanal** | Integración con **Firebase Cloud Messaging**, correo y Telegram para alertas fuera de la app, con horarios y canales configurables. |
| 🥉 | 📊 **Analítica y recomendaciones con IA** | Informes de gasto histórico, detección de servicios duplicados o infrautilizados y sugerencias de ahorro. |
| 4️⃣ | 👨‍👩‍👧 **Suscripciones compartidas** | División de costes entre usuarios (planes familiares) con invitaciones y liquidación de saldos. |
| 5️⃣ | 💱 **Multimoneda y exportación** | Conversión automática de divisas y exportación de informes a CSV/PDF. |

---

## 📜 Licencia

Distribuido bajo la licencia **MIT**. Consulta el archivo `LICENSE` para más información.

---

<p align="center">Hecho con ☕ y 💜 · <strong>SubCancel</strong> — <em>Tú decides cuándo dejar de pagar.</em></p>

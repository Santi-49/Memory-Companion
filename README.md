# 🧠 MemoryCompanion — Documentación del Proyecto

> Una aplicación para personas mayores que usa Inteligencia Artificial para ayudar a recordar eventos, personas y momentos importantes de su vida.

---

## Tabla de Contenidos

1. [¿Qué es la app?](#1-qué-es-la-app)
2. [¿Cómo se usa?](#2-cómo-se-usa)
3. [Funcionalidades](#3-funcionalidades)
4. [Tech Stack](#4-tech-stack)
5. [Modelo de Datos](#5-modelo-de-datos)
6. [Diseño del Sistema de IA](#6-diseño-del-sistema-de-ia)
7. [API — Endpoints](#7-api--endpoints)
8. [Cómo se conecta todo](#8-cómo-se-conecta-todo)
9. [Flujo de trabajo con GitHub](#9-flujo-de-trabajo-con-github)
10. [Deployment](#10-deployment)

---

## 1. ¿Qué es la app?

**MemoryCompanion** es una aplicación móvil pensada para personas mayores. Su función principal es ser un **compañero de conversación inteligente** que:

- Recuerda a las personas importantes de la vida del usuario
- Guarda recuerdos, anécdotas y eventos vividos
- Gestiona recordatorios (citas médicas, cumpleaños, etc.)
- Recupera memorias de forma proactiva ("¿Sabes que hoy hace 10 años que viajaste a Roma?")

La interacción principal es **por voz o texto**, a través de un botón grande y visible en la pantalla. El usuario habla con la IA como si hablara con alguien de confianza.

Existe también un **rol de cuidador** (familiar o persona de apoyo) que puede acceder a un panel para revisar el diario y los recordatorios, sin interferir en la conversación del usuario.

---

## 2. ¿Cómo se usa?

### Usuario principal (persona mayor)

1. Abre la app y ve un **botón grande** (rojo) en el centro de la pantalla
2. Pulsa y habla o escribe: "¿Qué hacía yo el verano pasado?" / "Recuérdame que tengo médico el martes"
3. La IA responde en lenguaje natural y, si detecta información nueva (una persona, un recuerdo, un recordatorio), la **guarda automáticamente** en segundo plano
4. La app puede enviar **notificaciones proactivas**: "Buenos días, María. Hoy es el cumpleaños de tu nieto Carlos."

### Cuidador (familiar o apoyo)

1. Accede con su propia cuenta vinculada al perfil del usuario
2. Puede ver el **diario de memorias**, los **recordatorios** y el **listado de personas**
3. Puede añadir información manualmente (personas, eventos pasados) para enriquecer la memoria del sistema
4. No puede ver el contenido literal de las conversaciones (privacidad del usuario)

---

## 3. Funcionalidades

### Para el usuario
- 💬 **Conversación natural** con la IA sobre el pasado, el presente y recordatorios
- 📖 **Diario automático**: la IA extrae y guarda recuerdos de las conversaciones
- 👥 **Gestión de personas**: recordar quién es quién, qué relación tienen, cómo contactarlos
- 🔔 **Recordatorios y alertas**: el sistema envía notificaciones en el momento adecuado
- 🎲 **Memorias proactivas**: la IA recupera recuerdos del pasado y los comparte espontáneamente

### Para el cuidador
- 👁️ **Panel de revisión**: ver diario, personas y recordatorios
- ✏️ **Edición manual**: añadir personas o eventos que el usuario no ha mencionado
- 🔔 **Alertas de cuidador**: recibir notificaciones si hay cambios relevantes

### Técnicas (internas)
- **RAG**: búsqueda semántica en memorias, personas y recordatorios → [¿Qué es RAG?](./docs/GUIDE_RAG.md)
- **Extracción proactiva**: el agente detecta y persiste nueva información automáticamente → [¿Qué es un agente?](./docs/GUIDE_AI_AGENT.md)
- **Tareas asíncronas**: guardar y buscar en segundo plano para que la conversación sea fluida → [¿Qué es async?](./docs/GUIDE_ASYNC.md)

---

## 4. Tech Stack

> 📖 Para entender qué hace cada tecnología y cómo se relacionan → [GUIDE_TECH_STACK.md](./docs/GUIDE_TECH_STACK.md)
> 📖 Para entender Python, cómo se organiza el código y cómo funciona FastAPI → [GUIDE_PYTHON.md](./docs/GUIDE_PYTHON.md)

| Capa | Tecnología | Para qué |
|---|---|---|
| App móvil | React Native + Expo Go | La app que usa el usuario en su teléfono |
| Servidor | FastAPI (Python) | Procesa la lógica y conecta con la IA |
| Orquestación | Docker Compose | Levanta todos los servicios juntos |
| Base de datos | PostgreSQL + pgvector | Guarda toda la información + búsqueda semántica |
| IA | Claude API (Anthropic) | El modelo de lenguaje que hace la conversación |
| Protocolo IA | MCP (tools) | Permite al agente buscar y guardar datos |
| Tareas async | Celery + Redis | Procesa tareas en segundo plano |
| Notificaciones | Expo Push + APScheduler | Envía alertas en el momento correcto |
| Túnel / acceso | Cloudflare Tunnel | Expone el servidor local a internet |

---

## 5. Modelo de Datos

> 📖 Para entender qué es una base de datos, tablas, columnas y tipos de datos → [GUIDE_DATABASE.md](./docs/GUIDE_DATABASE.md)

---

### `accounts` — Cuentas de usuario

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `name` | TEXT | Nombre completo |
| `email` | TEXT | Correo electrónico (único) |
| `phone` | TEXT | Teléfono |
| `role` | ENUM | `user` o `caregiver` |
| `linked_user_id` | UUID (FK → accounts) | Si es cuidador, a qué usuario está vinculado |
| `push_token` | TEXT | Token para notificaciones push |
| `created_at` | TIMESTAMP | Fecha de creación |

---

### `conversations` — Conversaciones

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `account_id` | UUID (FK → accounts) | A qué usuario pertenece |
| `started_at` | TIMESTAMP | Cuándo empezó |
| `ended_at` | TIMESTAMP | Cuándo terminó (null si en curso) |
| `summary` | TEXT | Resumen generado por la IA al finalizar |
| `summary_embedding` | VECTOR(1536) | Embedding del resumen para búsqueda semántica |

---

### `messages` — Mensajes individuales

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `conversation_id` | UUID (FK → conversations) | A qué conversación pertenece |
| `role` | ENUM | `user` o `assistant` |
| `content` | TEXT | Contenido del mensaje |
| `created_at` | TIMESTAMP | Timestamp del mensaje |

---

### `diary_entries` — Diario de memorias

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `account_id` | UUID (FK → accounts) | Propietario del recuerdo |
| `content` | TEXT | Descripción completa del recuerdo |
| `summary` | TEXT | Resumen corto generado por la IA |
| `category` | ENUM | `personal`, `family`, `travel`, `health`, `work`, `social`, `other` |
| `source` | ENUM | `conversation` o `manual` |
| `date_granularity` | ENUM | `year`, `month`, `day`, `datetime` |
| `date_year` | INTEGER | Año (null si desconocido) |
| `date_month` | INTEGER | Mes 1-12 (null si desconocido) |
| `date_day` | INTEGER | Día 1-31 (null si desconocido) |
| `date_time` | TIME | Hora exacta (null si desconocido) |
| `content_embedding` | VECTOR(1536) | Embedding del contenido |
| `summary_embedding` | VECTOR(1536) | Embedding del resumen |
| `created_at` | TIMESTAMP | Cuándo se guardó |

> **¿Por qué tantos campos de fecha?** Las memorias raramente tienen fecha exacta. "Fue un verano de los 80" se guarda con `date_year=1982, date_granularity=year`. Así se puede filtrar aunque la fecha sea parcial.

---

### `people` — Personas conocidas

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `account_id` | UUID (FK → accounts) | A qué usuario pertenece |
| `name` | TEXT | Nombre completo |
| `nickname` | TEXT | Apodo o nombre por el que el usuario les llama |
| `phone` | TEXT | Teléfono |
| `email` | TEXT | Correo |
| `relationship` | TEXT | Descripción libre ("mi vecino del 3º que tiene un perro") |
| `category` | ENUM | `direct_family`, `family`, `friend`, `acquaintance`, `other` |
| `notes` | TEXT | Notas adicionales |
| `name_embedding` | VECTOR(1536) | Embedding del nombre y notas |
| `created_at` | TIMESTAMP | Cuándo se añadió |

---

### `people_diary` — Relación Personas ↔ Diario

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `person_id` | UUID (FK → people) | La persona |
| `diary_entry_id` | UUID (FK → diary_entries) | La entrada del diario |

---

### `reminders` — Recordatorios

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `account_id` | UUID (FK → accounts) | A quién pertenece |
| `title` | TEXT | Nombre corto ("Médico de cabecera") |
| `description` | TEXT | Descripción detallada |
| `remind_at` | TIMESTAMP | Cuándo enviar la notificación |
| `recurrence` | ENUM | `none`, `daily`, `weekly`, `monthly`, `yearly` |
| `is_sent` | BOOLEAN | Si ya se envió |
| `source` | ENUM | `conversation` o `manual` |
| `created_at` | TIMESTAMP | Cuándo se creó |

---

### `people_reminders` — Relación Personas ↔ Recordatorios

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | UUID (PK) | Identificador único |
| `person_id` | UUID (FK → people) | La persona |
| `reminder_id` | UUID (FK → reminders) | El recordatorio |

---

### Diagrama de relaciones

```
accounts
  │
  ├──── conversations ──── messages
  │
  ├──── diary_entries ──── people_diary ──── people
  │                                              │
  ├──── reminders ────── people_reminders ───────┘
  │
  └──── (linked_user_id → accounts) [cuidador]
```

---

## 6. Diseño del Sistema de IA

> 📖 Para entender qué es un agente, tools, inferencia y TTS → [GUIDE_AI_AGENT.md](./docs/GUIDE_AI_AGENT.md)
> 📖 Para entender RAG y embeddings → [GUIDE_RAG.md](./docs/GUIDE_RAG.md)
> 📖 Para entender tareas asíncronas → [GUIDE_ASYNC.md](./docs/GUIDE_ASYNC.md)

### Arquitectura: Agente Único con Tools (MCP)

Se usa un único agente conversacional con acceso a herramientas. El agente puede buscar en la base de datos, guardar información nueva, y gestionar recordatorios — todo de forma transparente para el usuario.

### El problema de la latencia

Los agentes necesitan ser rápidos. Guardar información, generar embeddings o hacer búsquedas puede tardar segundos.

**Solución: tareas asíncronas**

```
Usuario → App → FastAPI → Agente IA  →  Respuesta rápida al usuario
                    └──→ Celery Worker  →  Guardar en BD (en segundo plano)
```

### Herramientas del Agente

**Lectura (búsqueda):**

| Tool | Qué hace |
|---|---|
| `search_people` | Busca personas por nombre, categoría o relación |
| `search_diary` | Busca memorias por fecha, contenido semántico, categoría o personas |
| `search_reminders` | Busca recordatorios por fecha, contenido o personas |
| `get_random_memory` | Obtiene una memoria aleatoria (para recordar momentos espontáneamente) |

**Escritura:**

| Tool | Qué hace |
|---|---|
| `create_diary_entry` | Guarda un recuerdo nuevo |
| `update_diary_entry` | Actualiza un recuerdo existente |
| `create_person` | Añade una persona nueva |
| `update_person` | Actualiza datos de una persona |
| `create_reminder` | Crea un recordatorio |
| `delete_reminder` | Elimina un recordatorio |

### Instrucciones del Agente (System Prompt)

El agente opera con instrucciones permanentes:

1. **Rol y tono**: compañero amable, paciente, lenguaje simple adaptado a personas mayores
2. **Extracción proactiva**: al final de cada respuesta, analiza si se mencionó algo nuevo (persona, recuerdo, recordatorio) y lo guarda de forma asíncrona
3. **Memorias espontáneas**: periódicamente llama a `get_random_memory` para compartir un recuerdo del pasado
4. **Contexto de sesión**: al iniciar conversación, recibe un resumen del usuario (últimas conversaciones, próximos recordatorios, personas frecuentes)

### Notificaciones Proactivas

Un proceso separado (APScheduler) revisa periódicamente:
- Recordatorios del día → envía notificación push
- Cumpleaños de personas → mensaje de aviso
- Selección semanal de memoria aleatoria → "Esta semana de hace X años..."

---

## 7. API — Endpoints

> 📖 Para entender qué es una API, HTTP, JSON y autenticación → [GUIDE_API.md](./docs/GUIDE_API.md)

La API está construida con FastAPI y se documenta automáticamente en `http://localhost:8000/docs` durante el desarrollo.

Todos los endpoints (excepto login/registro) requieren autenticación con token JWT:
```
Authorization: Bearer <token>
```

---

### Auth

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/auth/register` | Registrar nueva cuenta |
| `POST` | `/auth/login` | Iniciar sesión, devuelve token JWT |
| `POST` | `/auth/link-caregiver` | Vincular cuenta de cuidador a un usuario |

---

### Conversaciones

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/conversations/` | Iniciar nueva conversación |
| `POST` | `/conversations/{id}/messages` | Enviar mensaje al agente y recibir respuesta |
| `GET` | `/conversations/` | Listar conversaciones del usuario |
| `GET` | `/conversations/{id}` | Ver detalle de una conversación |
| `PATCH` | `/conversations/{id}/end` | Finalizar conversación (dispara resumen async) |

---

### Diario

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/diary/` | Listar entradas (filtros: fecha, categoría, persona, búsqueda semántica) |
| `GET` | `/diary/{id}` | Ver entrada específica |
| `POST` | `/diary/` | Crear entrada manualmente (cuidador) |
| `PATCH` | `/diary/{id}` | Actualizar entrada |
| `DELETE` | `/diary/{id}` | Eliminar entrada |

---

### Personas

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/people/` | Listar personas (filtros: categoría, búsqueda semántica) |
| `GET` | `/people/{id}` | Ver persona específica |
| `POST` | `/people/` | Crear persona |
| `PATCH` | `/people/{id}` | Actualizar persona |
| `DELETE` | `/people/{id}` | Eliminar persona |

---

### Recordatorios

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/reminders/` | Listar recordatorios |
| `POST` | `/reminders/` | Crear recordatorio |
| `PATCH` | `/reminders/{id}` | Actualizar recordatorio |
| `DELETE` | `/reminders/{id}` | Eliminar recordatorio |

---

### Notificaciones

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/notifications/push-token` | Registrar token de dispositivo |
| `GET` | `/notifications/history` | Historial de notificaciones enviadas |

---

### Cuidador

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/caregiver/user/{user_id}/diary` | Ver diario del usuario vinculado |
| `GET` | `/caregiver/user/{user_id}/people` | Ver personas del usuario vinculado |
| `GET` | `/caregiver/user/{user_id}/reminders` | Ver recordatorios del usuario vinculado |
| `POST` | `/caregiver/user/{user_id}/diary` | Añadir entrada manual |
| `POST` | `/caregiver/user/{user_id}/people` | Añadir persona al perfil del usuario |

---

## 8. Cómo se conecta todo

```
┌─────────────────────────────────────────────────────────┐
│                  DISPOSITIVO MÓVIL                      │
│               (React Native / Expo Go)                  │
│  [Botón Rojo] → Usuario habla → POST /messages         │
└────────────────────────┬────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────┐
│                 BACKEND (FastAPI)                        │
│                                                         │
│  1. Recibe mensaje + historial de conversación          │
│  2. Llama al Agente IA con contexto del usuario         │
│                                                         │
│     ┌─────────────────────────────────┐                 │
│     │        AGENTE (Claude API)      │                 │
│     │  Si necesita info → llama tool  │                 │
│     │  FastAPI ejecuta la tool        │                 │
│     │  (consulta PostgreSQL/pgvector) │                 │
│     │  Recibe resultado → responde    │                 │
│     └─────────────────────────────────┘                 │
│                                                         │
│  3. Devuelve respuesta (< 3 segundos)                   │
│  4. Delega a Celery: guardar datos detectados           │
└────────────────────────┬────────────────┬───────────────┘
                         │                │ async
              Respuesta  ▼                ▼
                    App móvil      Celery Worker
                    (muestra       (genera embeddings,
                    respuesta)      guarda en BD)
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │ PostgreSQL + pgvector│
                              └─────────────────────┘

Proceso paralelo (APScheduler):
┌──────────────────────────────┐
│  Cada hora: revisa reminders │
│  Envía push notifications    │
│  via Expo Push Service       │
└──────────────────────────────┘
```

---

## 9. Flujo de trabajo con GitHub

> 📖 Para entender qué es Git y GitHub → [GUIDE_GITHUB.md](./docs/GUIDE_GITHUB.md)

### Estructura del repositorio

```
/
├── mobile/                  # App React Native (Expo)
│   ├── app/                 # Pantallas
│   ├── components/          # Componentes reutilizables
│   └── services/            # Llamadas a la API
│
├── backend/                 # Servidor FastAPI
│   ├── app/
│   │   ├── api/             # Endpoints (routers)
│   │   ├── agent/           # Lógica del agente + tools MCP
│   │   ├── models/          # Modelos de BD (SQLAlchemy)
│   │   ├── schemas/         # Validación (Pydantic)
│   │   ├── workers/         # Tareas Celery
│   │   └── scheduler/       # Notificaciones programadas
│   ├── docker-compose.yml
│   └── Dockerfile
│
└── docs/                    # Esta carpeta
    ├── PROYECTO.md          ← este archivo
    ├── GUIDE_API.md
    ├── GUIDE_DATABASE.md
    ├── GUIDE_DOCKER.md
    ├── GUIDE_RAG.md
    ├── GUIDE_AI_AGENT.md
    ├── GUIDE_ASYNC.md
    ├── GUIDE_DEPLOYMENT.md
    ├── GUIDE_TECH_STACK.md
    └── GUIDE_GITHUB.md
```

### Ramas

| Rama | Propósito |
|---|---|
| `main` | Código estable |
| `develop` | Integración de features |
| `feature/xxx` | Una funcionalidad concreta |

---

## 10. Deployment

> 📖 Para entender todo el proceso de deployment, costes y cómo llega la app a iOS y Android → [GUIDE_DEPLOYMENT.md](./docs/GUIDE_DEPLOYMENT.md)

### Resumen rápido

| Fase | Estado | Descripción |
|---|---|---|
| **Local** | ✅ Primera fase | Todo en el mismo ordenador, Expo Go para la app |
| **Túnel Cloudflare** | ✅ Primera fase | Acceso externo sin servidor, dominio gratuito temporal |
| **Servidor 24/7** | 🔜 Segunda fase | Servidor siempre encendido con dominio propio |
| **App Stores** | 🔜 Futuro | Publicar en Google Play y App Store |

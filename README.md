# MemoryCompanion

> Aplicacion movil para personas mayores que usa IA conversacional para ayudar a recordar personas, eventos, rutinas y momentos importantes.

MemoryCompanion esta pensada como un compañero sencillo de voz o texto. El usuario puede preguntar por recuerdos, crear recordatorios, guardar anecdotas y recibir avisos proactivos. La misma cuenta familiar incluye tambien cuidadores, que pueden revisar y editar diario, personas y recordatorios.

## Documentacion Principal

| Documento                                                | Contenido                                                                   |
| -------------------------------------------------------- | --------------------------------------------------------------------------- |
| [docs/PRODUCT_DECISIONS.md](./docs/PRODUCT_DECISIONS.md) | Alcance MVP, roles, permisos, experiencia de uso y decisiones de producto   |
| [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)           | Arquitectura del sistema, servicios, Redis, scheduler y flujo de peticiones |
| [docs/DATA_MODEL.md](./docs/DATA_MODEL.md)               | Modelo de datos propuesto, fechas parciales, ubicaciones y recurrencias     |
| [docs/AI_DESIGN.md](./docs/AI_DESIGN.md)                 | Diseño del agente, tools, RAG, embeddings y proveedor de IA                 |
| [docs/PROJECT_RISKS.md](./docs/PROJECT_RISKS.md)         | Riesgos principales del proyecto y mitigaciones propuestas                  |
| [docs/GUIDE_TECH_STACK.md](./docs/GUIDE_TECH_STACK.md)   | Guia educativa del stack tecnico                                            |
| [docs/GUIDE_API.md](./docs/GUIDE_API.md)                 | Conceptos de API, HTTP, JSON y autenticacion                                |
| [docs/GUIDE_DEPLOYMENT.md](./docs/GUIDE_DEPLOYMENT.md)   | Desarrollo local, tunel y despliegue                                        |

## Funcionalidades MVP

### Usuario principal

- Conversacion natural por texto o voz.
- Diario automatico de recuerdos detectados durante la conversacion.
- Gestion de personas importantes: familiares, amigos, vecinos o conocidos.
- Recordatorios: citas medicas, cumpleaños, llamadas o tareas.
- Notificaciones proactivas para recordatorios y memorias relevantes.

### Cuidador

- Acceso dentro de la misma cuenta familiar.
- Puede ver y editar personas, diario y recordatorios.
- Puede ver resumenes de conversaciones, pero no el contenido exacto de cada mensaje.
- En el MVP, los permisos son compartidos dentro de la cuenta: todos los miembros ven los mismos datos estructurados.

## Tech Stack MVP

| Capa             | Tecnologia                         | Para que                                                       |
| ---------------- | ---------------------------------- | -------------------------------------------------------------- |
| App movil        | React Native + Expo                | App iOS/Android durante desarrollo y prototipo                 |
| Backend          | FastAPI (Python)                   | API, logica de negocio y conexion con IA                       |
| Base de datos    | PostgreSQL + pgvector              | Datos relacionales y busqueda semantica                        |
| IA               | Proveedor LLM por definir          | Conversacion, extraccion y razonamiento                        |
| Abstraccion IA   | LangChain                          | Capa comun para modelos, prompts, tools, RAG y embeddings      |
| Tools            | LangChain tools; MCP opcional      | Acceso controlado del agente a datos y acciones                |
| Redis            | Redis                              | Invalidacion de JWT, invitaciones de cuidadores y cache ligera |
| Notificaciones   | Expo Push + APScheduler            | Avisos programados en el MVP                                   |
| Desarrollo local | Docker Compose                     | Levantar API, BD y servicios locales                           |
| Acceso externo   | Cloudflare Tunnel                  | Exponer el backend local para pruebas                          |

## Arquitectura Resumida

```text
App movil
   |
   | HTTPS
   v
FastAPI
   |
   |-- PostgreSQL + pgvector
   |-- Redis
   |-- LangChain -> LLM provider
   |-- APScheduler -> Expo Push
```

El MVP evita una cola distribuida como Celery. Las tareas ligeras de postprocesado pueden ejecutarse con mecanismos simples del backend; si el volumen crece o hacen falta retries robustos, se podra introducir una cola dedicada mas adelante.

## Modelo de Datos Resumido

Entidades principales:

- `accounts`: cuenta familiar compartida.
- `users`: usuarios con rol `senior` o `caregiver`.
- `conversations`: sesiones de conversacion y resumen visible.
- `messages`: mensajes efimeros o con retencion corta.
- `diary_entries`: recuerdos guardados con fecha parcial, ubicacion y embeddings.
- `people`: personas conocidas.
- `reminders`: recordatorios con fecha, ubicacion y recurrencia simple para MVP.
- `invitations`: invitaciones de cuidadores, apoyadas por Redis para expiracion.

El detalle esta en [docs/DATA_MODEL.md](./docs/DATA_MODEL.md).

## API Inicial

Endpoints previstos:

| Area           | Endpoints                                                                                                   |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| Auth           | `POST /auth/register`, `POST /auth/login`, `POST /auth/logout`, `POST /auth/invite-caregiver`               |
| Cuenta         | `GET /account/members`, `DELETE /account/members/{user_id}`                                                 |
| Conversaciones | `POST /conversations`, `POST /conversations/{id}/messages`, `GET /conversations`, `GET /conversations/{id}` |
| Diario         | `GET /diary`, `POST /diary`, `PATCH /diary/{id}`, `DELETE /diary/{id}`                                      |
| Personas       | `GET /people`, `POST /people`, `PATCH /people/{id}`, `DELETE /people/{id}`                                  |
| Recordatorios  | `GET /reminders`, `POST /reminders`, `PATCH /reminders/{id}`, `DELETE /reminders/{id}`                      |
| Notificaciones | `POST /notifications/push-token`, `GET /notifications/history`                                              |

## Experiencia de Uso

La primera pantalla debe priorizar una accion principal clara: hablar o escribir al asistente. El boton puede ser grande y de alto contraste, pero el color no queda cerrado en rojo. Alternativas razonables para probar:

- Boton primario azul o verde con buen contraste.
- Estado visual distinto para escuchar, pensando y respondiendo.
- Entrada de texto siempre disponible como fallback.
- Modo cuidador con navegacion mas densa: diario, personas y recordatorios.

## Deployment MVP

| Fase             | Estado       | Descripcion                                            |
| ---------------- | ------------ | ------------------------------------------------------ |
| Local            | Primera fase | Backend, PostgreSQL, Redis y app con Expo              |
| Tunel Cloudflare | Primera fase | Pruebas desde movil real sin servidor propio           |
| Servidor 24/7    | Segunda fase | Dominio propio, scheduler mas robusto y monitorizacion |
| App Stores       | Futuro       | Builds con EAS y publicacion en stores                 |

## Estructura Esperada del Repositorio

```text
/
├── mobile/
│   ├── app/
│   ├── components/
│   └── services/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── agent/
│   │   ├── models/
│   │   ├── schemas/
│   │   └── scheduler/
│   ├── docker-compose.yml
│   └── Dockerfile
└── docs/
    ├── PRODUCT_DECISIONS.md
    ├── ARCHITECTURE.md
    ├── DATA_MODEL.md
    ├── AI_DESIGN.md
    ├── PROJECT_RISKS.md
    └── GUIDE_*.md
```

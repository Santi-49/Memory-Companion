# 🛠️ GUIDE: Tech Stack — Todas las tecnologías del proyecto

> Mapa de referencia rápida de todas las tecnologías que forman MemoryCompanion, qué hace cada una y cómo se relacionan.

---

## Visión general

```
┌─────────────────────────────────────────────────────────────────┐
│                    DISPOSITIVO DEL USUARIO                      │
│              React Native + Expo Go                             │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTPS (JSON)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      BACKEND (Docker Compose)                   │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐                 │
│  │ FastAPI  │  │ APSched  │  │    Redis     │                 │
│  │ (API)    │  │ (Notif.) │  │ Auth/Cache   │                 │
│  └────┬─────┘  └────┬─────┘  └──────┬───────┘                 │
│       │              │               │                         │
│       └──────────────┴───────────────┴─────────┐              │
│                                                  ▼              │
│                                    ┌─────────────────────────┐ │
│                                    │  PostgreSQL + pgvector  │ │
│                                    └─────────────────────────┘ │
│                                                                 │
│  FastAPI ──► LLM Provider ──► Tools internas / MCP opcional    │
└─────────────────────────────────────────────────────────────────┘
                            │
                    Cloudflare Tunnel
                            │
                        Internet
```

---

## Frontend (App Móvil)

### React Native
- **Qué es**: framework de Meta para crear apps móviles con JavaScript/TypeScript
- **Por qué**: un solo código para iOS y Android; gran ecosistema de librerías
- **Alternativas consideradas**: Flutter (Google) — descartado por menor ecosistema JS

### Expo / Expo Go
- **Qué es**: plataforma sobre React Native que simplifica el desarrollo y compilación
- **Por qué**: desarrollo ultrarrápido (QR → app en el móvil), sin necesidad de Mac para probar en iOS, compilación en la nube (EAS)
- **Expo Go**: la app que permite probar la app durante el desarrollo sin compilar

### Expo Push Notifications
- **Qué es**: servicio de Expo para enviar notificaciones push a iOS y Android
- **Por qué**: integrado nativamente con Expo, abstrae las diferencias entre APNs (Apple) y FCM (Google)

---

## Backend

### FastAPI
- **Qué es**: framework de Python para crear APIs web
- **Por qué**: muy rápido, documentación automática en `/docs`, tipado estricto con Pydantic, soporte nativo async
- **Alternativas consideradas**: Django REST Framework (más pesado), Flask (menos moderno)

### Python
- **Qué es**: lenguaje de programación
- **Por qué**: ecosistema de IA/ML sin rival, buenas librerías para FastAPI, proveedores LLM, PostgreSQL y pgvector

### Tareas en segundo plano
- **Qué es**: trabajo que el backend puede hacer después de responder al usuario
- **MVP**: tareas ligeras dentro de FastAPI o procesos simples
- **Futuro**: Celery, RQ o Dramatiq si hacen falta retries robustos o workers separados
- **Ver**: [GUIDE_ASYNC.md](./GUIDE_ASYNC.md)

### APScheduler
- **Qué es**: librería Python para ejecutar funciones en horarios programados
- **Por qué**: gestiona el envío de recordatorios y notificaciones proactivas a la hora correcta

---

## Base de Datos

### PostgreSQL
- **Qué es**: base de datos relacional de código abierto, una de las más populares del mundo
- **Por qué**: robusta, fiable, soporte excelente de tipos de datos avanzados, gratuita
- **Ver**: [GUIDE_DATABASE.md](./GUIDE_DATABASE.md)

### pgvector
- **Qué es**: extensión de PostgreSQL que añade soporte para vectores (embeddings)
- **Por qué**: permite búsqueda semántica sin necesidad de una base de datos vectorial separada (Pinecone, Weaviate, etc.)
- **Ver**: [GUIDE_RAG.md](./GUIDE_RAG.md)

### Redis
- **Qué es**: base de datos en memoria, extremadamente rápida
- **Por qué**: invalidación de JWT, invitaciones temporales de cuidadores, rate limiting y caché ligera
- **Ver**: [GUIDE_ASYNC.md](./GUIDE_ASYNC.md)

---

## Inteligencia Artificial

### Proveedor LLM
- **Qué es**: API para acceder a un modelo de lenguaje como Claude, OpenAI o Gemini
- **Por qué**: genera la conversación, resume, extrae datos estructurados y decide cuándo usar tools
- **Estado**: proveedor por definir; Claude es una opción candidata
- **Ver**: [GUIDE_AI_AGENT.md](./GUIDE_AI_AGENT.md)

### Tools / MCP
- **Qué es**: capa que permite al agente buscar datos o ejecutar acciones
- **MVP**: tool calling interno desde FastAPI
- **Futuro**: MCP si interesa estandarizar herramientas reutilizables
- **Ver**: [GUIDE_AI_AGENT.md](./GUIDE_AI_AGENT.md)

### Embeddings
- **Qué es**: representaciones vectoriales del significado de textos
- **Por qué**: necesarios para la búsqueda semántica (RAG)
- **Proveedor**: pendiente de decidir; no se debe fijar todavía una dimensión concreta de vector
- **Ver**: [GUIDE_RAG.md](./GUIDE_RAG.md)

---

## Infraestructura y Deployment

### Docker / Docker Compose
- **Qué es**: sistema de contenedores para empaquetar y ejecutar aplicaciones de forma aislada
- **Por qué**: "funciona igual en cualquier sitio", levanta todo el entorno con un comando
- **Ver**: [GUIDE_DOCKER.md](./GUIDE_DOCKER.md)

### Cloudflare Tunnel (cloudflared)
- **Qué es**: herramienta de Cloudflare para exponer servicios locales a internet de forma segura
- **Por qué**: permite acceso externo sin servidor ni configuración de router, HTTPS automático, gratuito
- **Ver**: [GUIDE_DEPLOYMENT.md](./GUIDE_DEPLOYMENT.md)

### GitHub
- **Qué es**: plataforma para alojar código y colaborar en proyectos de software
- **Por qué**: control de versiones, historial de cambios, colaboración
- **Ver**: [GUIDE_GITHUB.md](./GUIDE_GITHUB.md)

---

## Resumen en una tabla

| Tecnología | Capa | Coste | Dificultad |
|---|---|---|---|
| React Native | Frontend | Gratis | Media |
| Expo / Expo Go | Frontend | Gratis (dev) | Baja |
| FastAPI | Backend | Gratis | Media |
| Tareas en segundo plano | Backend | Gratis | Baja-Media |
| APScheduler | Backend | Gratis | Baja |
| PostgreSQL | Base de datos | Gratis | Baja |
| pgvector | Base de datos | Gratis | Baja |
| Redis | Infraestructura | Gratis | Baja |
| Docker Compose | Infraestructura | Gratis | Media |
| Proveedor LLM | IA | **De pago** (por uso) | Baja |
| Cloudflare Tunnel | Deployment | Gratis | Baja |
| Dominio | Deployment | ~$10/año | Baja |

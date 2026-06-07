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
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ FastAPI  │  │  Celery  │  │  APSched │  │    Redis     │  │
│  │ (API)    │  │ (Worker) │  │ (Notif.) │  │   (Cola)     │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────────┘  │
│       │              │              │                           │
│       └──────────────┴──────────────┴───────────┐              │
│                                                  ▼              │
│                                    ┌─────────────────────────┐ │
│                                    │  PostgreSQL + pgvector  │ │
│                                    └─────────────────────────┘ │
│                                                                 │
│  FastAPI ──► Claude API (Anthropic) ──► MCP Tools              │
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
- **Por qué**: ecosistema de IA/ML sin rival, las librerías de Anthropic (Claude), pgvector y Celery son todas Python

### Celery
- **Qué es**: librería Python para tareas asíncronas distribuidas
- **Por qué**: permite delegar trabajo pesado (guardar embeddings, procesar datos) sin bloquear la respuesta al usuario
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
- **Por qué**: actúa como cola de mensajes entre FastAPI y Celery; también puede usarse como caché
- **Ver**: [GUIDE_ASYNC.md](./GUIDE_ASYNC.md)

---

## Inteligencia Artificial

### Claude API (Anthropic)
- **Qué es**: API para acceder al modelo de lenguaje Claude
- **Por qué**: conversación natural de alta calidad, soporte nativo de tools/MCP, buenas capacidades de seguir instrucciones complejas
- **Modelo usado**: Claude Sonnet (balance entre calidad y coste)
- **Ver**: [GUIDE_AI_AGENT.md](./GUIDE_AI_AGENT.md)

### MCP (Model Context Protocol)
- **Qué es**: protocolo de Anthropic para conectar la IA con herramientas externas de forma estandarizada
- **Por qué**: permite al agente buscar en la BD, guardar datos y ejecutar acciones de forma estructurada y segura
- **Ver**: [GUIDE_AI_AGENT.md](./GUIDE_AI_AGENT.md)

### Embeddings
- **Qué es**: representaciones vectoriales del significado de textos
- **Por qué**: necesarios para la búsqueda semántica (RAG)
- **Proveedor**: se pueden usar los embeddings de Anthropic o OpenAI (text-embedding-3-small)
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
| Celery | Backend | Gratis | Media |
| APScheduler | Backend | Gratis | Baja |
| PostgreSQL | Base de datos | Gratis | Baja |
| pgvector | Base de datos | Gratis | Baja |
| Redis | Infraestructura | Gratis | Baja |
| Docker Compose | Infraestructura | Gratis | Media |
| Claude API | IA | **De pago** (por uso) | Baja |
| Cloudflare Tunnel | Deployment | Gratis | Baja |
| Dominio | Deployment | ~$10/año | Baja |

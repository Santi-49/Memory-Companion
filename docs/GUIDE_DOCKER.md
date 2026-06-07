# 🐳 GUIDE: Docker y Docker Compose

> Este documento explica qué es Docker, por qué lo usamos y cómo funciona en MemoryCompanion.

---

## El problema que resuelve Docker

Cuando desarrollas una aplicación, esta depende de muchas cosas: una versión concreta de Python, librerías específicas, PostgreSQL instalado de cierta manera, variables de configuración...

El problema clásico: **"en mi ordenador funciona, pero en el tuyo no"**. Porque tu ordenador tiene configuraciones distintas al de otra persona, o al servidor de producción.

**Docker resuelve esto**: empaqueta la aplicación junto con todo lo que necesita para funcionar en una "caja" aislada. Esa caja funciona igual en cualquier ordenador o servidor.

---

## ¿Qué es una Imagen Docker?

Una **imagen** es la "receta" o "plantilla" de esa caja. Define qué sistema operativo usar, qué instalar, qué archivos incluir y cómo arrancar la aplicación.

Es como una imagen ISO de un sistema operativo, pero mucho más pequeña y específica.

Ejemplos de imágenes que usamos:
- `postgres:16` — imagen oficial de PostgreSQL versión 16
- `redis:7` — imagen oficial de Redis
- La imagen de nuestra propia API (definida en el `Dockerfile` del proyecto)

---

## ¿Qué es un Contenedor?

Un **contenedor** es una instancia en ejecución de una imagen. Si la imagen es la receta, el contenedor es el plato ya preparado.

```
Imagen (receta)  →  docker run  →  Contenedor (en ejecución)
```

Puedes tener múltiples contenedores corriendo a la vez, cada uno aislado de los demás, aunque compartan el mismo ordenador.

---

## ¿Qué es Docker Compose?

En MemoryCompanion necesitamos varios servicios corriendo a la vez:
- El servidor FastAPI
- PostgreSQL (base de datos)
- Redis (cola de mensajes)
- El worker de Celery
- El scheduler de notificaciones

Levantar cada uno por separado sería tedioso. **Docker Compose** permite definir todos los servicios en un único archivo (`docker-compose.yml`) y levantarlos todos con un solo comando.

---

## El archivo `docker-compose.yml`

Este archivo describe todos los servicios del backend:

```yaml
services:
  db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: memorycompanion
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      DATABASE_URL: postgresql://...
      REDIS_URL: redis://redis:6379

  worker:
    build: .
    command: celery worker
    depends_on:
      - redis
      - db

  scheduler:
    build: .
    command: python scheduler.py
    depends_on:
      - db
```

Cada bloque bajo `services:` es un servicio. `depends_on` indica que un servicio espera a que otro esté listo antes de arrancar.

---

## Comandos básicos

Todos se ejecutan desde la carpeta `backend/` donde está el `docker-compose.yml`:

```bash
# Levantar todos los servicios (en segundo plano)
docker-compose up -d

# Ver qué servicios están corriendo
docker-compose ps

# Ver los logs de un servicio concreto
docker-compose logs api
docker-compose logs worker

# Parar todos los servicios
docker-compose down

# Parar y eliminar también los datos (¡cuidado! borra la BD)
docker-compose down -v

# Reconstruir las imágenes (después de cambiar código)
docker-compose up -d --build
```

---

## Puertos: cómo se accede a cada servicio

Cuando un contenedor expone un puerto, ese servicio es accesible en `localhost:<puerto>` desde tu ordenador:

| Servicio | Puerto | Para qué |
|---|---|---|
| `api` | `8000` | La API de FastAPI — `http://localhost:8000` |
| `db` | `5432` | PostgreSQL — para conectar con un cliente de BD |
| `redis` | `6379` | Redis — normalmente no se accede directamente |

El puerto `8000` es al que apunta la app móvil cuando está en la misma red local.

---

## ¿Qué es el Dockerfile?

El `Dockerfile` define cómo construir la imagen de nuestra API. Es la "receta" específica del proyecto:

```dockerfile
# Usar Python 3.12 como base
FROM python:3.12-slim

# Instalar dependencias
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copiar el código
COPY ./app /app

# Arrancar el servidor
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Resumen visual

```
Tu ordenador
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  docker-compose up                                      │
│       │                                                 │
│       ├── Contenedor: api      (puerto 8000)            │
│       ├── Contenedor: db       (puerto 5432)            │
│       ├── Contenedor: redis    (puerto 6379)            │
│       ├── Contenedor: worker   (sin puerto)             │
│       └── Contenedor: scheduler(sin puerto)             │
│                                                         │
│  Todos se comunican entre sí por nombre de servicio     │
│  (api habla con db usando "db:5432", no "localhost")    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

# Architecture

> Arquitectura tecnica propuesta para el MVP.

## Servicios

```text
React Native / Expo
        |
        | HTTPS JSON
        v
FastAPI
        |
        |-- PostgreSQL + pgvector
        |-- Redis
        |-- LLM provider
        |-- APScheduler
        |-- Expo Push
```

## Backend

FastAPI concentra:

- Endpoints HTTP.
- Autenticacion JWT.
- Reglas de negocio.
- Ejecucion de tools del agente.
- Escritura y lectura en PostgreSQL.
- Generacion de resumenes y extraccion de datos.

Para el MVP no se introduce Celery. Las tareas de postprocesado pueden empezar como operaciones internas del backend o tareas ligeras en segundo plano. Si aparecen trabajos largos, retries complejos o volumen real, se puede introducir una cola dedicada.

## Redis

Redis se mantiene en el MVP, pero no como cola de Celery.

Usos previstos:

- Invalidacion de JWT en logout o revocacion.
- Tokens temporales de invitacion para cuidadores.
- Rate limiting sencillo.
- Cache ligera para respuestas o busquedas repetidas.

## PostgreSQL + pgvector

PostgreSQL guarda la informacion principal. `pgvector` permite busqueda semantica sobre recuerdos, personas y resumenes.

El proveedor y dimension de embeddings aun no estan definidos, asi que el codigo no debe asumir una dimension fija como `VECTOR(1536)` hasta escoger modelo.

## Scheduler

APScheduler es suficiente para el MVP local:

- Revisar recordatorios.
- Enviar push notifications.
- Lanzar memorias proactivas simples.

Limitacion: si el backend corre en varios procesos o servidores, APScheduler puede duplicar trabajos. Antes de produccion 24/7 conviene mover esta responsabilidad a un scheduler con bloqueo en base de datos, un worker dedicado o un servicio externo.

## Flujo de Mensaje

```text
Usuario envia mensaje
        |
        v
FastAPI carga contexto
        |
        v
LLM responde y puede llamar tools
        |
        v
FastAPI ejecuta tools contra PostgreSQL
        |
        v
Respuesta vuelve a la app
        |
        v
Backend extrae recuerdos/personas/recordatorios si aplica
```

## Evolucion Posible

| Necesidad | Alternativa futura |
| --- | --- |
| Trabajos lentos y retries | Celery, RQ, Dramatiq o cola gestionada |
| Scheduler en produccion | Celery Beat, cron externo o job runner con locking |
| Mucha busqueda vectorial | Indices pgvector afinados o base vectorial separada |
| Multiples proveedores IA | Capa `LLMProvider` con adaptadores |

# ⚡ GUIDE: Tareas Asíncronas

> Este documento explica qué es una tarea asíncrona, por qué es clave para que el agente sea rápido, y cómo funcionan Celery y Redis en MemoryCompanion.

---

## El problema: velocidad vs trabajo pesado

Cuando el usuario envía un mensaje, espera una respuesta rápida. Pero hay operaciones que tardan tiempo:

- Generar embeddings de un texto nuevo (llamada a la API de embeddings)
- Guardar en la base de datos con varios campos relacionados
- Analizar si se mencionó una persona nueva y crear el registro

Si el agente hiciera todo esto **antes** de responder, el usuario esperaría 5-10 segundos. Inaceptable para una conversación.

**Solución: hacer el trabajo pesado después de responder.**

---

## ¿Qué es "asíncrono"?

**Síncrono** = paso a paso, uno detrás de otro. No puedes hacer B hasta que A termina.

**Asíncrono** = puedes empezar B sin esperar a que A termine. Ambos ocurren "en paralelo".

Analogía del restaurante:
- **Síncrono**: el camarero toma tu pedido, va a la cocina, espera a que esté listo, te lo trae y solo entonces atiende a otra mesa.
- **Asíncrono**: el camarero toma tu pedido, lo pasa a la cocina y mientras esperas, atiende a otras mesas. Cuando la cocina avisa, te trae la comida.

En nuestro caso:
- El agente responde al usuario (rápido)
- Y en paralelo, un "worker" guarda los datos nuevos en la BD (puede tardar más, no importa)

---

## ¿Qué es una Cola de Mensajes?

Una **cola de mensajes** es un sistema donde puedes poner tareas pendientes y otro proceso las va recogiendo y ejecutando.

Funciona como una cola de espera real: las tareas se añaden por un lado y se procesan por el otro, en orden.

```
FastAPI (productor)          Cola (Redis)          Worker (consumidor)
        │                                                  │
        │ → "guarda este recuerdo en BD" ──────────────►  │
        │                                                  │ procesa tarea
        │ → "genera embedding de este texto" ──────────►  │
        │                                                  │ procesa tarea
```

---

## ¿Qué es Redis?

**Redis** es una base de datos en memoria, extremadamente rápida, que se usa como **cola de mensajes** en MemoryCompanion.

Cuando FastAPI quiere delegar una tarea, escribe en Redis. Celery lee de Redis y ejecuta la tarea. Redis actúa de intermediario.

Redis también puede usarse como caché (para no repetir cálculos que ya se hicieron), aunque en este proyecto su rol principal es la cola de Celery.

---

## ¿Qué es Celery?

**Celery** es una librería de Python para gestionar tareas asíncronas. Es el "worker" que procesa las tareas que FastAPI encola en Redis.

En MemoryCompanion, Celery se encarga de:

| Tarea | Cuándo se dispara |
|---|---|
| Generar embedding de una nueva memoria | Al guardar una `diary_entry` |
| Guardar persona nueva detectada por el agente | Al finalizar un mensaje del agente |
| Actualizar datos de una persona existente | Cuando el agente detecta información nueva |
| Crear un recordatorio detectado en conversación | Cuando el usuario menciona algo futuro |
| Generar resumen de conversación | Al finalizar una conversación |

---

## El flujo completo en MemoryCompanion

```
1. Usuario envía mensaje
        │
        ▼
2. FastAPI recibe el mensaje

3. FastAPI llama al Agente IA
        │
        ▼ (puede tardar 1-3 segundos)
4. Agente genera respuesta
   + detecta: "el usuario mencionó a su hermano Pedro, no estaba en la BD"

5. FastAPI devuelve respuesta al usuario ← AQUÍ EL USUARIO YA VE LA RESPUESTA
        │
        │ (en paralelo, sin que el usuario espere)
        ▼
6. FastAPI encola en Redis:
   → "crear persona: Pedro, hermano, teléfono desconocido"

7. Celery Worker (proceso separado) coge la tarea:
   → Crea la persona en PostgreSQL
   → Genera el embedding del nombre+notas
   → Guarda el embedding en la BD

8. Listo. El usuario nunca notó el paso 6-7.
```

---

## ¿Cuándo es el momento "correcto" para guardar?

El agente analiza el historial al final de cada respuesta y detecta:

- **Persona nueva mencionada** → crear persona (async)
- **Datos nuevos de persona existente** → actualizar persona (async)
- **Recuerdo o anécdota contada** → crear entrada en el diario (async)
- **Fecha futura o recordatorio** → crear reminder (async)

Si el usuario habla durante 20 minutos y menciona muchas cosas, todas se guardan progresivamente sin que la conversación se ralentice nunca.

---

## Resumen visual de los procesos

```
Tu servidor corre 3 tipos de procesos simultáneamente:

┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   FastAPI       │   │  Celery Worker  │   │  APScheduler    │
│                 │   │                 │   │                 │
│ - Recibe HTTP   │   │ - Lee tareas    │   │ - Cada hora:    │
│ - Llama al      │   │   de Redis      │   │   revisa        │
│   agente IA     │   │ - Guarda en BD  │   │   reminders     │
│ - Responde      │   │ - Genera        │   │ - Envía push    │
│   rápido        │   │   embeddings    │   │   notifications │
│ - Encola tasks  │   │                 │   │                 │
└─────────────────┘   └─────────────────┘   └─────────────────┘
         │                     ▲
         │   Redis (cola)      │
         └────────────────────►┘
```

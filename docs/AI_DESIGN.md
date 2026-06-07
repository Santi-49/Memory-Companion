# AI Design

> Diseño del sistema de IA para MemoryCompanion.

## Objetivo del Agente

El agente debe conversar de forma amable, clara y paciente. Sus responsabilidades principales son:

- Responder preguntas sobre recuerdos, personas y recordatorios.
- Buscar informacion estructurada antes de responder cuando haga falta.
- Detectar recuerdos, personas o recordatorios nuevos.
- Proponer memorias proactivas sin resultar invasivo.

## Proveedor LLM

El proveedor aun no esta cerrado. Claude, OpenAI o Gemini son opciones razonables.

Para evitar acoplamiento temprano, el backend usara LangChain como capa de abstraccion sobre modelos, prompts, tools y RAG. Encima de LangChain conviene mantener una capa interna propia:

```text
AgentService
  |-- generate_response(...)
  |-- summarize_conversation(...)
  |-- extract_structured_data(...)

EmbeddingService
  |-- embed_text(...)
  |-- embed_documents(...)
```

Claude puede ser el primer candidato para conversacion, pero el README no debe asumirlo como decision irreversible.

## LangChain

LangChain actua como capa comun para:

- Cambiar entre proveedores LLM sin reescribir todo el agente.
- Definir prompts reutilizables.
- Declarar tools con entradas/salidas estructuradas.
- Construir flujos RAG con retrievers.
- Integrar embeddings de distintos proveedores.

No deberia sustituir la logica de dominio. Las reglas importantes del producto, permisos, filtros por cuenta y validaciones deben vivir en servicios propios del backend.

## Tools

El MVP puede empezar con LangChain tools ejecutadas por servicios internos de FastAPI. MCP queda como opcion si mas adelante interesa exponer herramientas de forma estandarizada.

### Lectura

| Tool | Uso |
| --- | --- |
| `search_people` | Buscar personas por nombre, apodo, relacion o categoria |
| `search_diary` | Buscar recuerdos por texto, fecha, lugar, personas o embeddings |
| `search_reminders` | Buscar recordatorios por fecha, estado, lugar o persona |
| `get_random_memory` | Recuperar una memoria para uso proactivo |

### Escritura

| Tool | Uso |
| --- | --- |
| `create_diary_entry` | Guardar recuerdo nuevo |
| `update_diary_entry` | Corregir o ampliar recuerdo |
| `create_person` | Crear persona conocida |
| `update_person` | Actualizar datos de persona |
| `create_reminder` | Crear recordatorio |
| `update_reminder` | Cambiar fecha, texto, estado o ubicacion |
| `delete_reminder` | Eliminar recordatorio |

## RAG

La busqueda debe combinar:

- Filtros estructurados: fecha, categoria, persona, estado, ubicacion.
- Busqueda textual simple.
- Busqueda semantica con embeddings.

No todo debe resolverse con vectores. Para preguntas como "que tengo mañana" o "quien es Carlos", los filtros estructurados son mas fiables.

## Embeddings

El proveedor aun no esta definido. Por eso:

- No fijar `VECTOR(1536)` en documentacion de producto.
- Guardar `embedding_model` junto al vector.
- Centralizar la generacion de embeddings.
- Encapsular embeddings tras `EmbeddingService`, aunque internamente use LangChain.
- Preparar migracion si se cambia de modelo.

Opciones futuras:

| Proveedor | Nota |
| --- | --- |
| OpenAI embeddings | Muy habituales, buena relacion calidad/coste |
| Voyage | Buen rendimiento semantico |
| Google | Integracion si se usa Gemini |
| Local embeddings | Menos coste variable, mas complejidad operativa |

## Extraccion de Datos

Despues de una respuesta, el sistema puede extraer informacion nueva:

- Personas mencionadas.
- Recuerdos con fecha parcial o ubicacion.
- Recordatorios con fecha, recurrencia y lugar.

En el MVP, esto puede hacerse dentro del backend sin Celery. Si la extraccion tarda demasiado, puede degradarse a una tarea diferida.

## System Prompt

El prompt del agente debe incluir:

- Tono calmado, simple y respetuoso.
- No inventar datos personales.
- Buscar antes de responder cuando la pregunta dependa de memoria guardada.
- Confirmar antes de guardar informacion sensible o ambigua.
- Crear recordatorios solo cuando haya fecha/hora suficiente o pedir aclaracion.

## Evaluacion

Casos que conviene probar desde el principio:

- "Quien es Carlos?"
- "Que hice el verano pasado?"
- "Recuerdame llamar a Ana mañana por la tarde."
- "Eso no fue en Roma, fue en Florencia."
- "No guardes eso."

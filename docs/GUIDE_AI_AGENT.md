# 🤖 GUIDE: IA — Agentes, Tools, Inferencia y Voz

> Este documento explica qué es un modelo de lenguaje, qué es un agente, cómo funciona la inferencia y cómo podría integrarse la voz en MemoryCompanion.

---

## ¿Qué es un LLM?

Un **LLM** (Large Language Model, Modelo de Lenguaje Grande) es una IA entrenada con enormes cantidades de texto para entender y generar lenguaje humano de forma natural.

Ejemplos conocidos: ChatGPT (OpenAI), Gemini (Google), Claude (Anthropic). En MemoryCompanion usamos **Claude**, de Anthropic.

Un LLM no "sabe" cosas como una enciclopedia. Lo que hace es predecir, con mucha precisión, qué texto tiene sentido como respuesta dado un contexto. El resultado es una conversación que parece humana.

---

## ¿Qué es un Chatbot vs un Agente?

### Chatbot simple

Solo genera texto. Recibe un mensaje → devuelve una respuesta. No puede hacer nada más.

```
Usuario: "¿Qué tiempo hace en Barcelona?"
Chatbot: "Lo siento, no tengo acceso a información en tiempo real."
```

### Agente

Un agente puede **usar herramientas** (tools) para obtener información o ejecutar acciones antes de responder. No está limitado a lo que "sabe" de su entrenamiento.

```
Usuario: "¿Cuándo fue mi última visita al médico?"
Agente:
  1. Llama a la tool search_diary("visita médico")
  2. Recibe resultado de la base de datos: "15 marzo 2023, revisión rodilla"
  3. Responde: "Tu última visita al médico fue el 15 de marzo de 2023..."
```

El agente decide **cuándo y qué tools usar** de forma autónoma, según la pregunta del usuario.

---

## ¿Qué es el System Prompt?

Antes de que empiece la conversación, se le dan al agente instrucciones permanentes que definen su comportamiento. Esto es el **system prompt**.

En MemoryCompanion el system prompt incluye cosas como:
- "Eres un compañero amable para personas mayores. Habla de forma simple y pausada."
- "Después de cada respuesta, analiza si el usuario mencionó una persona nueva o un recuerdo y guárdalo."
- "Si el usuario pregunta por alguien, primero busca en la base de datos de personas."

El usuario no ve el system prompt, pero condiciona todo el comportamiento del agente.

---

## ¿Qué es la Inferencia?

La **inferencia** es el proceso de generar una respuesta. Cuando el agente recibe un mensaje, el modelo de IA procesa el texto y calcula (infiere) cuál es la respuesta más adecuada.

Este proceso consume **recursos de computación** (CPU/GPU) y tarda un tiempo. Por eso se habla de **latencia de inferencia**: el tiempo que tarda el modelo en responder.

Para un agente conversacional, la latencia ideal es **menos de 3 segundos**. Si el agente tiene que hacer múltiples búsquedas en la BD antes de responder, ese tiempo aumenta. Por eso usamos tareas asíncronas para las operaciones de escritura (ver [GUIDE_ASYNC.md](./GUIDE_ASYNC.md)).

### ¿Por qué la inferencia cuesta dinero?

Los LLMs se ejecutan en servidores con hardware muy potente (GPUs). Anthropic (la empresa detrás de Claude) cobra por el uso de su API según:
- **Tokens de entrada**: cuánto texto le mandas al modelo (historial de conversación + contexto)
- **Tokens de salida**: cuánto texto genera el modelo en la respuesta

Un **token** es aproximadamente una palabra (o parte de una). Una conversación larga con mucho contexto del diario puede consumir miles de tokens por mensaje.

---

## ¿Qué es TTS? (Text-to-Speech)

**TTS** (Text-to-Speech, Texto a Voz) es la tecnología que convierte texto escrito en voz hablada.

Para personas mayores que prefieren escuchar en lugar de leer, esto es especialmente valioso. La app podría **leer en voz alta** las respuestas del agente.

### Cómo se integraría en MemoryCompanion

El flujo con TTS:

```
Usuario habla (voz) → App → STT convierte voz a texto
                                    │
                                    ▼
                            Agente IA (texto → texto)
                                    │
                                    ▼
                        TTS convierte respuesta a voz
                                    │
                                    ▼
                         App reproduce el audio
```

**STT** (Speech-to-Text) es el proceso inverso: convierte la voz del usuario en texto para que el agente lo procese.

### Opciones técnicas para TTS

| Servicio | Calidad | Coste | Notas |
|---|---|---|---|
| **Expo Speech** (nativo) | Media | Gratis | Funciona sin API externa, voz robótica |
| **OpenAI TTS** | Alta | ~$0.015/1000 chars | Voces muy naturales |
| **ElevenLabs** | Muy alta | Desde gratis (limitado) | Ideal para una voz personalizada |
| **Google Cloud TTS** | Alta | Desde gratis (limitado) | Buen soporte de español |

Para el prototipo, **Expo Speech** es suficiente y gratis. Para una versión más pulida, OpenAI TTS o ElevenLabs dan una experiencia mucho más humana.

---

## Resumen del flujo del agente en MemoryCompanion

```
System Prompt (instrucciones permanentes)
        +
Contexto del usuario (personas frecuentes, próximos recordatorios)
        +
Historial de la conversación actual
        +
Mensaje del usuario
        │
        ▼
    AGENTE IA (Claude)
        │
        ├── ¿Necesito buscar algo?
        │       └── Llama a search_diary / search_people / search_reminders
        │               └── FastAPI consulta PostgreSQL y devuelve resultado
        │
        ├── ¿El usuario mencionó algo nuevo?
        │       └── (async) create_diary_entry / create_person / create_reminder
        │
        └── Genera respuesta de texto
                │
                ▼ (opcional)
            TTS convierte a audio
                │
                ▼
          App reproduce respuesta
```

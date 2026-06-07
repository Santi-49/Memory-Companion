# 🔍 GUIDE: RAG y Búsqueda Semántica

> Este documento explica qué es RAG, qué son los embeddings y por qué permiten encontrar recuerdos aunque el usuario no use las palabras exactas.

---

## El problema de la búsqueda tradicional

En una búsqueda normal (como en Google o en un buscador de base de datos), el sistema busca la coincidencia exacta de palabras.

Si en el diario de María está guardado:
> "Fui al hospital con mi hija Ana para una revisión de rodilla"

Y María pregunta: **"¿Cuándo estuve en el médico con mi hija?"**

Una búsqueda tradicional fallaría: el texto dice "hospital" y "revisión", no "médico". Las palabras no coinciden exactamente, aunque el significado es el mismo.

**RAG** y los **embeddings** resuelven esto.

---

## ¿Qué es un Embedding?

Un **embedding** es una representación matemática del significado de un texto. La IA convierte un texto en una lista de números (un "vector") que captura su significado semántico.

```
"Fui al hospital con mi hija"  →  [0.23, -0.45, 0.89, 0.12, ...]
"Estuve en el médico con Ana"  →  [0.25, -0.43, 0.87, 0.11, ...]
"El gato se subió al tejado"   →  [-0.67, 0.34, -0.21, 0.88, ...]
```

Los dos primeros textos tienen significados parecidos → sus vectores son **parecidos** (números similares).
El tercero habla de algo completamente diferente → su vector es **distinto**.

---

## ¿Cómo funciona la búsqueda semántica?

1. Cuando se guarda una memoria en el diario, el sistema genera su embedding y lo guarda en la base de datos junto al texto
2. Cuando el usuario hace una pregunta, el sistema genera también el embedding de esa pregunta
3. El sistema compara ese embedding con todos los embeddings guardados, buscando los más parecidos
4. Devuelve los textos cuyos embeddings son más cercanos → los más relevantes semánticamente

```
Pregunta del usuario:
"¿Cuándo estuve en el médico con mi hija?"
        │
        ▼
Embedding de la pregunta: [0.25, -0.43, 0.87, ...]
        │
        ▼ Comparar con todos los embeddings del diario
        │
        ├── "Fui al hospital con mi hija"   → similitud: 0.97 ✅ MUY PARECIDO
        ├── "Viajé a París con mi marido"   → similitud: 0.23 ❌ poco parecido
        └── "El gato se subió al tejado"    → similitud: 0.05 ❌ nada parecido
        │
        ▼
Resultado: "Fui al hospital con mi hija Ana para una revisión de rodilla"
```

---

## ¿Qué es RAG?

**RAG** significa **Retrieval-Augmented Generation** (Generación Aumentada por Recuperación). Es la técnica que combina:

1. **Retrieval** (Recuperación): buscar en la base de datos la información relevante para la pregunta del usuario
2. **Augmented** (Aumentada): incluir esa información en el contexto que se le da a la IA
3. **Generation** (Generación): la IA genera una respuesta usando tanto la pregunta como la información recuperada

### Sin RAG:

```
Usuario: "¿Cuándo estuve en el médico con mi hija?"
IA: "No tengo información sobre eso."  ❌
```

### Con RAG:

```
Sistema recupera del diario:
  → "Fui al hospital con mi hija Ana para una revisión de rodilla (marzo 2023)"

La IA recibe:
  [Información del diario]: "Fui al hospital con mi hija Ana..."
  [Pregunta del usuario]: "¿Cuándo estuve en el médico con mi hija?"

IA responde:
  "Fuiste al hospital con tu hija Ana en marzo de 2023 para una revisión
   de rodilla. ¿Quieres que te cuente más sobre ese día?" ✅
```

---

## ¿Qué es pgvector?

**pgvector** es una extensión de PostgreSQL que permite guardar y comparar vectores (embeddings) directamente en la base de datos.

Sin pgvector necesitaríamos una base de datos separada solo para los embeddings (como Pinecone o Weaviate). Con pgvector, todo vive en el mismo PostgreSQL: los textos y sus representaciones semánticas.

Esto simplifica enormemente la arquitectura del proyecto.

---

## En MemoryCompanion, RAG se usa para:

| Búsqueda | Cómo funciona |
|---|---|
| **Buscar memorias** | El agente convierte la pregunta en embedding y busca en `diary_entries` las entradas más similares |
| **Buscar personas** | Si el usuario dice "mi amiga la rubia del trabajo", busca en `people` por similitud semántica |
| **Buscar recordatorios** | Encuentra recordatorios relacionados aunque el usuario no recuerde el nombre exacto |

---

## Resumen en una frase

RAG = la IA primero busca información relevante en tu base de datos personal, y después genera una respuesta usando esa información. Los embeddings permiten que esa búsqueda funcione por significado, no solo por palabras exactas.

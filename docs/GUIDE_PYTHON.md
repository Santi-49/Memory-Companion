# 🐍 GUIDE: Python

> Este documento explica qué es Python, cómo se organiza un proyecto Python, y los conceptos clave que aparecen en el backend de MemoryCompanion.

---

## ¿Qué es Python?

**Python** es un lenguaje de programación. Un lenguaje de programación es simplemente una forma de darle instrucciones a un ordenador, escritas en texto que los humanos podemos leer y entender.

Python es especialmente popular porque:
- Su sintaxis es muy legible (se parece al inglés)
- Tiene una comunidad enorme y millones de librerías gratuitas
- Es el lenguaje dominante en Inteligencia Artificial y ciencia de datos
- Es el lenguaje de FastAPI, Celery, y todas las librerías de IA que usamos

---

## Un programa Python básico

```python
# Esto es un comentario — Python lo ignora, es para humanos

nombre = "María"           # Variable de tipo texto
edad = 78                  # Variable de tipo número entero
activa = True              # Variable de tipo booleano (True/False)

print("Hola, " + nombre)   # Muestra texto en pantalla
# → Hola, María
```

Las líneas que empiezan con `#` son **comentarios** — notas para el programador que Python ignora completamente.

---

## Variables y Tipos de Dato

Una **variable** es un nombre que guarda un valor. En Python no hay que declarar el tipo — Python lo detecta solo.

```python
texto = "Fui a París con mi marido"   # str (texto)
numero = 1978                          # int (entero)
decimal = 3.14                         # float (decimal)
verdadero = True                       # bool (verdadero/falso)
lista = ["Ana", "Pedro", "Luis"]       # list (lista)
nada = None                            # None (sin valor, equivale a null en BD)
```

---

## Funciones

Una **función** es un bloque de código con nombre que hace una tarea concreta. Se define una vez y se puede llamar todas las veces que se quiera.

```python
def saludar(nombre):
    mensaje = "Hola, " + nombre
    return mensaje

resultado = saludar("María")
print(resultado)
# → Hola, María
```

- `def` indica que se está definiendo una función
- El nombre va seguido de paréntesis con los **parámetros** (datos de entrada)
- `return` devuelve el resultado

---

## Clases y Objetos

Una **clase** es una plantilla para crear objetos. Un objeto agrupa datos y funciones relacionadas.

```python
class Persona:
    def __init__(self, nombre, categoria):  # Constructor: se ejecuta al crear un objeto
        self.nombre = nombre
        self.categoria = categoria

    def presentarse(self):
        return f"Soy {self.nombre}, categoría: {self.categoria}"

# Crear un objeto de la clase Persona
ana = Persona("Ana García", "direct_family")
print(ana.presentarse())
# → Soy Ana García, categoría: direct_family
```

En FastAPI, las **clases se usan constantemente** para definir la estructura de los datos que entran y salen de la API.

---

## Listas y Diccionarios

Son las dos estructuras de datos más usadas en Python.

### Lista
Una colección ordenada de elementos:

```python
categorias = ["personal", "family", "travel", "health"]
print(categorias[0])   # → personal (los índices empiezan en 0)
print(len(categorias)) # → 4 (cuántos elementos tiene)

# Recorrer una lista
for cat in categorias:
    print(cat)
```

### Diccionario
Colección de pares clave-valor (como JSON):

```python
persona = {
    "nombre": "Ana García",
    "categoria": "direct_family",
    "telefono": "612345678"
}

print(persona["nombre"])       # → Ana García
persona["email"] = "ana@gmail.com"  # Añadir un campo nuevo
```

---

## Imports y Librerías

Python tiene miles de librerías gratuitas. Para usarlas se "importan" al principio del archivo:

```python
import os                          # Librería estándar (viene con Python)
from datetime import datetime      # Importar solo una parte
from fastapi import FastAPI        # Librería externa instalada con pip
```

### ¿Qué es pip?

**pip** es el gestor de paquetes de Python — como una tienda de librerías. Se usa desde la terminal:

```bash
pip install fastapi        # Instalar FastAPI
pip install celery         # Instalar Celery
pip install anthropic      # Instalar la librería de Claude
```

En el proyecto, todas las dependencias están listadas en `requirements.txt`:

```
fastapi==0.111.0
celery==5.3.6
anthropic==0.25.0
pgvector==0.3.0
sqlalchemy==2.0.30
```

Con un solo comando se instalan todas:
```bash
pip install -r requirements.txt
```

---

## Funciones Asíncronas (async/await)

Python moderno permite escribir código asíncrono — funciones que pueden "pausarse" mientras esperan algo (una respuesta de la BD, una llamada a la API) sin bloquear el resto del programa.

```python
import asyncio

async def obtener_respuesta():          # "async def" = función asíncrona
    respuesta = await llamar_a_claude() # "await" = espera sin bloquear
    return respuesta
```

FastAPI usa `async def` para todos los endpoints, lo que permite manejar muchas peticiones simultáneas de forma eficiente.

---

## Cómo se organiza el proyecto Python

El backend de MemoryCompanion está organizado así:

```
backend/
├── requirements.txt          # Lista de dependencias
├── Dockerfile                # Instrucciones para Docker
├── docker-compose.yml        # Orquestación de servicios
│
└── app/
    ├── main.py               # Punto de entrada — arranca FastAPI
    ├── config.py             # Variables de configuración (URL de BD, claves API...)
    │
    ├── api/                  # Endpoints de la API
    │   ├── auth.py           # POST /auth/login, POST /auth/register
    │   ├── conversations.py  # POST /conversations/, etc.
    │   ├── diary.py          # GET/POST/PATCH/DELETE /diary/
    │   ├── people.py         # GET/POST/PATCH/DELETE /people/
    │   ├── reminders.py      # GET/POST/PATCH/DELETE /reminders/
    │   └── caregiver.py      # Endpoints del cuidador
    │
    ├── agent/                # Lógica del agente IA
    │   ├── agent.py          # El agente principal (Claude + tools)
    │   └── tools.py          # Definición de las tools MCP
    │
    ├── models/               # Modelos de base de datos
    │   ├── account.py        # Tabla accounts
    │   ├── conversation.py   # Tablas conversations + messages
    │   ├── diary.py          # Tabla diary_entries
    │   ├── people.py         # Tablas people + people_diary
    │   └── reminder.py       # Tablas reminders + people_reminders
    │
    ├── schemas/              # Validación de datos de entrada/salida
    │   ├── conversation.py
    │   ├── diary.py
    │   └── ...
    │
    ├── workers/              # Tareas asíncronas (Celery)
    │   └── tasks.py          # generate_embedding, save_person, etc.
    │
    └── scheduler/            # Notificaciones programadas
        └── jobs.py           # check_reminders, send_proactive_memory, etc.
```

---

## Variables de Entorno (.env)

Las claves secretas (contraseñas de BD, claves de API) **nunca** se escriben directamente en el código. Se guardan en un archivo `.env` que no se sube a GitHub:

```bash
# .env (nunca subir a GitHub)
DATABASE_URL=postgresql://user:password@localhost:5432/memorycompanion
REDIS_URL=redis://localhost:6379
ANTHROPIC_API_KEY=sk-ant-...
SECRET_KEY=mi_clave_secreta_para_jwt
```

En Python se leen así:

```python
import os
from dotenv import load_dotenv

load_dotenv()  # Carga el archivo .env

database_url = os.getenv("DATABASE_URL")
api_key = os.getenv("ANTHROPIC_API_KEY")
```

---

## Un endpoint real en FastAPI

Para que sea concreto, así es como se vería el endpoint de enviar un mensaje al agente:

```python
from fastapi import FastAPI, Depends
from anthropic import Anthropic

app = FastAPI()
client = Anthropic()

@app.post("/conversations/{conversation_id}/messages")
async def send_message(conversation_id: str, body: MessageInput, user=Depends(get_current_user)):
    # 1. Recuperar historial de la BD
    history = await get_conversation_history(conversation_id)

    # 2. Llamar al agente IA
    response = client.messages.create(
        model="claude-sonnet-4-5",
        system=SYSTEM_PROMPT,
        messages=history + [{"role": "user", "content": body.content}],
        tools=MCP_TOOLS
    )

    # 3. Guardar mensaje en BD
    await save_message(conversation_id, body.content, response.content)

    # 4. Delegar extracción de datos a Celery (asíncrono)
    extract_and_save_data.delay(conversation_id, response.content, user.id)

    # 5. Devolver respuesta
    return {"content": response.content, "role": "assistant"}
```

- `@app.post(...)` — el decorador indica que esta función responde a peticiones POST en esa ruta
- `async def` — función asíncrona para no bloquear mientras espera la respuesta de Claude
- `Depends(get_current_user)` — verifica automáticamente el token JWT del usuario
- `.delay(...)` — encola la tarea en Celery sin esperar a que termine

---

## Errores comunes al empezar

| Error | Qué significa | Solución |
|---|---|---|
| `ModuleNotFoundError` | No está instalada una librería | `pip install nombre-libreria` |
| `IndentationError` | El código no está bien indentado | Python usa espacios para estructurar el código — revisar la alineación |
| `KeyError` | Se intenta acceder a una clave que no existe en un diccionario | Verificar que la clave existe antes de acceder |
| `None` cuando no debería | Una función devolvió `None` inesperadamente | Revisar que la función tiene `return` con un valor |
| `AttributeError` | Se llama a algo que no existe en un objeto | Revisar el nombre del atributo o método |

---

## Recursos para aprender más

- **Documentación oficial de Python**: [docs.python.org/es](https://docs.python.org/es/3/) — disponible en español
- **FastAPI**: [fastapi.tiangolo.com](https://fastapi.tiangolo.com) — documentación muy bien escrita con ejemplos
- **Tutorial interactivo gratuito**: [learnpython.org/es](https://www.learnpython.org/es/)

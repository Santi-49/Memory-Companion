# 📡 GUIDE: ¿Qué es una API?

> Este documento explica los conceptos de API, HTTP y autenticación que se usan en MemoryCompanion. No hace falta saber programar para entenderlo.

---

## ¿Qué es una API?

Imagina que estás en un restaurante. Tú (el cliente) no entras a la cocina a preparar tu comida — le dices al **camarero** lo que quieres, y él se lo transmite a la cocina y te trae el resultado.

En una aplicación:
- **Tú** = la app móvil (lo que ve el usuario)
- **El camarero** = la API
- **La cocina** = el servidor (donde está la lógica y la base de datos)

La **API** (Application Programming Interface) es el sistema de comunicación entre la app y el servidor. Define exactamente qué se puede pedir y cómo se pide.

---

## ¿Qué es HTTP?

HTTP es el "idioma" que usa internet para que dos dispositivos se comuniquen. Cuando visitas una página web, tu navegador le envía una petición HTTP al servidor, y el servidor responde con el contenido.

En nuestra app pasa lo mismo: la app envía peticiones HTTP al servidor cada vez que el usuario hace algo.

### Los métodos HTTP

Cada petición tiene un "método" que indica qué tipo de acción se quiere hacer:

| Método | Analogía | Para qué se usa |
|---|---|---|
| `GET` | "Dame información" | Consultar datos (ver el diario, listar personas...) |
| `POST` | "Crea algo nuevo" | Enviar un mensaje, registrarse, crear un recordatorio |
| `PATCH` | "Modifica algo existente" | Actualizar el nombre de una persona, editar una memoria |
| `DELETE` | "Elimina esto" | Borrar un recordatorio |

---

## ¿Qué es un Endpoint?

Un endpoint es una **dirección específica** del servidor a la que se puede hacer una petición. Es como una extensión de teléfono: el número principal es el servidor, y el endpoint es la extensión para el departamento concreto.

Ejemplos de endpoints en MemoryCompanion:

```
POST  https://mi-servidor.com/auth/login          ← para iniciar sesión
GET   https://mi-servidor.com/diary/              ← para ver el diario
POST  https://mi-servidor.com/conversations/      ← para iniciar conversación
```

Cada endpoint tiene:
- Un **método** (GET, POST, etc.)
- Una **ruta** (`/diary/`, `/people/`, etc.)
- Opcionalmente, datos que se envían junto a la petición

---

## ¿Qué es JSON?

JSON es el formato en el que la app y el servidor se pasan información. Es texto estructurado, fácil de leer para humanos y para máquinas.

**Ejemplo:** cuando el usuario envía un mensaje, la app manda esto al servidor:

```json
{
  "content": "¿Recuerdas cuando fui a París con mi marido?"
}
```

Y el servidor responde con esto:

```json
{
  "message_id": "abc-123",
  "role": "assistant",
  "content": "¡Claro! Fuiste a París en el verano de 1978, fue vuestro primer viaje juntos al extranjero.",
  "actions_taken": ["searched_diary"]
}
```

Todo entre `{` y `}` es un objeto JSON. Cada línea tiene una **clave** (izquierda) y un **valor** (derecha), separados por `:`.

---

## ¿Qué es la Autenticación? ¿Qué es un Token JWT?

Para que el servidor sepa **quién** está haciendo cada petición, se usa autenticación.

### El flujo de login

1. El usuario introduce email y contraseña en la app
2. La app hace `POST /auth/login` con esos datos
3. El servidor verifica que son correctos
4. El servidor devuelve un **token JWT** — una cadena de texto larga y codificada
5. La app guarda ese token en el dispositivo

A partir de ese momento, **en cada petición**, la app incluye ese token en la cabecera:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

El servidor lee el token y sabe quién es el usuario, sin que tenga que volver a introducir la contraseña.

### ¿Por qué JWT y no solo un ID de usuario?

El token JWT contiene información firmada criptográficamente. El servidor puede verificar que el token es auténtico (que lo generó él mismo) sin necesidad de consultarlo en la base de datos en cada petición. Esto hace el sistema más rápido y seguro.

El token caduca (normalmente en horas o días), después del cual el usuario tiene que volver a iniciar sesión.

---

## Resumen visual del flujo completo

```
DISPOSITIVO (App móvil)                    SERVIDOR (FastAPI)
       │                                          │
       │  POST /auth/login                        │
       │  {"email": "...", "password": "..."}     │
       │ ────────────────────────────────────────►│
       │                                          │ Verifica credenciales
       │  {"token": "eyJ..."}                     │
       │ ◄────────────────────────────────────────│
       │                                          │
       │  [Guarda token en el dispositivo]         │
       │                                          │
       │  POST /conversations/{id}/messages        │
       │  Authorization: Bearer eyJ...             │
       │  {"content": "¿Recuerdas París?"}        │
       │ ────────────────────────────────────────►│
       │                                          │ Verifica token
       │                                          │ Llama al agente IA
       │                                          │ Busca en BD
       │  {"content": "Claro, fue en 1978..."}    │
       │ ◄────────────────────────────────────────│
```

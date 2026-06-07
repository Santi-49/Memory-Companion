# 🗄️ GUIDE: Bases de Datos

> Este documento explica qué es una base de datos, cómo se organiza la información y qué significan los tipos de columna que aparecen en el modelo de datos de MemoryCompanion.

---

## ¿Qué es una base de datos?

Una base de datos es un sistema para **guardar información de forma organizada** y poder consultarla, modificarla o eliminarla de manera eficiente.

Piensa en ella como un conjunto de hojas de cálculo muy potentes, donde cada hoja es una **tabla**, cada fila es un **registro** (un dato concreto) y cada columna es un **campo** (un tipo de información).

---

## Tablas, Filas y Columnas

### Ejemplo con la tabla `people` de MemoryCompanion:

| id | name | nickname | category | phone |
|---|---|---|---|---|
| `a1b2-...` | Ana García López | Anita | `direct_family` | 612345678 |
| `c3d4-...` | Paco Martínez | El vecino | `acquaintance` | null |
| `e5f6-...` | Dra. Méndez | La doctora | `other` | 934567890 |

- Cada **fila** es una persona diferente
- Cada **columna** es un tipo de dato (nombre, apodo, categoría...)
- La tabla tiene 3 filas (3 personas) y 5 columnas

---

## Tipos de Columna

Cada columna tiene un **tipo** que define qué clase de información puede guardar. Es como el formato de una celda en Excel.

### TEXT
Texto libre, sin límite de longitud.
```
"Fuimos a París en nuestro viaje de luna de miel, fue precioso"
"Mi vecino del 3º que tiene un perro labrador llamado Tobi"
```

### INTEGER
Número entero (sin decimales).
```
1978    ← un año
7       ← un mes
25      ← un día
```

### BOOLEAN
Solo puede ser `true` (verdadero) o `false` (falso).
```
true    ← el recordatorio ya fue enviado
false   ← el recordatorio todavía no se ha enviado
```

### TIMESTAMP
Una fecha y hora exacta. Incluye año, mes, día, hora, minuto y segundo.
```
2024-03-15 09:32:00   ← 15 de marzo de 2024 a las 9:32
```

### TIME
Solo la hora del día, sin fecha.
```
14:30:00   ← las 2 y media de la tarde
```

### UUID
Un identificador único universal. Es una cadena de letras y números generada automáticamente, garantizada como única en todo el sistema.
```
550e8400-e29b-41d4-a716-446655440000
```
Se usa como `id` de cada registro para identificarlo de forma inequívoca.

### ENUM
Un valor que solo puede ser uno de una lista predefinida. Como un desplegable en un formulario.

Ejemplo, la columna `category` en `people` solo puede ser uno de:
```
direct_family | family | friend | acquaintance | other
```

Si intentas guardar `"colega"`, la base de datos lo rechaza automáticamente.

### VECTOR(1536)
Un tipo especial de PostgreSQL (extensión pgvector). Guarda una lista de 1536 números decimales que representan el "significado" de un texto (embedding).

```
[0.023, -0.451, 0.892, 0.103, ... ]  ← 1536 números
```

No está pensado para que lo lea un humano — es para que el sistema pueda hacer búsquedas semánticas. Ver [GUIDE_RAG.md](./GUIDE_RAG.md) para más detalle.

---

## Claves Primarias (PK) y Foráneas (FK)

### Clave Primaria (PK — Primary Key)

Cada tabla tiene una columna especial llamada **clave primaria**. Su función es identificar de forma única cada fila. No pueden existir dos filas con el mismo valor de PK, y nunca puede estar vacía.

En MemoryCompanion, todas las tablas usan `id` (tipo UUID) como clave primaria.

### Clave Foránea (FK — Foreign Key)

Una clave foránea es una columna que **apunta a la clave primaria de otra tabla**. Sirve para crear relaciones entre tablas.

Ejemplo: la tabla `conversations` tiene la columna `account_id`, que es una FK que apunta a `accounts.id`. Esto significa: "esta conversación pertenece a esta cuenta".

```
accounts                    conversations
┌──────────────────┐        ┌────────────────────────────┐
│ id (PK)          │◄───────│ account_id (FK → accounts) │
│ name             │        │ started_at                 │
│ email            │        │ summary                    │
└──────────────────┘        └────────────────────────────┘
```

---

## Relaciones entre Tablas

### Uno a Muchos (1:N)

Una cuenta puede tener muchas conversaciones, pero cada conversación pertenece a una sola cuenta. Esto se llama relación "uno a muchos".

```
1 cuenta → N conversaciones
1 conversación → N mensajes
1 cuenta → N entradas del diario
```

### Muchos a Muchos (N:M)

Una entrada del diario puede mencionar a varias personas, y una persona puede aparecer en varias entradas del diario. Esto es una relación "muchos a muchos".

Para implementarla se usa una **tabla intermedia**:

```
diary_entries        people_diary         people
┌─────────────┐     ┌──────────────────┐  ┌────────────┐
│ id (PK)     │◄────│ diary_entry_id   │  │ id (PK)    │
│ content     │     │ person_id        │──►│ name       │
│ ...         │     └──────────────────┘  │ ...        │
└─────────────┘                           └────────────┘
```

---

## ¿Por qué PostgreSQL?

PostgreSQL es una base de datos **relacional** (organizada en tablas relacionadas entre sí), gratuita y de código abierto. Es una de las más usadas en el mundo para aplicaciones profesionales.

Se elige para este proyecto porque:
- Es robusta y fiable para datos importantes
- Soporta la extensión **pgvector** para búsqueda semántica
- Docker Compose la levanta fácilmente sin instalación manual
- Escala bien cuando el proyecto crezca

---

## NULL: cuando un campo no tiene valor

`null` significa "sin valor" o "desconocido". Es diferente de cero o de texto vacío.

Ejemplo en el diario: si el usuario dice "fue un verano de los 80", el sistema guarda:
```
date_year  = 1983   ← año aproximado
date_month = null   ← no se sabe el mes
date_day   = null   ← no se sabe el día
```

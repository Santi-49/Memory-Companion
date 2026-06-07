# 🐙 GUIDE: Git y GitHub

> Este documento explica qué es Git, qué es GitHub y cómo se trabaja con ellos en MemoryCompanion.

---

## ¿Qué es Git?

**Git** es un sistema de **control de versiones**. Su función es registrar todos los cambios que se hacen en el código a lo largo del tiempo, de forma que puedas:

- Ver el historial completo de cambios ("quién cambió qué y cuándo")
- Volver a una versión anterior si algo se rompe
- Trabajar en nuevas funcionalidades sin afectar el código que ya funciona
- Colaborar con otras personas sin que los cambios se pisen entre sí

Piensa en Git como el "historial de versiones" de Word, pero mucho más potente y pensado para código.

---

## ¿Qué es GitHub?

**GitHub** es una plataforma web donde se almacenan proyectos de Git en la nube. Es como Google Drive, pero para código.

- El código vive en un **repositorio** (repo) en GitHub
- Cualquier persona con acceso puede clonar el repo, hacer cambios y proponer que se incorporen
- GitHub también ofrece herramientas para gestionar tareas, revisar código y automatizar procesos

---

## Conceptos básicos

### Repositorio (repo)
La "carpeta raíz" del proyecto en Git. Contiene todo el código y el historial de cambios.

### Commit
Un **commit** es una "foto" del estado del código en un momento dado. Cada commit tiene:
- Un mensaje que describe qué cambió ("Añadir endpoint de recordatorios")
- El autor y la fecha
- Los archivos que cambiaron

```bash
git add .                           # Marcar los archivos modificados
git commit -m "Añadir búsqueda semántica en el diario"  # Guardar la foto
```

### Rama (Branch)
Una **rama** es una línea de desarrollo paralela. Permite trabajar en una nueva funcionalidad sin tocar el código principal.

```
main (código estable)
  │
  ├── develop (integración)
  │       │
  │       ├── feature/agente-ia
  │       └── feature/notificaciones
```

### Push y Pull
- **Push**: subir tus commits locales a GitHub
- **Pull**: descargar los commits de GitHub a tu ordenador

```bash
git push origin develop   # Subir cambios a GitHub
git pull origin develop   # Bajar cambios de GitHub
```

### Pull Request (PR)
Una **Pull Request** es una propuesta de incorporar los cambios de una rama a otra. Permite revisar el código antes de fusionarlo.

Flujo típico:
1. Trabajas en `feature/notificaciones`
2. Abres una PR hacia `develop`
3. Otra persona revisa el código
4. Se aprueba y hace "merge" → los cambios se incorporan a `develop`

---

## Flujo de trabajo en MemoryCompanion

### Día a día

```bash
# 1. Situarte en la rama de desarrollo
git checkout develop

# 2. Bajar los últimos cambios
git pull origin develop

# 3. Crear una rama para la nueva funcionalidad
git checkout -b feature/recordatorios-recurrentes

# 4. Trabajar y hacer commits
git add .
git commit -m "Añadir campo recurrence a la tabla reminders"

# 5. Subir la rama a GitHub
git push origin feature/recordatorios-recurrentes

# 6. Abrir Pull Request en GitHub (desde el navegador)
# 7. Revisar y hacer merge a develop
```

### Cuando algo se rompe

```bash
# Ver el historial de commits
git log --oneline

# Volver a un commit anterior (solo para ver, sin perder nada)
git checkout abc1234

# Volver al estado anterior de un archivo concreto
git checkout HEAD -- app/api/reminders.py
```

---

## Clonar el repositorio

Si es la primera vez que trabajas con el proyecto:

```bash
# Clonar el repo
git clone https://github.com/tu-usuario/memory-companion.git

# Entrar en la carpeta
cd memory-companion

# Instalar dependencias del backend
cd backend
pip install -r requirements.txt

# Instalar dependencias de la app
cd ../mobile
npm install
```

---

## Estructura de ramas del proyecto

| Rama | Propósito | Quién hace push |
|---|---|---|
| `main` | Código estable, "producción" | Solo via PR desde develop |
| `develop` | Integración de features | Via PR desde feature branches |
| `feature/xxx` | Una funcionalidad concreta | El desarrollador que la trabaja |
| `fix/xxx` | Corrección de un bug | El desarrollador que lo corrige |

---

## El archivo `.gitignore`

Git no debe guardar ciertos archivos: contraseñas, claves de API, archivos temporales...

El archivo `.gitignore` lista lo que Git debe ignorar:

```
# Variables de entorno (contienen contraseñas y claves)
.env
.env.local

# Dependencias instaladas (se reinstalan con npm install / pip install)
node_modules/
__pycache__/

# Archivos del sistema operativo
.DS_Store
Thumbs.db
```

**Nunca subas a GitHub archivos `.env` con contraseñas o claves de API.**

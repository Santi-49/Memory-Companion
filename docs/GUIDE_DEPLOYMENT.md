# 🚀 GUIDE: Deployment — Cómo llega la app a los usuarios

> Este documento explica qué significa "desplegar" una aplicación, cómo funciona el proceso desde el desarrollo local hasta llegar a iOS y Android, y cuánto puede costar.

---

## ¿Qué es el Deployment?

**Deployment** (despliegue) es el proceso de hacer que tu aplicación esté disponible para que otras personas la usen — no solo tú en tu ordenador.

Hay varias fases, desde lo más básico (solo tú, en local) hasta lo más completo (cualquier persona en el mundo, desde su móvil).

---

## Fase 1 — Desarrollo Local

**Estado: ✅ Primera fase del proyecto**

Todo corre en tu ordenador. Solo funciona si el ordenador está encendido y en la misma red WiFi que el móvil de prueba.

### Backend

```bash
# Desde la carpeta backend/
docker-compose up -d
```

Levanta todos los servicios: API en `http://localhost:8000`, PostgreSQL, Redis, Celery, Scheduler.

### App móvil

```bash
# Desde la carpeta mobile/
npx expo start
```

Expo muestra un QR. El móvil con **Expo Go** instalado escanea el QR y carga la app directamente. No hace falta instalar nada en el teléfono más allá de Expo Go (disponible en App Store y Google Play).

La app apunta a `http://[IP-LOCAL]:8000` para hablar con el backend.

> **Limitación**: si el móvil no está en la misma WiFi que el ordenador, no funciona. Y si apagas el ordenador, la app deja de funcionar.

---

## Fase 2 — Acceso Externo con Cloudflare Tunnel

**Estado: ✅ Primera fase (complementario al local)**

Permite que la app funcione desde cualquier red, sin servidor externo y sin configurar el router.

### ¿Cómo funciona un Tunnel?

Normalmente, tu ordenador no es accesible desde internet (está protegido por el router/firewall). Un **tunnel** crea un canal seguro desde tu ordenador hasta los servidores de Cloudflare, que sí son accesibles públicamente.

```
Internet                  Cloudflare               Tu ordenador
    │                         │                         │
App móvil ──► mi-app.com ──► Tunnel ──────────────► localhost:8000
```

### Configuración con Cloudflare

1. Crear cuenta gratuita en [cloudflare.com](https://cloudflare.com)
2. Registrar un dominio propio (ver sección de costes)
3. Instalar `cloudflared` en el ordenador:

```bash
# En Mac
brew install cloudflare/cloudflare/cloudflared

# En Windows (PowerShell, como admin)
winget install Cloudflare.cloudflared
```

4. Autenticarse y crear el tunnel:

```bash
cloudflared tunnel login
cloudflared tunnel create memory-companion
```

5. Configurar el tunnel para apuntar a `localhost:8000` y asociarlo al dominio

Una vez configurado, el backend es accesible en `https://api.tudominio.com` desde cualquier lugar del mundo.

> **Importante**: el ordenador sigue teniendo que estar encendido. El tunnel solo hace de "puerta de entrada", el backend sigue corriendo en tu máquina.

---

## Fase 3 — Servidor 24/7

**Estado: 🔜 Segunda fase (necesario para uso real)**

Para que la app funcione siempre — aunque apagues tu ordenador, aunque sea de noche — necesitas un servidor que esté **encendido las 24 horas del día, los 7 días de la semana**.

### ¿Por qué 24/7?

- Los recordatorios se envían a la hora programada (si el servidor está apagado, no se envían)
- Los usuarios pueden usar la app a cualquier hora
- Las conversaciones no se interrumpen

### Opciones de servidor

#### Opción A — Servidor compartido (shared resources) — el más económico

Un servidor compartido significa que tu aplicación comparte los recursos físicos (CPU, RAM) con otras aplicaciones de otros clientes. Es más barato, pero tiene limitaciones de rendimiento.

**Recomendado para el prototipo y primeros usuarios.**

| Proveedor | Plan | Coste/mes | RAM | Notas |
|---|---|---|---|---|
| **Railway** | Starter | ~$5-10 | 512MB-1GB | Fácil de usar, incluye BD |
| **Render** | Individual | ~$7-14 | 512MB | Se "duerme" si no hay tráfico* |
| **Fly.io** | Hobby | ~$3-10 | 256MB-1GB | Más técnico, muy flexible |
| **DigitalOcean** | Droplet básico | $6 | 1GB | VPS simple, control total |

*"Dormirse" significa que si nadie usa la app durante 15 minutos, el servidor se suspende. La siguiente petición tarda 30-60 segundos en "despertar". Para producción real, se necesita un plan de pago que no se duerma.

#### Opción B — Servidor dedicado (VPS) — más control

Un VPS (Virtual Private Server) te da un servidor virtual solo para ti, con recursos garantizados. Más caro, más control.

| Proveedor | Plan | Coste/mes | RAM | Notas |
|---|---|---|---|---|
| **DigitalOcean** | 2GB Droplet | $12 | 2GB | Recomendado para producción |
| **Hetzner** | CX21 | ~$5 | 4GB | Muy buena relación precio/recursos |
| **Linode (Akamai)** | Nanode | $5 | 1GB | Opción económica |

### Con el tunnel de Cloudflare (sin servidor propio)

En la Fase 2, el tunnel funciona con el ordenador encendido. Para simular un servidor 24/7 barato, se puede usar un ordenador pequeño y económico como servidor en casa:

| Dispositivo | Coste inicial | Consumo eléctrico/mes | Total 1er año |
|---|---|---|---|
| **Raspberry Pi 4 (4GB)** | ~$60-70 | ~$1-2 | ~$80-90 |
| **Mini PC usado** | ~$80-120 | ~$2-4 | ~$100-170 |
| **Ordenador viejo** | $0 (ya existe) | ~$5-15 | ~$60-180 |

Con esta opción: el ordenador corre en casa, Cloudflare Tunnel expone el backend, y es efectivamente 24/7 si no se apaga.

---

## Dominio Propio

Para una app real se necesita un **dominio** (como `mi-app.com`). Cloudflare es el proveedor recomendado porque:
- Gestiona el dominio
- El tunnel se integra nativamente
- Ofrece HTTPS automático (conexión segura)
- Protección básica gratuita contra ataques

### Costes de dominio

| Tipo de dominio | Coste/año aprox. |
|---|---|
| `.com` | ~$10-12 |
| `.es` | ~$8-10 |
| `.app` | ~$14-16 |
| `.io` | ~$30-40 |

---

## Fase 4 — App en iOS y Android (App Stores)

**Estado: 🔜 Futuro**

Expo Go es perfecto para desarrollo, pero para distribuir la app a usuarios reales (personas mayores que no saben instalar Expo Go), necesitas publicarla en las tiendas oficiales.

### ¿Cómo funciona?

Expo tiene un servicio llamado **EAS (Expo Application Services)** que compila la app y genera:
- Un archivo `.apk` o `.aab` para **Google Play** (Android)
- Un archivo `.ipa` para **App Store** (iOS)

```bash
# Instalar EAS CLI
npm install -g eas-cli

# Compilar para Android
eas build --platform android

# Compilar para iOS
eas build --platform ios
```

### Requisitos y costes para publicar

| Tienda | Requisito | Coste |
|---|---|---|
| **Google Play** | Cuenta de desarrollador | $25 (pago único) |
| **App Store (Apple)** | Apple Developer Program | $99/año |

> La App Store de Apple es más estricta en su proceso de revisión (puede tardar 1-3 días en aprobar la app).

### Actualizaciones sin pasar por la tienda

Expo permite enviar **actualizaciones over-the-air (OTA)**: cambios en el código de la app llegan al usuario automáticamente sin que tenga que actualizar desde la tienda. Esto es muy útil para correcciones rápidas.

---

## Resumen de costes estimados

### Escenario A — Prototipo mínimo (arranque)

| Concepto | Coste |
|---|---|
| Servidor local (ordenador propio) | $0 |
| Cloudflare Tunnel | $0 |
| Dominio `.com` | ~$10/año |
| Claude API (uso moderado, ~100 conversaciones/mes) | ~$5-15/mes |
| Expo Go (desarrollo) | $0 |
| **Total mensual** | **~$5-15/mes** |
| **Total primer año** | **~$70-190** |

### Escenario B — Primeros usuarios reales (10-50 usuarios)

| Concepto | Coste |
|---|---|
| Servidor Railway/Render (plan básico) | ~$10/mes |
| Base de datos PostgreSQL (incluida o Railway add-on) | ~$5/mes |
| Dominio `.com` | ~$10/año |
| Claude API (~500 conversaciones/mes) | ~$20-50/mes |
| EAS Build (compilar apps) | $0 (plan gratis: 30 builds/mes) |
| Google Play Developer | $25 (único) |
| **Total mensual** | **~$35-65/mes** |
| **Total primer año** | **~$450-810** |

### Escenario C — Crecimiento (100+ usuarios activos)

| Concepto | Coste |
|---|---|
| VPS DigitalOcean 2GB | $12/mes |
| Base de datos managed | $15/mes |
| Dominio | $10/año |
| Claude API (~2000 conv/mes) | $80-150/mes |
| Apple Developer | $99/año |
| EAS Production plan | $29/mes |
| **Total mensual** | **~$136-206/mes** |

> Los costes de Claude API dependen mucho de la longitud de las conversaciones y del contexto que se incluya en cada llamada. Con usuarios activos y diarios largos, puede ser el mayor coste del sistema.

---

## Simulando un entorno real en local

Antes de pagar por servidores, puedes simular el entorno completo en tu ordenador:

1. **Backend**: Docker Compose (ya configurado)
2. **Acceso externo**: Cloudflare Tunnel → simula tener un servidor en internet
3. **App móvil**: Expo Go apuntando al tunnel URL → simula la app en producción
4. **Notificaciones**: APScheduler corriendo en Docker → simula el scheduler de producción
5. **HTTPS**: Cloudflare lo proporciona automáticamente → igual que en producción

Con esta configuración, el entorno local es funcionalmente idéntico a producción. La única diferencia es que el servidor es tu ordenador y no está disponible si lo apagas.

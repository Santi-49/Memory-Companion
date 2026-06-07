# Data Model

> Modelo de datos propuesto para el MVP.

## Principios

- Una cuenta familiar agrupa un senior y uno o mas cuidadores.
- Los cuidadores ven datos estructurados compartidos, no mensajes exactos.
- Las fechas de recuerdos pueden ser parciales o inciertas.
- Recuerdos y recordatorios pueden tener ubicacion.
- Los embeddings dependen de un proveedor aun no elegido.

## `accounts`

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Cuenta familiar |
| `display_name` | TEXT | Nombre visible |
| `created_at` | TIMESTAMP | Fecha de creacion |

## `users`

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Usuario |
| `account_id` | UUID FK | Cuenta familiar |
| `username` | TEXT | Login unico |
| `password_hash` | TEXT | Password hasheada |
| `role` | ENUM | `senior` o `caregiver` |
| `name` | TEXT | Nombre visible |
| `phone` | TEXT NULL | Telefono |
| `birth_date` | DATE NULL | Fecha de nacimiento |
| `timezone` | TEXT | Zona horaria |
| `push_token` | TEXT NULL | Token push |
| `is_active` | BOOLEAN | Usuario activo |
| `last_login_at` | TIMESTAMP NULL | Ultimo login |
| `created_at` | TIMESTAMP | Creacion |

Regla MVP: una cuenta tiene exactamente un `senior` y uno o mas `caregiver`.

## `invitations`

Las invitaciones de cuidadores pueden vivir parcialmente en Redis por su expiracion natural, pero conviene persistir un registro minimo si se quiere trazabilidad.

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Invitacion |
| `account_id` | UUID FK | Cuenta |
| `email_or_phone` | TEXT | Destinatario |
| `role` | ENUM | Normalmente `caregiver` |
| `status` | ENUM | `pending`, `accepted`, `expired`, `revoked` |
| `created_by_user_id` | UUID FK | Quien invita |
| `created_at` | TIMESTAMP | Creacion |
| `accepted_at` | TIMESTAMP NULL | Aceptacion |

## `conversations`

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Conversacion |
| `account_id` | UUID FK | Cuenta |
| `user_id` | UUID FK | Usuario que inicio |
| `started_at` | TIMESTAMP | Inicio |
| `ended_at` | TIMESTAMP NULL | Fin |
| `summary` | TEXT NULL | Resumen visible para cuidador |
| `summary_embedding` | VECTOR NULL | Dimension segun proveedor |
| `embedding_model` | TEXT NULL | Modelo usado |

## `messages`

Mensajes exactos. Deben tratarse como datos efimeros o con retencion corta.

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Mensaje |
| `conversation_id` | UUID FK | Conversacion |
| `role` | ENUM | `user` o `assistant` |
| `content` | TEXT | Texto exacto |
| `created_at` | TIMESTAMP | Creacion |
| `expires_at` | TIMESTAMP NULL | Retencion corta opcional |

## `diary_entries`

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Recuerdo |
| `account_id` | UUID FK | Cuenta |
| `created_by_user_id` | UUID FK | Creador |
| `content` | TEXT | Descripcion completa |
| `summary` | TEXT NULL | Resumen corto |
| `category` | ENUM | `personal`, `family`, `travel`, `health`, `work`, `social`, `other` |
| `source` | ENUM | `conversation` o `manual` |
| `happened_at` | TIMESTAMP NULL | Fecha exacta si existe |
| `date_precision` | ENUM | `unknown`, `year`, `season`, `month`, `day`, `datetime`, `range` |
| `date_text` | TEXT NULL | Ej: "verano de los 80" |
| `date_range_start` | DATE NULL | Inicio aproximado |
| `date_range_end` | DATE NULL | Fin aproximado |
| `date_confidence` | NUMERIC NULL | Confianza 0-1 |
| `location_name` | TEXT NULL | Ej: "Roma", "Hospital La Paz" |
| `location_address` | TEXT NULL | Direccion si se conoce |
| `location_lat` | NUMERIC NULL | Latitud opcional |
| `location_lng` | NUMERIC NULL | Longitud opcional |
| `content_embedding` | VECTOR NULL | Dimension segun proveedor |
| `summary_embedding` | VECTOR NULL | Dimension segun proveedor |
| `embedding_model` | TEXT NULL | Modelo usado |
| `created_at` | TIMESTAMP | Creacion |

## `people`

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Persona |
| `account_id` | UUID FK | Cuenta |
| `created_by_user_id` | UUID FK | Creador |
| `name` | TEXT | Nombre |
| `nickname` | TEXT NULL | Apodo |
| `phone` | TEXT NULL | Telefono |
| `email` | TEXT NULL | Email |
| `relationship` | TEXT | Relacion en lenguaje natural |
| `category` | ENUM | `direct_family`, `family`, `friend`, `acquaintance`, `other` |
| `notes` | TEXT NULL | Notas |
| `name_embedding` | VECTOR NULL | Dimension segun proveedor |
| `embedding_model` | TEXT NULL | Modelo usado |
| `created_at` | TIMESTAMP | Creacion |

## `reminders`

Para MVP basta una recurrencia simple. No se necesita un modelo perfecto de calendarios desde el dia uno.

| Campo | Tipo | Nota |
| --- | --- | --- |
| `id` | UUID PK | Recordatorio |
| `account_id` | UUID FK | Cuenta |
| `created_by_user_id` | UUID FK | Creador |
| `title` | TEXT | Titulo |
| `description` | TEXT NULL | Detalle |
| `remind_at` | TIMESTAMP | Proxima fecha de aviso |
| `recurrence` | ENUM | `none`, `daily`, `weekly`, `monthly`, `yearly` |
| `last_sent_at` | TIMESTAMP NULL | Ultimo envio |
| `status` | ENUM | `active`, `paused`, `done`, `cancelled` |
| `location_name` | TEXT NULL | Lugar |
| `location_address` | TEXT NULL | Direccion |
| `location_lat` | NUMERIC NULL | Latitud |
| `location_lng` | NUMERIC NULL | Longitud |
| `source` | ENUM | `conversation` o `manual` |
| `created_at` | TIMESTAMP | Creacion |

## Relaciones

| Tabla | Nota |
| --- | --- |
| `people_diary` | Relaciona personas con recuerdos |
| `people_reminders` | Relaciona personas con recordatorios |

```text
accounts
  |
  |-- users
  |-- conversations -- messages
  |-- diary_entries -- people_diary -- people
  |-- reminders ----- people_reminders -- people
```

# Product Decisions

> Decisiones de producto para el MVP de MemoryCompanion.

## Objetivo

MemoryCompanion ayuda a una persona mayor a recordar informacion importante mediante una experiencia conversacional simple. El producto combina tres superficies:

- Asistente conversacional para el usuario principal.
- Diario estructurado de recuerdos, personas y recordatorios.
- Panel de apoyo para cuidadores.

## Roles

### Senior

Es el usuario principal de la aplicacion. Puede conversar con la IA, crear y editar recuerdos, gestionar personas y crear recordatorios.

### Caregiver

Es un familiar o persona de apoyo dentro de la misma cuenta familiar. En el MVP puede ver y editar los mismos datos estructurados que el senior:

- Personas.
- Diario.
- Recordatorios.
- Resumenes de conversaciones.

No ve el contenido exacto de cada mensaje de conversacion.

## Permisos MVP

Para simplificar el primer prototipo, todos los miembros de una cuenta familiar comparten acceso a los datos estructurados. No hay permisos por campo ni por entrada.

| Recurso | Senior | Caregiver |
| --- | --- | --- |
| Personas | Ver y editar | Ver y editar |
| Diario | Ver y editar | Ver y editar |
| Recordatorios | Ver y editar | Ver y editar |
| Resumenes de conversacion | Ver | Ver |
| Mensajes exactos | Ver durante la sesion | No visible en panel |

## Conversaciones y Privacidad Basica

El MVP separa:

- `messages`: mensajes exactos, efimeros o con retencion corta.
- `conversations.summary`: resumen visible para cuidador y util para contexto futuro.

Esto permite que el sistema recuerde lo importante sin convertir el panel del cuidador en un historial literal de conversaciones.

## Experiencia Principal

La pantalla principal debe tener una accion clara para hablar o escribir. La decision de color queda abierta:

- Un boton rojo puede llamar la atencion, pero tambien comunica alarma.
- Azul o verde suelen transmitir accion segura y tranquila.
- Lo importante es alto contraste, tamaño tactil generoso y estados claros: escuchando, pensando, respondiendo.

Para personas mayores, la interfaz debe priorizar legibilidad, estabilidad visual y recuperacion facil cuando algo sale mal.

## Alcance No MVP

- Permisos finos por tipo de dato.
- Auditoria completa de cada edicion.
- Consentimiento granular por cuidador.
- Multiples seniors en la misma cuenta.
- Publicacion en App Store / Google Play.

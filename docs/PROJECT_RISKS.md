# Project Risks

> Riesgos principales de MemoryCompanion y mitigaciones recomendadas para el MVP.

## Resumen

MemoryCompanion mezcla IA conversacional, datos personales, recordatorios y una audiencia potencialmente vulnerable. Los riesgos mas importantes no estan solo en la tecnologia: estan en la confianza, la fiabilidad y la experiencia de uso.

| Riesgo | Impacto | Probabilidad | Mitigacion MVP |
| --- | --- | --- | --- |
| La IA inventa recuerdos o datos personales | Alto | Media | RAG con fuentes claras, respuestas con incertidumbre y no inventar cuando falten datos |
| Recordatorios no enviados o duplicados | Alto | Media | Tests de scheduler, `last_sent_at`, logs de envio y evitar multiples schedulers en produccion |
| Extraccion automatica guarda informacion incorrecta | Alto | Alta | Marcar origen/confianza, permitir edicion facil y pedir confirmacion ante datos ambiguos |
| Las personas mayores no quieren usarlo | Alto | Media | Validar motivacion real, onboarding acompañado y valor visible desde el primer uso |
| Las personas mayores no saben usarlo | Alto | Alta | UI simple, texto grande, estados claros, fallback de texto y pruebas tempranas con usuarios reales |
| La app se percibe como infantilizante o invasiva | Alto | Media | Tono adulto, control del usuario y explicacion clara de que se guarda y comparte |
| Coste del LLM crece con historiales largos | Medio | Media | Resumenes, limites de contexto, cache y medicion de tokens desde el inicio |
| Dependencia de proveedor IA | Medio | Media | LangChain, adaptadores propios, embeddings desacoplados y `embedding_model` persistido |
| Abstraccion excesiva con LangChain | Medio | Media | Encapsular LangChain dentro del modulo de agente y mantener reglas de dominio fuera |
| Datos sensibles mal expuestos entre roles | Alto | Baja-Media | Permisos MVP claros: datos estructurados compartidos, mensajes exactos no visibles al cuidador |
| Cloudflare/local no sirve para uso real 24/7 | Medio | Alta | Documentar como fase de pruebas y planear servidor estable antes de usuarios reales |
| Scheduler con APScheduler duplica trabajos en varios procesos | Medio | Media | Usarlo solo en MVP local; migrar a scheduler con locking antes de produccion |
| Modelo de fechas/ubicaciones insuficiente | Medio | Media | Guardar texto original, precision, rango y ubicacion opcional |

## Riesgos de IA

### Alucinaciones

El agente puede responder con seguridad sobre algo que no esta en la base de datos. En esta app eso es especialmente delicado porque un recuerdo inventado puede confundir al usuario.

Mitigaciones:

- Buscar en datos guardados antes de responder preguntas personales.
- Decir "no lo tengo guardado" cuando no haya evidencia.
- Guardar resumenes y fuentes internas para explicar de donde sale una respuesta.
- Evaluar casos comunes: personas, fechas, recuerdos corregidos y recordatorios.

### Extraccion Incorrecta

El agente puede interpretar una frase como recuerdo o recordatorio cuando no lo era.

Mitigaciones:

- Guardar `source`, `confidence` y texto original cuando sea util.
- Confirmar antes de crear recordatorios ambiguos.
- Hacer que editar o borrar sea muy facil desde la app.

## Riesgos de Producto

### Adopcion Por Personas Mayores

La app puede fallar aunque la tecnologia funcione si la persona mayor no ve un motivo claro para usarla. El riesgo no es solo usabilidad; tambien es deseo, habito, confianza y dignidad.

Mitigaciones:

- Validar con usuarios reales si el problema les importa a ellos, no solo al cuidador.
- Diseñar el primer uso alrededor de una accion valiosa en menos de un minuto: crear un recordatorio, preguntar por alguien o guardar un recuerdo.
- Evitar un tono infantil o condescendiente; debe sentirse como ayuda adulta, no vigilancia.
- Permitir que el cuidador configure datos iniciales para que la app no empiece "vacía".
- Medir senales simples: vuelve al dia siguiente, crea recordatorios, pregunta por personas, corrige recuerdos.

### Usabilidad y Comprension

Algunos usuarios pueden tener poca familiaridad con apps, problemas de vision, audicion, motricidad fina o memoria de trabajo. Si no entienden que esta pasando, abandonaran rapido.

Mitigaciones:

- Primera pantalla con una accion principal evidente.
- Texto grande, alto contraste y botones tactiles amplios.
- Estados visibles y persistentes: escuchando, procesando, respondiendo, error.
- Entrada de texto como alternativa a la voz.
- Confirmaciones simples: "He guardado este recordatorio para mañana a las 10:00".
- Recuperacion facil: deshacer, corregir, borrar y repetir respuesta.
- Pruebas tempranas con personas mayores, no solo con familiares o equipo tecnico.

### Voz y Entorno Real

La voz puede ser la interfaz mas natural, pero tambien puede fallar por ruido, acentos, habla pausada, problemas de pronunciacion o miedo a "hablarle al movil".

Mitigaciones:

- Mantener texto como fallback siempre disponible.
- Mostrar transcripcion antes de guardar datos importantes.
- Permitir repetir, cancelar y corregir sin navegar por menus.
- Probar en entornos reales: salon, cocina, calle, consulta medica.

### Dependencia Del Cuidador

Si el cuidador tiene que configurarlo todo o resolver cada error, el producto puede convertirse en una carga mas para la familia.

Mitigaciones:

- Separar claramente modo senior y modo cuidador.
- Hacer que el cuidador pueda preparar datos iniciales, pero que el uso diario sea independiente.
- Notificar al cuidador solo cambios relevantes, no cada interaccion.
- Diseñar flujos de ayuda remota: corregir personas, añadir recordatorios y revisar resumenes.

### Confianza Familiar

El cuidador necesita ayudar sin invadir. Para el MVP, el equilibrio es simple: puede ver datos estructurados y resumenes, pero no el texto exacto de los mensajes.

Mitigaciones:

- Explicar dentro del producto que se comparte y que no.
- Evitar mostrar conversaciones literales en el panel de cuidador.
- Permitir borrar recuerdos o corregir datos.

## Riesgos Tecnicos

### Notificaciones

Los recordatorios son una promesa fuerte. Si fallan, el usuario pierde confianza rapidamente.

Mitigaciones:

- Registrar intentos de envio.
- Usar `last_sent_at` y `status` para evitar duplicados.
- Testear zonas horarias y cambios de hora.
- No usar APScheduler sin bloqueo en entornos con multiples procesos.

### Coste y Latencia

El coste puede crecer si cada mensaje manda mucho historial al proveedor LLM. La latencia tambien puede empeorar si se hacen demasiadas busquedas o extracciones en linea.

Mitigaciones:

- Resumir conversaciones.
- Limitar contexto enviado al modelo.
- Medir tokens y tiempos desde el inicio.
- Dejar tareas pesadas como evolucion futura hacia una cola dedicada.

### Cambios de Proveedor

El proveedor de chat y embeddings aun no esta decidido. Fijar pronto dimensiones o APIs concretas puede encarecer una migracion.

Mitigaciones:

- Usar LangChain como abstraccion, pero detras de servicios propios (`AgentService`, `EmbeddingService`).
- Guardar `embedding_model`.
- No asumir `VECTOR(1536)` hasta elegir proveedor.

### Abstraccion Excesiva

LangChain ayuda a construir agentes y RAG, pero tambien puede ocultar demasiado flujo si se usa sin limites.

Mitigaciones:

- Encapsular LangChain dentro de `backend/app/agent/`.
- No usar objetos de LangChain directamente en routers de FastAPI.
- Mantener permisos, filtros por `account_id` y validaciones en servicios propios.
- Escribir tests alrededor de los servicios propios, no solo de chains.

## Riesgos de Scope

El proyecto puede crecer rapido: voz, cuidadores, diario, recordatorios, RAG, notificaciones, panel familiar y deployment. Para el MVP conviene proteger el alcance.

Prioridad recomendada:

1. Conversacion basica con memoria consultable.
2. Diario/personas/recordatorios editables manualmente.
3. Extraccion automatica con confirmacion o facil correccion.
4. Notificaciones simples.
5. Voz y proactividad mas pulidas.

## Senales de Alerta

- El usuario no sabe si la app esta escuchando o pensando.
- El agente responde con datos personales sin haberlos buscado.
- Crear o corregir un recuerdo requiere demasiados pasos.
- Los recordatorios no tienen historial de envio.
- El coste por conversacion no se mide.
- El scheduler corre en mas de un proceso sin bloqueo.

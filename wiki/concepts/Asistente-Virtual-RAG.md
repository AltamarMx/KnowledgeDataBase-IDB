---
title: Asistente Virtual del Curso (RAG)
type: concepto
tags: [concepto, ia, rag, asistente, telegram, opencode, claude, ier]
aliases: [asistente virtual, bot del curso, rag, chat ia, asistente ia]
clases: [001, 011, 012]
updated: 2026-05-08
---

# Asistente Virtual del Curso (RAG)

## Qué es

Asistente virtual basado en **IA + RAG (Retrieval-Augmented Generation)** desarrollado para el taller IDB y la materia subsiguiente (Energía en Edificaciones). Responde preguntas sobre los temas del curso usando un **corpus curado** (transcripciones de clase, scripts Python, libretas Jupyter, artículos seleccionados) en lugar de el conocimiento general del modelo.

> "Su intención no es sustituir a quien da clases, sino complementar."

## Por qué un corpus curado y no un LLM general

LLMs comerciales (ChatGPT, Claude) **se equivocan mucho con Energy Plus**. Razón: hay **poca data pública** sobre E+ → los modelos tienen poco material de entrenamiento → alucinan objetos, ecuaciones, sintaxis IDF.

> "Cada vez menos, pero todavía se equivocan mucho con Energy Plus. En transferencia de calor sí saben bastante bien."

Solución del grupo IER: **construir un corpus curado** específico de los temas del curso, y obligar al asistente a contestar **solo desde ese corpus** (no desde el conocimiento general del LLM).

## Stack técnico

| Componente | Función |
|------------|---------|
| **Raspberry Pi 8GB** | Hardware base, en casa del profesor (mudará al IER) |
| **OpenCode** | Framework agéntico de Anthropic — escribe/lee archivos en su entorno aislado |
| **Claude (Opus)** | LLM remoto que produce las respuestas (la Raspberry no aguanta correr LLM local) |
| **Telegram bot** | Interfaz de chat con los usuarios |
| **Corpus curado** | Transcripciones de clases + scripts + artículos, todo en un repo |

> "OpenCode está conectado a una Raspberry Pi que vive en mi casa. Es peligroso ponerlo en tu computadora porque puede borrar cosas o leer cosas y compartirlas. Por eso debe estar aislado."

### Plan futuro

- **Máquina dedicada** en el IER con modelo open-weights (Qwen 3 32B, ~32 GB RAM) → autonomía sin depender del API de Claude.
- **Encriptación** de canales (Telegram grupal no es privado por default).
- **Eliminar memoria** del bot para respetar privacidad.
- **Filtros de sesgo** — perspectiva de género, anti-clasismo, anti-racismo.

## Filosofía — complementar, no sustituir

Casos de uso esperados:

- Estudiantes con preguntas conceptuales en madrugadas o cuando el profesor no responde.
- Recordar dónde se vio un tema específico ("¿en qué clase se habló de Sunlit Fraction?").
- Ejecutar pasos repetitivos del flujo (ej. "¿cómo configurar Ideal Air Loads?").

Casos NO deseados:

- **Resolver el proyecto final** sin entender los conceptos.
- Sustituir el aprendizaje conceptual con copy-paste de respuestas.

> "La responsabilidad de aprender, qué temas decidir aprender a profundidad y con qué capacidad de autosuficiencia — depende de ustedes. La IA no las/los exime."

## Errores conocidos en el corpus

El corpus se construye desde **transcripciones automáticas de las clases** — tiene errores de ASR + errores conceptuales del profesor en vivo:

| Error observado en clase | Origen |
|--------------------------|--------|
| `iertools` se transcribe como "orejas" | ASR confunde la pronunciación |
| Ecuación de transferencia de calor mal escrita en alguna transcripción (sin $\rho c_p$) | Error del profesor en pizarrón en vivo |
| Otros bugs sutiles en código copiado | Combinación de los anteriores |

**Implicación**: el asistente puede repetir errores del corpus. Por eso necesita un proceso de **revisión y limpieza**.

## "Minar errores" — gamificación

Idea introducida en clase 011: cuando los estudiantes detecten errores del asistente:

- Reportarlos a través del propio chat.
- El asistente los registra (capacidad de escritura limitada en el sistema).
- El profesor los revisa periódicamente y **otorga puntos** como recompensa.

> Analogía con minería de Bitcoin: "antes la gente andaba minando bitcoins. Aquí van a minar errores."

Beneficio doble:

- **Mejora el corpus** y el asistente con feedback real.
- **Engancha a los estudiantes** con el aprendizaje activo (encontrar el error implica entender el concepto).

## Onboarding por screenshot (clase 012)

El alta inicial pedía mensaje directo del alumno al profesor con su usuario de Telegram. **Cambio** anunciado en la clase 012, motivado por la regla del curso de **no contacto privado profesor↔estudiante** ([[../REGLAS_CURSO]]):

1. El bot tiene nombre público fijo (ver Classroom para el nombre exacto).
2. El alumno le manda un mensaje **al bot** desde Telegram.
3. El bot responde con un saludo + ID/texto de emparejamiento.
4. El alumno hace **screenshot del mensaje del bot** (puede recortar el usuario propio para preservar privacidad) y lo sube como tarea en Classroom.
5. El profesor copia el ID a la Raspberry y autoriza al usuario.
6. Al confirmarse el acceso, el profesor regresa la tarea con la calificación — la entrega vale como confirmación de alta.

> "Algo que yo prefiero no tener es contacto directo con ustedes. Es una cuestión de ética, de cuidado."

### Falla por calor de la Raspberry

> "El bot ayer estuvo fallando. No sé si alguien lo notó o si les colapsó también por el calor la Raspberry."

La Raspberry vive físicamente en casa del profesor — sin acceso remoto desde el aula no puede reiniciarse en el momento. **Migración pendiente** a una máquina dedicada en el IER en 2-3 semanas (mismo plan futuro que ya se mencionó en la clase 011).

### Pregunta abierta — pedagogía del asistente

> "¿Cómo haces un asistente que no sea barco y que propicie el aprendizaje? Si tú le dices 'cómo hago esto', te va a contestar con código en Python y tú lo vas a copiar y pegar."

El profesor invita al grupo a proponer **roles/prompts de sistema** que hagan al asistente **preguntar de vuelta** antes de responder, en lugar de entregar la solución. Es un problema abierto del prototipo.

## Privacidad — caveats

> "Las conversaciones en Telegram grupal **no son privadas**. Yo puedo ver lo que escriben."

- Telegram grupal no encripta E2E.
- El bot **tiene memoria** por default.
- En el estado actual (prototipo) el profesor puede ver los logs.

Plan: encriptar canales, eliminar memoria, conversaciones privadas.

> "Que sepan: yo no tengo interés en ver qué platican. Sí tengo interés en evaluar el impacto cuando quieran retroalimentación. Pero ahorita es prototipo y no es seguro."

Aplicación general: igual que con ChatGPT — no compartir credenciales, datos sensibles, contraseñas.

## Por qué necesario — la limitación de escala

> "La LIER inventó esta materia. Ahora otras universidades replican el plan de estudios. Pero ¿quién les enseña a simular? Habemos muy poquitos."

Problema:

- **Pocos profesores** en México saben simular E+/Open Studio para enseñar diseño bioclimático.
- Imposible escalar las clases una persona / 30 estudiantes.
- IA bien curada **puede multiplicar** la capacidad pedagógica sin reemplazar al profesor.

El asistente es una respuesta a esa limitación de escala — habilita acceso a este conocimiento incluso cuando no hay docente disponible.

## Ya integrado en `REGLAS_CURSO`

El concepto del asistente se introdujo **desde la clase 002** ([[REGLAS_CURSO]] sección "Asistente IA del curso (RAG)") con NotebookLM y Gemini Gems como prototipos iniciales. La versión actual (clase 011) está en producción con OpenCode + Telegram.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — primera mención del proyecto del asistente
- [[../classes/011-EnerHabitatParte2]] — presentación del prototipo funcional con Telegram
- [[../classes/012-ProyectoFinal]] — onboarding por screenshot, falla por calor, pregunta abierta sobre pedagogía del asistente

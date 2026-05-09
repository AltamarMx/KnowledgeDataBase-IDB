---
title: Reglas del Curso
type: rules
clases_referencia: [001, 002, 012]
updated: 2026-05-08
tags: [reglas, operacion, curso, proyecto-final]
---

# Reglas del curso

Reglas de operación del Taller de Diseño Bioclimático establecidas en la primera clase. Aplican durante todo el semestre.

## Comunicación

- **Canal único:** Google Chat Space público con toda la clase + profesor.
- **No** WhatsApp.
- **No** mensajes privados por Classroom (no escribir al profesor por ahí).
- **Razón:** crear espacios seguros, evitar repetir respuestas, dejar registro de todo lo conversado.
- **Hilos:** responder dentro del hilo correspondiente para mantener el orden. Tema nuevo ⇒ hilo nuevo.
- Las respuestas no son inmediatas — el profesor responde cuando tiene tiempo. Si la pregunta involucra análisis de datos, adjuntar libreta o captura para que pueda contestar de una vez.
- El profesor puede responder con un video corto cuando la explicación lo amerita.

## Equipos

- Toda la parte de taller se trabaja **por equipos**.
- Una vez definidos, **no se aceptan cambios** (excepción: caso especial bajo consideración del profesor).
- **La calificación del equipo es la calificación de cada miembro** — no se separan responsabilidades individuales.
- Si un equipo entrega un ejercicio, todo el equipo recibe la puntuación. Si el equipo no asiste el día del ejercicio, pierde los puntos.
- El equipo es responsable de su organización interna y de cómo distribuye el trabajo.

## Evaluación

- **Taller + Teoría van juntos:** no pasar la parte de teoría implica no pasar la materia, aunque se pase el taller con 10.
- Las reglas de calificación se fijan el primer día y **no se cambian** durante el semestre.
- Asignación por puntos en ejercicios. Algunos ejercicios son para resolverse en clase (puntos solo para quienes asistan), otros se pueden llevar de tarea.

## Asistencia

- **No se toma asistencia.**
- Recomendación: **mejor faltar que estar distraído** — los videos quedan grabados.
- Si se tiene otro pendiente, salir del salón a hacerlo y ver el video después. No vale la pena estar en clase sin atención y luego volver a verla.

## Política de cero tolerancia a la violencia

- Aplica en **todas direcciones**: profesor↔estudiantes, entre estudiantes.
- Diferenciar humor consentido (chiste de hobby ñoño) vs. ridiculización dirigida.
- Si alguien identifica violencia, puede nombrarla.
- Recursos disponibles en el instituto: POC, Claudia.
- El registro público de los chats sirve también como evidencia si hay que levantar acta.

## Lenguaje

- Se usa lenguaje incluyente.
- Corresponsabilidad: si el profesor se excede, los estudiantes pueden señalarlo. Y viceversa.

## Uso de inteligencia artificial

- **No se prohíbe ni se persigue.** El profesor la usa activamente (paga Claude Max).
- **Recomendación durante el curso:** dejarla como **último recurso**, después de intentar resolver y de preguntar al profesor.
- **Energy Plus específicamente:** las IA generales **alucinan mucho** sobre objetos, física y opciones del programa. **No preguntar a un chat genérico sobre Energy Plus** — su conocimiento del dominio es muy malo. El asistente del curso (ver abajo) sí es seguro porque usa la documentación oficial como fuente.
- **Análisis de datos / código Python:** la IA es útil pero abusar impide aprender a analizar datos.
- Responsabilidad del estudiante: desarrollar criterio para saber **cuándo usarla y cuándo no**.

## Asistente IA del curso (RAG)

El profesor está construyendo un **asistente IA específico para el curso** basado en RAG (Retrieval-Augmented Generation): un modelo que solo responde a partir de **documentación curada**, no de conocimiento general.

### Cómo funciona

- Se le carga la documentación oficial de Energy Plus: **Input Output Reference** y **Engineering Reference**.
- El modelo solo busca y razona sobre ese material → **no alucina** sobre objetos inexistentes.
- Eventualmente se sumará el contenido de las clases (transcripciones de videos) para que pueda decir "esto se ve en la clase NNN, minuto X".

### Plataformas usadas

- **NotebookLM** (Google) — permite cargar listas de videos publicados en YouTube y hacerles preguntas. También genera podcasts de 10 minutos a partir de una clase.
- **Gems de Gemini** — equivalentes a GPTs personalizados; aceptan documentación de fondo. El profesor subió un Gem al **Classroom** del curso.

### Por qué se graban las clases

1. **Construir el material de RAG** — los videos alimentan el asistente del curso.
2. **Disponibilidad** — ver clases perdidas, repasar partes específicas, preguntar al asistente qué se vio en cada minuto.
3. **Renovación de material** — los videos previos del profesor (en "De Cero a Infinito") tienen >10 años; el proceso de simulación cambió.
4. **Tiempo del profesor** — el asistente puede responder preguntas frecuentes 24/7, liberando tiempo para casos complejos.

### Si algo te incomoda en la grabación

Avisa al profesor y él edita la sección. Si aún así no quedas conforme, díselo otra vez.

### Proyecto en proceso

Hay una **reconsideración de financiamiento** pendiente para construir el asistente en una **máquina propia del grupo** (no dependiente de Google/OpenAI). La motivación incluye:

- Reducir dependencia tecnológica externa.
- **Medir el consumo energético del modelo** — el profesor señala que hay mucha desinformación sobre el consumo de los LLMs (mito del agua: el agua se usa para enfriar en circuitos casi siempre cerrados).

## Proyecto final 2026-2

Reglas específicas del proyecto final del semestre 2026-2, fijadas en la clase 012.

### Fechas

| Fecha | Evento |
|---|---|
| 13 mayo (aprox) | Profesor entrega rúbrica |
| 22 mayo | Última clase técnica con el profesor (infiltración + ventanas complejas) |
| 29 mayo | Fin del semestre UNAM — última clase del taller |
| 3 junio | Asesoría opcional (a confirmar por equipo en el chat) |
| **5 junio 10 AM** | **Presentación + entrega del proyecto** |

### Formato

- **Reporte: máximo 5 páginas** (la portada no cuenta).
- **Google Doc preferido** sobre PDF — facilita el comentario inline. LaTeX OK pero exporta como PDF, lo que ralentiza la revisión.
- **Presentación: 15 min máx + ~10 min preguntas** por equipo. Audiencia especializada — no contextualizar conceptos básicos.
- **Etiquetar las estrategias por nombre** ("estrategia color", "estrategia ventanas"), no "estrategia 1, 2, 3".
- **Archivos**: zip del espacio de trabajo (`OSM/`, `EPW/`, `notebooks/`) + reporte + presentación. **Una persona por equipo entrega.**
- **No** se exige reproducibilidad con `uv` (todavía no es clase de reproducibilidad).
- **No le echen ganas a la portada** — el profesor sólo busca quién es el equipo.

### Evaluación

- **Total: 250 puntos** (rúbrica detallada pendiente).
- La calificación es **del equipo** — todos los miembros reciben la misma puntuación (regla general del curso).
- La persona que presenta **habla por todo el equipo** — preguntas y respuestas se evalúan colectivamente.

### Encuadre técnico

Detalle metodológico (Casa 11, CONUEE, caso base + 3 estrategias + integrado, mes crítico, métricas) en [[classes/012-ProyectoFinal]].

## Tecnología en clase

- Celular y computadora permitidos, **úsenlos con responsabilidad**.
- Es notorio cuando alguien está chateando — la expresión cambia. No se sanciona, pero se nota.

## Cambios de horario

- Miriam y Guillermo pueden mover clases por compromisos.
- Los videos quedan grabados en Classroom (no en stream temporal — ahí se ordenan).

## Recursos del curso

- **Classroom:** material organizado, videos, tareas. **No** usar Classroom para mensajes privados al profesor.
- **Google Chat Space:** comunicación pública del grupo.
- **Hackatón de visualización de datos** (extra): premio 3 computadoras al 1° y 2° lugar, ~6 semanas de duración.

## Relacionado

- Establecidas en: [[classes/001-IntroduccionTallerIDB]]
- Asistente IA del curso anunciado en: [[classes/002-ConceptosBasicosBalancesCalor]]
- Reglas del proyecto final 2026-2: [[classes/012-ProyectoFinal]]

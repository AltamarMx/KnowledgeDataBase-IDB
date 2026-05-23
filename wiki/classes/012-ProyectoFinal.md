---
title: 012 — Proyecto Final
type: clase
clase: 012
profesor: Guillermo Barrios del Valle
fuente: raw/videos/012_ProyectoFinal.md
fecha_clase: 2026-05-08
fecha_ingesta: 2026-05-08
tags: [clase, proyecto-final, logistica, metodologia, vivienda-social, conuee, asistente-ia]
aliases: [Clase 012]
---

# 012 — Proyecto Final

## Metadatos

- **Clase:** 012 (8 de mayo de 2026)
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/012_ProyectoFinal.md`
- **Tipo:** Encuadre del proyecto final (logística + metodología). No introduce conceptos nuevos de simulación; consolida los anteriores y fija reglas de entrega.

## Resumen

Tres bloques:

1. **Logística del cierre del semestre** — calendario UNAM termina el 29 de mayo; última clase con el profesor el 29 (cafecito); presentación del proyecto el **viernes 5 de junio 2026 a las 10 AM**; asesoría opcional a confirmar.
2. **Encuadre del proyecto final** — **Casa 1** del programa **Decide y Construye** (vivienda social mexicana, 60-65 m²). Cada equipo aplica un bioclima distinto, define meses críticos vía CONUEE, monta caso base + 3 estrategias + caso integrado.
3. **Onboarding nuevo del asistente IA** — protocolo via screenshot para preservar privacidad (no contacto directo profesor↔estudiante).

> "Tienen que decir, 'Tengo un clima cálido por esto'… ya estamos asumiendo que es un público especializado."

## Calendario del cierre

| Fecha | Evento |
|---|---|
| 8 mayo (clase 012) | Encuadre del proyecto final |
| 15 mayo | Sin clase con el profesor |
| 22 mayo | Clase con el profesor — infiltración + ventanas complejas |
| 27 mayo | Clase con Miriam (teoría) |
| 29 mayo | Última clase del taller (cafecito) |
| 3 junio | Asesoría opcional **a confirmar** por equipo |
| **5 junio 10 AM** | **Presentación + entrega del proyecto final** |

> "Si terminamos rápido, yo me voy. No se confíen — si cada equipo tiene una duda o quiere asesoría, preséntense todos."

La asesoría se confirma por el chat público del curso. Puede ser **presencial o por Meet**, en el horario de clase.

## Casa 1 → Casa 3 — Decide y Construye

**Decide y Construye** es un programa del gobierno mexicano que publica planos con detalles constructivos de vivienda social, algunas **progresivas** (se construyen por etapas).

> ⚠️ **Actualización clase 014 (22-may-2026):** la Casa 1 mencionada originalmente **ya no está disponible** en la página oficial del programa. El profesor cambió la edificación de referencia a la **Casa 3**. El PDF actualizado está en Classroom. Las especificaciones del caso base se mantienen idénticas — solo cambia la edificación. Ver [[014-InfiltracionFloorspaceWindowLBNL]].

- **Casa 3** (vigente): vivienda social progresiva, 60-65 m², dos plantas + techo.
- En México hay viviendas autorizadas de hasta **54 m²** ("pies de casa") — el profesor lo flagea como preocupante por la calidad espacial mínima.
- Cochera al sur en el plano (define la orientación de referencia).

> "La casa 1 pues tiene esta planta… ustedes podrían simplemente importar esto y luego dibujar." _(cita original — la edificación vigente es la **Casa 3**, ver actualización arriba)_

Algunas ligas del programa están rotas — se recomienda usar los planos disponibles tal cual sin recolectar más detalle del estrictamente necesario para el modelo simplificado.

## Especificación del caso base del proyecto

| Aspecto | Valor |
|---|---|
| Absortancia solar (todas las superficies) | **0.4** |
| Sombreado | Sin elementos |
| Aire acondicionado | Sin AC |
| Cargas térmicas internas | Sin cargas |
| Piso | **Adiabático** |
| Infiltración | **Sí** (procedimiento pendiente — el profesor grabará video de 15 min si no alcanza en clase) |
| Ventanas | Vidrio simple de **3 mm**, dimensiones según planos |
| Sub-superficies interiores | **No simular** (FloorspaceJS tiene bugs con ventanas/puertas entre zonas) |
| Muros exteriores | Capas según plano (yeso 5 cm + tabique 14 cm + acabado interior) |
| Techo / piso planta alta | Composición específica del programa Decide y Construye |
| Ventilación natural | No se modela (cambios de aire iguales para todos los casos) |

**Propiedades térmicas de los materiales**: cada equipo las busca y **reporta su fuente** — ASHRAE, Incropera, libros de transferencia de calor, [[../tools/EnerHabitat|EnerHabitat]], etc.

> "Las propiedades se recomienda que las busquen y reporten su fuente."

## Workflow del proyecto

1. **Bioclima asignado** → bajar EPW de OneBuilding ([[../procedures/Descargar-EPW-OneBuilding]]).
2. **Determinar meses críticos** con CONUEE (Comisión Nacional para el Uso Eficiente de la Energía), Climat Consultant o Python.
3. **Caso base** según especificación de arriba ([[../concepts/Caso-Base]]).
4. **Tres estrategias bioclimáticas** que **mejoren** el desempeño. Si una no mejora, descartarla y proponer otra.
5. **Caso integrado** — todas las estrategias juntas. No es lineal — puede haber sinergia o antagonismo ([[../concepts/Estudio-Parametrico]]).
6. **Análisis comparativo** en Python (`iertools`) por mes crítico.

> "Las estrategias **que mejoren**. Si proponen una estrategia que no mejora, es como que no están aplicando el concepto adecuado."

### Estrategias permitidas

| Estrategia | Parámetro | Notas |
|---|---|---|
| Color | Absortancia solar | Cambia amplitud y promedios |
| Sistemas constructivos | Capas, aislante, masa térmica | Posición del aislante importa ([[../concepts/Posicion-Aislante]]) |
| Orientación | `North Axis` en Facility | "Es tu terreno y esa es la orientación" — restricción real |
| Tamaño y posición de ventanas | WWR, ubicación | |
| Protecciones solares | Aleros, parteluces | |

**Ventilación natural NO** — está fijada por cambios de aire iguales para todos los casos; meterla sería artificial.

### Definición de meses críticos

- **Climas templados / cálidos puros**: un solo mes crítico (cálido).
- **Climas extremosos** (Monterrey, Sonora): **dos meses críticos** — cálido y frío.
- Para una vivienda real: análisis de todo el año. **Para este proyecto: solo mes(es) crítico(s)**.

> "Los meses críticos muchas veces se vuelven indispensables si uno quiere dimensionar equipos. Como aquí simulamos sin AC, basta el mes crítico, pero que sepan que la metodología real es estacional."

## Métricas requeridas

Para **cada caso × cada mes crítico**:

| Mes evaluado | Métrica de promedios | Métrica de severidad |
|---|---|---|
| Mes cálido | Promedio mensual del **máximo diario** de T aire interior | **Grados-hora cálidos** ([[../concepts/Grados-Hora-Disconfort]]) |
| Mes frío | Promedio mensual del **mínimo diario** de T aire interior | **Grados-hora fríos** |

Hay que **proponer un modelo de confort** y aplicar la banda al mes evaluado ([[../concepts/Confort-Adaptativo]]).

> "Una métrica no dice todo. Si yo cambio el aislamiento, a veces el promedio no baja, pero la amplitud sí — entonces hay que usar conocimiento porque a veces la métrica no les va a ayudar."

### Trampa pedagógica — climas extremosos

> "Para los que tienen dos meses críticos: si el problema es más frío que calor, es probable que no alcancemos a resolver todos los problemas. Si subo acá, baja allá. No es lineal."

En climas con dos meses críticos hay que **priorizar** explícitamente y reportar la decisión.

### No automatizar a ciegas

> "Automatizar las simulaciones no es buena idea. Uno tiene que estar viendo los resultados."

Una métrica puede sugerir mejora cuando la realidad física es ambigua. Por ejemplo, cambiar la absortancia mueve la **amplitud** sin necesariamente bajar el promedio; combinada con otra estrategia el efecto puede revertirse. Hay que **mirar las series temporales** (no sólo la métrica) en cada caso.

## Cuestiones de simulación específicas

- **Zonas térmicas**: cada espacio relevante. El cuarto de servicio puede no evaluarse "porque uno entra, deja las cosas y se va".
- **Promedio pesado por volumen** para reportar el desempeño global de la vivienda.
- **Verificar condiciones de frontera**: muros interiores **verdes** (Surface), exteriores **azules** (Outdoor), piso **rojo** (Adiabatic). Render By Boundary antes de correr ([[../procedures/Debuggear-Simulacion-OpenStudio]]).

## Entregables

| Pieza | Detalle |
|---|---|
| **Reporte** | Máximo **5 páginas** (la portada no cuenta). Preferido **Google Doc** sobre PDF — facilita comentarios. LaTeX OK pero exporta como PDF, lo que ralentiza la revisión. |
| **Presentación** | **15 min máx**, audiencia especializada |
| **Archivos** | Zip del **espacio de trabajo**: reporte + presentación + carpeta `OSM/` con todos los `.osm` + `EPW/` + `Notebooks/`. Pueden purgar OSMs intermedios del proceso si quieren. |
| **Rúbrica** | El profesor la entregará la **semana siguiente** (~13 mayo). Total **250 puntos**. |
| **Plazo** | **5 de junio de 2026, 10 AM** |
| **Subida** | **Una persona** del equipo entrega; no hace falta que sea reproducible con `uv` (todavía no es clase de reproducibilidad). |

> "No le echen ganas a las portadas — me interesa nada más quién es el equipo." (Anécdota: en su maestría le bajaban puntos por portadas creativas, sin rúbrica explícita.)

## Presentación — formato

- **15 min** por equipo + **~10 min** de preguntas. Cuatro equipos → ~2 horas.
- Énfasis en puntos **1 (bioclima), 3 (mes crítico) y 4 (estrategias)**.
- **Etiquetar las estrategias por nombre** ("estrategia ventanas", "estrategia color"), no "estrategia 1, 2, 3" — la etiqueta debe comunicar.
- La persona que habla **habla por todo el equipo**; las preguntas se evalúan como del equipo.
- Practicar el speech, tener notas, decidir cómo se corrigen entre sí (libertad total para organización interna).

## Onboarding nuevo del asistente IA

Hasta ahora el alta del asistente Telegram requería que el alumno mandara su usuario en un mensaje directo al profesor. **Cambio**:

1. El bot tiene nombre público fijo (ver Classroom para el nombre exacto).
2. El alumno le manda un mensaje **al bot** en Telegram.
3. El bot responde con un saludo + ID/texto de emparejamiento.
4. El alumno hace **screenshot del mensaje del bot** (puede recortar el usuario propio para preservar privacidad) y lo sube como tarea en Classroom.
5. El profesor copia el ID a la Raspberry y autoriza al usuario.

> "Algo que yo prefiero no tener es contacto directo con ustedes. Es una cuestión de ética, de cuidado."

Detalle ampliado en [[../concepts/Asistente-Virtual-RAG]].

### Falla del bot por calor

> "El bot ayer estuvo fallando. No sé si alguien lo notó o si les colapsó también por el calor la Raspberry."

La Raspberry está físicamente en casa del profesor; sin acceso remoto desde el aula, no se puede reiniciar el día de la clase. **Está pendiente** una máquina dedicada en el instituto en 2-3 semanas.

### Pregunta abierta — pedagogía del asistente

> "¿Cómo haces un asistente que no sea barco y que propicie el aprendizaje? Si tú le dices 'cómo hago esto', te va a contestar con código en Python y tú lo vas a copiar y pegar."

El profesor solicita recomendaciones del grupo sobre roles/prompts de sistema para asistentes pedagógicos — un asistente que pregunte de vuelta antes de responder.

## Anécdotas

- **Plaza en Manzanillo**: restaurante con ventanales hacia la avenida, **mar atrás** — ejemplo de orientación absurda. "Ay, aquí cabe, papas." Refuerza por qué la orientación importa, pero también por qué hay restricciones reales (terreno fijo, vista preciosa hacia el oeste, etc.).
- **Murciélago** atrapado en el aula que chocó tres veces contra el vidrio — comentado al inicio de la clase.
- **Maestría del profesor**: hace 20 años le bajaban puntos por usar hojas recicladas y por portadas creativas, sin rúbrica explícita. De ahí su énfasis en rúbricas claras y "no le echen ganas a la portada".

## Conexiones

- ← **Anterior:** [[011-EnerHabitatParte2]]
- → **Siguiente:** [[013-CalculoGradosHoraDisconfort]] (cálculo en pandas) → [[014-InfiltracionFloorspaceWindowLBNL]] (infiltración + Casa 1→Casa 3 + Window LBNL)
- → Conceptos clave consolidados:
  - [[../concepts/Caso-Base]] — reglas específicas del proyecto final
  - [[../concepts/Estudio-Parametrico]] — 3 estrategias + integrado
  - [[../concepts/Grados-Hora-Disconfort]] — métrica de severidad
  - [[../concepts/Confort-Adaptativo]] — banda de confort por bioclima
- → Procedimientos clave:
  - [[../procedures/Descargar-EPW-OneBuilding]]
  - [[../procedures/Debuggear-Simulacion-OpenStudio]]
  - [[../procedures/Comparar-Simulaciones-Python]]
  - [[../procedures/Estructura-Proyecto-Simulacion]]
- → Reglas: [[../REGLAS_CURSO]] — sección "Proyecto final 2026-2"

## Recursos mencionados

- **Decide y Construye** — programa MX de planos de vivienda social. Casa 1 originalmente; cambiada a **Casa 3** en clase 014 (la 1 dejó de estar disponible online).
- **CONUEE** — Comisión Nacional para el Uso Eficiente de la Energía; metodología para meses críticos.
- **Climat Consultant** — software para localizar temporadas críticas (alternativa).
- **OneBuilding** — fuente de EPWs.
- **Asistente Telegram del curso** — onboarding por screenshot (ver Classroom para nombre del bot).

---
title: 004 — Interpretando mensajes de simulaciones y Construction Sets
type: clase
clase: 004
profesor: Guillermo Barrios del Valle
fuente: raw/videos/004_InterpretandoMensajesSimulacionesConstructionSets.md
fecha_ingesta: 2026-05-02
tags: [clase, openstudio, debugging, err, construction-set, warm-up, measures, sql]
aliases: [Clase 004]
---

# 004 — Interpretando mensajes de simulaciones y Construction Sets

## Metadatos

- **Clase:** 004
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/004_InterpretandoMensajesSimulacionesConstructionSets.md`
- **Tipo:** Clase mixta — revisión en vivo de la tarea de un equipo + tour de mensajes de simulación + introducción a Construction Sets

## Apertura — paro del lunes

El profesor abre haciendo un llamado a los hombres del grupo a sumarse al **paro del lunes**, a no asistir a clases y participar en el seminario de las 11. Reflexión sobre violencia, feminicidios, importancia de generar espacios entre hombres para hablar de estos temas y romper "el pacto patriarcal".

> "Hagamos todo lo posible para que ustedes que decidan hacer paro lo puedan hacer sin ninguna preocupación."

(Esta apertura se documenta en `REGLAS_CURSO.md` cuando corresponda — postura del profesor sobre el clima social del aula.)

## Resumen técnico

Tres bloques principales:

1. **Revisión de tarea de un equipo** en vivo — surge el problema de **mover el OSM al folder hermano** que crea Open Studio. Demostración de cómo eso rompe el path al EPW. Refuerzo de la regla: nunca mover el OSM, dejar que cada OSM cree su propio folder hermano.
2. **Tour del archivo `.err`** — qué es Severe vs Warning, cómo leerlos, catálogo de warnings ignorables (LCA, design days), errores típicos (`outside layer not found`, `no weather file`). Introducción al flujo OSM→IDF→E+ con dos puntos de Measures.
3. **Construction Sets** — el concepto, cómo asignar masivamente constructions a superficies por (tipo × condición de frontera), asignación a la edificación desde Facility, defaults verdes vs sobreescrituras locales, columnas Sun/Wind Exposure.

Conceptos colaterales: **Warm-up Period**, **Shadow Update**, salidas SQL/CSV/HTML, **Site/Source factors**, particiones internas como masa térmica, rotación de la edificación desde Facility.

Cierra con la **tarea**: dos zonas térmicas con tres sistemas constructivos (sin ventanas todavía), pedir datos al simulador y empezar análisis con pandas.

## Objetivos de aprendizaje

- Entender el flujo **OSM → (OSM Measures) → IDF → (IDF Measures) → Energy Plus** y por qué existen dos puntos de inyección.
- Saber abrir y leer el archivo **`.err`**: distinguir Severe vs Warning, identificar el catálogo de warnings ignorables.
- Entender qué es un **[[../concepts/Construction-Set]]** y cómo asignarlo masivamente.
- Entender el **[[../concepts/Warm-up-Period]]** y por qué el día de inicio de simulación importa.
- Conocer las **salidas** que produce E+: SQL, CSV, HTML, ERR.
- Conocer el rol de los **[[../concepts/Site-Source-Factor]]** y la limitación en México.
- Saber configurar **Sun Exposure** y **Wind Exposure** para casos como estacionamientos subterráneos.

## Revisión de tarea — el caso del OSM movido

El equipo movió accidentalmente el OSM dentro del folder hermano que crea Open Studio. Cuando se vuelve a abrir el OSM desde ahí, **se rompe el path al EPW**.

Encadenamiento de errores observados:

1. **Severe**: `no weather file found` — porque el path al EPW se perdió al mover el archivo.
2. Tras re-asignar EPW (Site → Set Weather File), nuevo run.
3. **Severe**: `Construction "Cubo": missing material assignments` — la construction quedó sin material (probablemente al rehacer no se completó).
4. Tras arrastrar el material a la construction, nuevo run.
5. **Complete with 14 Warnings** — todos del catálogo LCA → ignorables → simulación válida.

> Refuerzo de regla: el OSM se queda en `OSM/` del proyecto. **Nunca** se mueve dentro del folder hermano (que se llama igual y se borra/regenera en cada Run). Detalle en [[../procedures/Estructura-Proyecto-Simulacion]].

## El flujo OSM → IDF → Energy Plus

Cuando se da `Run`, ocurre internamente:

```
OSM (texto plano, formato Open Studio)
    │
    ▼
[OSM Measures]  ← scripts Ruby que modifican el OSM
    │
    ▼
OSM modificado
    │
    ▼
Traductor → IDF (formato Energy Plus)
    │
    ▼
[IDF Measures]  ← scripts Ruby que modifican el IDF
    │
    ▼
IDF modificado
    │
    ▼
Energy Plus corre el IDF
    │
    ▼
Resultados: SQL + CSV + HTML + ERR
```

> Hay **dos oportunidades** para modificar la simulación: antes de la traducción (OSM Measures) y después (IDF Measures). Esto permite acceder a features de E+ que la GUI no expone, y hacer estudios paramétricos.

Detalle en [[../concepts/Measures]].

### Por qué importan los Measures

- **Estudios paramétricos**: una measure puede generar 4 variantes con ventanas al 25/50/75/100% del muro. Cada variante corre como simulación independiente.
- **Acceso a features no expuestas**: ventanas operables con controles avanzados, Airflow Network, controles de iluminación dinámicos.
- **Compliance** con normativas (en EE.UU. — ASHRAE 90.1, Title 24, LEED).

En el curso se mencionan brevemente; no se usan a fondo.

## Lectura del archivo `.err`

Procedimiento detallado en [[../procedures/Leer-Archivo-ERR]]. Resumen:

1. **Show Simulation** → abrir el folder de outputs.
2. Abrir `eplusout.err` con un editor de texto (Notepad / TextEdit).
3. **Buscar Severes / Fatals primero** — los Severes detienen la simulación.
4. Después, leer Warnings y filtrar.

### Severe vs Warning

| Nivel | Comportamiento | Acción |
|-------|----------------|--------|
| **Severe / Fatal** | E+ se detiene. Sin resultados. | Corregir antes de continuar. |
| **Warning** | E+ continúa con suposiciones. | Decidir si la suposición invalida la simulación. |

### Errores severos típicos

- `outside layer not found` → construction sin material.
- `no weather file found` → EPW no asignado o path roto.
- `invalid polygon` → geometría rota.

### Catálogo de warnings ignorables (en el alcance del curso)

Open Studio está pensado para ASHRAE / análisis de ciclo de vida (LCA). Genera warnings cuando faltan inputs típicos de ese flujo:

| Warning | Por qué aparece | Ignorable porque |
|---------|-----------------|-------------------|
| `No design days defined` | Espera días de diseño para HVAC sizing | El curso no dimensiona HVAC |
| `Output:Variable <X> not found` (variables LCA) | Espera outputs de ciclo de vida | El curso no hace LCA |
| `Site/Source factors not specified` | Espera factores de conversión sitio→fuente | El curso no calcula consumo neto |

Detalle y catálogo más amplio en [[../concepts/Mensajes-EnergyPlus]].

> "Open Studio está pensado para cumplir métricas de consumo de energía y de Análisis de Ciclo de Vida. Pero aquí ni siquiera hay consumo de energía y no he definido las salidas necesarias. Por eso aparecen 14 warnings — todos esos van a estar sucediendo una y otra vez y está bien."

## Warm-up Period

Cuando E+ arranca una simulación, **inicializa todas las temperaturas a 23 °C** — un valor fijo, arbitrario.

Como en una simulación dependiente del tiempo la condición inicial importa mucho (sobre todo con masa térmica), E+ **repite el primer día** hasta que el cambio de temperatura entre repeticiones cae bajo un criterio de convergencia (~0.1 °C).

Esquemáticamente:

1. Día 1 inicializado a 23 °C → termina en, p. ej., 25 °C.
2. Repetir día 1 con 25 °C → termina en 24.5 °C.
3. Repetir con 24.5 °C → termina en 24.4 °C → converge.
4. Solo entonces avanza al día 2.

> "Estoy haciendo que se le olvide la condición inicial de 23 °C. Cuántos días lo tuvo que hacer hasta alcanzar el criterio de convergencia."

### Implicación crítica: el día de inicio importa

La edificación tiene **memoria del clima reciente** (días, no semanas). El primer día simulado **no tiene** esa memoria correctamente — solo recuerda repeticiones del mismo día.

**Regla práctica**: si comparas varias simulaciones (caso base vs. variantes), **arranca todas el mismo día**. Una que empieza el 1 de enero no es comparable con una que empieza el 1 de febrero — la edificación recuerda climas distintos.

Detalle en [[../concepts/Warm-up-Period]].

## Shadow Update

E+ no recalcula sombras en cada paso temporal. Por default lo hace **cada 20 días** (visible en el `.err`):

```
Updating Shadowing Calculations, Start Date=February 03
Updating Shadowing Calculations, Start Date=February 22
```

La aproximación es razonable para análisis térmico (la trayectoria solar cambia poco día a día). Pero **no es ideal para iluminación natural** — Radiance es mejor para eso (recalcula cada hora con backward ray tracing).

Detalle en [[../concepts/Calculo-Sombramientos]].

## Salidas de Energy Plus

E+ produce los resultados en varios formatos en paralelo:

| Formato | Archivo | Uso |
|---------|---------|-----|
| **SQL** | `eplusout.sql` | Análisis con scripts (eficiente) |
| **CSV** | `eplusout.csv` | Excel (formato "medio feo") |
| **HTML** | `eplustbl.htm` | Reporte para ASHRAE/LCA |
| **ERR** | `eplusout.err` | Mensajes de simulación |

El **SQL** es el formato preferido para postprocesamiento porque normaliza los datos (descompone fechas en componentes, etc.). No se ve abriéndolo directamente — se necesita un parser.

> "Yo escribí un paquetito en Python que pueden instalar con pip y hace la lectura directa al SQL — me ahorro un paso, es mucho más rápido y tiene varias bondades."

El paquete del profesor se introducirá formalmente en clases siguientes (probablemente 005). Detalle del formato en [[../concepts/Salida-SQL-EnergyPlus]].

### Reporte HTML y Site/Source

El HTML está pensado para análisis ASHRAE. Incluye una sección **Site and Source Energy** con factores de conversión de EE.UU. precargados (~3× para electricidad).

> "México **no tiene** site source factors completos. Existen factores del Sistema Eléctrico Nacional, pero solo de transmisión — nos faltan los de eficiencia de plantas. México está en pañalitos."

Tesis mencionada de **Nachito** (egresado del IER) sobre las múltiples definiciones de edificaciones de energía cero.

Detalle en [[../concepts/Site-Source-Factor]].

## Construction Sets

El tema central de la segunda mitad. Procedimiento detallado en [[../procedures/Configurar-Construction-Set]]. Resumen:

### Qué es

Plantilla que mapea automáticamente una construction a cada superficie según:

- **Tipo de superficie** (Wall, Roof, Floor)
- **Condición de frontera** (Outdoors, Surface, Ground, Adiabatic)

Slots típicos:

| Slot | Aplica a |
|------|----------|
| Exterior Surface — Wall | Muros con Outdoors |
| Exterior Surface — Roof | Techos con Outdoors |
| Interior Surface — Wall | Muros con Surface (interzona) |
| Ground Contact — Floor | Pisos con Ground |
| Adiabatic — Floor | Pisos con Adiabatic |
| Sub Surface — Window / Door | Sub-superficies |

### Cómo se usa

1. Crear el Construction Set en Construction → Construction Sets.
2. Llenar slots arrastrando constructions desde My Model.
3. **Asignar el set a la edificación** desde Facility → Default Construction Set.
4. Verificar en Spaces → Surfaces que las constructions aparezcan en **verde** (default heredado).

### Defaults vs sobreescritura local

- **Verde** = construction viene del Construction Set. Cambia automáticamente si cambias el set.
- **Sin color (negra)** = construction sobreescrita localmente. Independiente del set.

Detalle en [[../concepts/Construction-Set]].

## Pestaña Facility — controles globales

La pestaña Facility permite además:

- **Rotar la edificación respecto al norte** (campo `North Axis`) sin tocar la geometría dibujada — útil para estudios paramétricos de orientación.
- Asignar **Default Schedule Set** (análogo a Construction Set para horarios — no se usa en el curso aún).
- Asignar **Default Space Type**.

## Sun Exposure y Wind Exposure

En la pestaña Spaces → Surfaces, dos columnas adicionales para superficies con condición Outdoors:

- **Sun Exposure** — `SunExposed` o `NoSun`.
- **Wind Exposure** — `WindExposed` o `NoWind`.

Combinaciones posibles:

| Caso típico | Sun | Wind | Comentario |
|-------------|-----|------|------------|
| Muro o techo expuesto | SunExposed | WindExposed | Default |
| **Estacionamiento subterráneo** (techo del estacionamiento = piso del edificio) | NoSun | WindExposed | Aire del estacionamiento, sin sol |
| Edificios muy pegados sin espacio entre ellos | NoSun | NoWind | Cuando uno no quiere modelar el vecino como geometría |
| Caverna / sótano técnico | NoSun | NoWind | Sin convección al ambiente |

> "Donde más se usa es en pisos que tienen estacionamiento subterráneo. No tengo cielo, no tengo exposición al sol, pero sí tengo convección. Es lo más común — un edificio con un estacionamiento de aire en lugar de simularlo como zona térmica."

Es una caricatura: la temperatura del aire del estacionamiento se asume = T_amb del EPW. Imperfecto pero pragmático.

## Particiones interiores como masa térmica

Open Studio permite agregar **particiones internas** (paredes ligeras, cubículos, mobiliario virtual) que **no separan zonas** pero sí **agregan masa térmica al modelo**.

> "Esa idea de que las casas abandonadas son frías es porque no hay masa térmica. La masa térmica permite almacenar energía y luego liberarla lentamente. Una casa sin muebles tiene menos masa térmica — los muebles, libros, todo lo que esté ahí genera masa térmica y eso permite que las variaciones de temperatura sean menores."

En E+ esto se representa con el objeto `InternalMass`. Útil para:

- Modelar mobiliario sin dibujarlo.
- Modelar cubículos en oficinas (paredes ligeras a media altura).
- Compensar masa "perdida" cuando se simplifica un sistema constructivo heterogéneo.

Ver [[../concepts/Masa-Termica]].

## Tip — colocar la edificación cerca del origen

Detalle de UX que ahorra tiempo:

> "Coloquen su casita siempre lo más cerca del 0,0, porque luego queda muy lejos del 0,0 y no puedo hacer mucho zoom."

Si se dibujó sobre el mapa de OpenStreetMap (clase 003), las coordenadas pueden quedar lejos del origen y el zoom del preview 3D no funciona bien. Si pasa: redibujar cerca del origen, o usar un measure que reposicione la geometría.

## Tarea de la semana

> **Dos zonas térmicas** con **tres sistemas constructivos** distintos (sin ventanas todavía). El profesor enviará el esquema con medidas y alturas en la tarde-noche.
>
> Adicional: **pedir datos** al simulador (output variables) y empezar a **analizar series temporales con pandas**.

Cálculo de volumen de datos esperado:

- 2 zonas térmicas
- Paso temporal de 10 minutos → 144 datos/día
- Año completo → ~52,560 datos/zona/variable
- Múltiples variables → cientos de miles de filas → **pandas obligado**

Próxima clase: introducción al análisis con Python.

## Otros temas mencionados

### Versionado de Open Studio

> "Las versiones nuevas son capaces de abrir versiones anteriores, pero versiones anteriores no son capaces de abrir versiones nuevas."

El profesor descubre durante la clase que tiene una versión vieja de Open Studio (3.10) y los estudiantes pueden tener 3.11. Tiene que actualizar para abrir la tarea. Refuerzo de la regla del grupo: **todos en la misma versión**.

### Comunicación asíncrona

El profesor revisa pendientes a las 22-23 h, a veces madrugada. Anima al grupo a usar el chat sin pena por la hora.

> "Las comunicaciones asíncronas no son el futuro, son el presente. Aprendan a plantear un problema de manera adecuada en tres minutos."

### Classroom — feedback con video

Google Classroom permite responder con **mensajes de voz** o **screen recordings**. El profesor planea usarlos para retroalimentar tareas. Anima al grupo a usar Classroom porque "permite ese tipo de interacciones que están a tres clicks de alcance".

## Conceptos derivados (referencias)

Conceptos nuevos introducidos:

- [[../concepts/Mensajes-EnergyPlus]] — Severe vs Warning, lectura del `.err`
- [[../concepts/Construction-Set]] — plantilla de asignación masiva
- [[../concepts/Measures]] — scripts que modifican OSM/IDF
- [[../concepts/Warm-up-Period]] — convergencia de la condición inicial
- [[../concepts/Calculo-Sombramientos]] — actualización cada 20 días
- [[../concepts/Salida-SQL-EnergyPlus]] — formatos de salida
- [[../concepts/Site-Source-Factor]] — energía sitio vs fuente

Conceptos profundizados:

- [[../concepts/Masa-Termica]] — particiones interiores y mobiliario como masa térmica
- [[../concepts/Condiciones-de-Frontera]] — Sun y Wind Exposure como dimensiones independientes

## Conexiones

- ← **Anterior:** [[003-MiPrimeraSimulacion]] — primer modelo, primer Run
- → **Siguiente:** _005-AnalisisSimulacionesPython_ — análisis con pandas, paquete del profesor para leer SQL
- → Procedimientos clave:
  - [[../procedures/Leer-Archivo-ERR]]
  - [[../procedures/Configurar-Construction-Set]]
  - [[../procedures/Debuggear-Simulacion-OpenStudio]]

## Recursos mencionados

- **NREL BCL** (Building Component Library) — repositorio público de measures.
- **Tesis de Nachito** (egresado IER, Acapulco) — sobre definiciones de edificaciones de energía cero.
- **CFE** — fuente de consumo eléctrico en México (limitada — solo refleja transmisión).
- **Sistema Eléctrico Nacional** — factores oficiales en México (incompletos).
- **DB Browser for SQLite** — para inspeccionar el SQL manualmente (no se nombró pero es la herramienta estándar).

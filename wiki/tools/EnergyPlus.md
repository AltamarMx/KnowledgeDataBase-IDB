---
title: Energy Plus
type: herramienta
tags: [herramienta, motor, software-libre, energyplus]
aliases: [EnergyPlus, Energy+, E+]
clases: [001, 002, 003, 004, 005, 006]
updated: 2026-05-02
---

# Energy Plus

## Qué es

**Motor de cálculo libre** para simulación energética de edificaciones. Resuelve el [[../concepts/Balance-de-Calor]] dependiente del tiempo a través de la [[../concepts/Envolvente-Arquitectonica]], dadas unas condiciones climáticas.

Es el **kernel** detrás de [[Open-Studio]] (y también el motor que usan Design Builder, OpenStudio SDK, varios plugins de Rhino+LadyBug, etc.). En este curso **no se instala por separado** — viene dentro de Open Studio.

## Archivos de entrada

- **IDF (Input Data File)** — archivo de **texto plano** que describe la edificación como una lista de **objetos** (Site:Location, Material, Construction, BuildingSurface, Zone, Schedule, etc.).
- **EPW (Energy Plus Weather)** — archivo de **clima** con:
  - Ubicación geográfica (latitud, longitud, elevación, uso horario)
  - Datos horarios (o sub-horarios): T_amb, radiación global/directa/difusa, HR, viento (vel/dir), lluvia, presión atmosférica
  - Cambiar el EPW = mover la edificación a otro clima

## Cómo se ejecuta

Energy Plus tiene una interfaz "ñoña" (descripción del profesor): una ventanita donde se selecciona el IDF + EPW y se ejecuta. No es interactiva como Ansys.

Por eso normalmente se opera vía Open Studio (que internamente llama a Energy Plus).

## IDF Editor

Energy Plus incluye un **IDF Editor** (interfaz tabular) para editar el IDF sin tocar el texto. Útil para:

- Acceder a objetos que Open Studio no expone.
- Auditar exactamente qué se está mandando al motor.

## Módulos principales

Energy Plus está compuesto por módulos que resuelven cada parte del problema. Los relevantes para diseño bioclimático:

| Módulo | Qué resuelve | Uso en el curso |
|--------|--------------|-----------------|
| **Conduction Transfer Function (CTF)** | Conducción 1D dependiente del tiempo a través de muros — solución semi-analítica con funciones de transferencia (tipo serie de Fourier). Solución instantánea, eficiente. | **Default — se usa.** |
| **Diferencias finitas (Conduction FD)** | Alternativa a CTF: discretización temporal y espacial. Necesario para materiales con cambio de fase o conductividad variable. | No se usa. |
| **Window glass** | Transferencia de calor a través de ventanas. Modelos sencillos y complejos; soporta marcos, capas múltiples, intercambio radiativo onda corta y onda larga, ventanas dobles con interacción convectiva o semi-vacío. | Se usa. |
| **Shading** | Cálculo geométrico de obstrucciones sobre ventanas y superficies (aleros horizontales y verticales, vecinos, vegetación). | Se usa. |
| **Sky model** | Discretización de la semiesfera del cielo en parches (uno de ellos es el sol con radiación directa, los demás difusos). Modelo típico: 156 parches. | Default — se usa. |
| **Day Lighting** | Iluminación natural — niveles de iluminancia interior, deslumbramiento. | No se usa (Radiance es mejor para esto). |
| **Air Heat Balance / Mass Balance** | Balance de energía y masa del aire en cada zona térmica. | Se usa. |
| **Surface Heat Balance Manager** | Coordina los tres componentes (radiación onda corta + onda larga + convección) en cada superficie. | Se usa. |
| **Airflow Network** | Modelo más complejo de E+ para ventilación natural: resuelve flujos por presión incluso con velocidad de viento cero (efecto chimenea por diferencia de densidades). | **No se usa en el curso** — demasiado complejo. |
| **HVAC** | Aire acondicionado: dimensionamiento, consumo, sistemas. | Limitado (en taller); más en Energía en Edificaciones. |
| **Photovoltaics** | Módulos FV (single-diode, etc.) acoplados al cálculo de radiación incidente sobre superficies. | Mencionado, no se usa. |
| **District Heating/Cooling** | Sistemas centralizados de distribución de calor/frío. | Mencionado, no se usa. |

> **Filosofía:** todo en Energy Plus está **validado** — han hecho experimentos y mediciones para cada módulo. Es complejo pero auditable, no caja negra.

## Restricciones fundamentales

Estas restricciones limitan lo que se puede modelar:

1. **Flujo de calor 1D perpendicular a la superficie.** No hay flujo lateral entre superficies. Implicación: los **puentes térmicos** en cambios de material no se capturan bien (zonas conocidas como "vanos"). El acoplamiento entre muros adyacentes pasa solo por **convección** con el aire interior, no por conducción esquina-a-esquina.
2. **Solo líneas rectas y superficies planas.** No hay superficies curvas, no hay ventanas circulares. Razón: el [[../concepts/Factor-de-Vista]] de una superficie consigo misma se asume **cero**, lo que solo es cierto para planos.
3. **Material homogéneo en cada superficie.** Para representar trabes embebidas o cambios laterales de material, hay que subdividir la superficie o usar el objeto `InternalMass` (ver [[../concepts/Masa-Termica]]).
4. **Mezclado perfecto del aire** en cada zona térmica — toda la zona, una sola temperatura instantánea. Sin estratificación, sin plumas térmicas. Detalle en [[../concepts/Mezclado-Perfecto]].
5. **Radiación interior distribuida uniformemente** — la radiación solar/visible que entra por una ventana o un proyector no se proyecta sobre la superficie real (la directa se asume al piso, la difusa se reparte uniformemente). Modificable con `FullInteriorAndExterior`. Detalle en [[../concepts/Radiacion-Interior-Distribuida]].
6. **Sub-superficies (ventanas, puertas) deben vivir dentro de una superficie** y no pueden ocupar el 100% de su superficie padre. Detalle en [[../concepts/Subsuperficie]].
7. **Tres tipos de superficie**: Wall, Roof, Floor — el coeficiente convectivo depende del tipo. Detalle en [[../concepts/Tipos-Superficie]].

> Conjunto de simplificaciones = **caricatura computacional**. Ver [[../concepts/Caricatura-Computacional]].

## Time steps

- **Mínimo simulable:** un día.
- **Resolución típica:** horaria.
- **Resolución máxima:** cada minuto → ~525,600 pasos/año por variable.
- **Resolución típica del curso:** cada 10 minutos → 144 pasos/día → ~52,560 pasos/año/variable.

Cada variable solicitada al output produce una serie temporal de ese tamaño.

## Warm-up Period

Antes de simular, E+ inicializa todas las temperaturas a **23 °C** y repite el primer día hasta que la diferencia entre repeticiones converge (~0.1 °C por default). Recién entonces avanza al día siguiente.

Implicación práctica: si comparas varias simulaciones, **arrancalas todas el mismo día** — la edificación tiene memoria del clima reciente que el warm-up no reproduce. Detalle en [[../concepts/Warm-up-Period]].

## Shadow Update

E+ recalcula máscaras de sombramiento **cada 20 días** por default (no cada paso temporal). Aproximación razonable para análisis térmico; insuficiente para iluminación natural fina (Radiance es la alternativa). Detalle en [[../concepts/Calculo-Sombramientos]].

## Mensajes y debugging

E+ produce un archivo `eplusout.err` con tres niveles:

- **Severe / Fatal** — detiene la simulación.
- **Warning** — continúa con suposiciones; el usuario decide si la suposición es aceptable.
- **Info** — informativo.

Detalle de catálogo y lectura en [[../concepts/Mensajes-EnergyPlus]] y [[../procedures/Leer-Archivo-ERR]].

## Salidas

E+ produce los resultados en varios formatos en paralelo:

| Formato | Archivo | Uso |
|---------|---------|-----|
| **SQL** | `eplusout.sql` | Análisis con scripts (preferido) |
| **CSV** | `eplusout.csv` | Excel ad hoc |
| **HTML** | `eplustbl.htm` | Reporte ASHRAE/LCA |
| **ERR** | `eplusout.err` | Mensajes |
| **RDD** | `eplusout.rdd` | Catálogo de variables disponibles para reportar — ver [[../concepts/RDD-Variables-Disponibles]] |
| **MDD** | `eplusout.mdd` | Catálogo de medidores de consumo |
| **EIO** | `eplusout.eio` | Informe de inicialización |

Detalle del SQL en [[../concepts/Salida-SQL-EnergyPlus]]. El reporte HTML incluye [[../concepts/Site-Source-Factor|factores Site/Source]] precargados de EE.UU. — México no tiene factores oficiales completos.

## Pedir variables al output

E+ solo reporta variables explícitamente solicitadas. Hay tres vías:

1. Pestaña **Output Variables** de Open Studio (subset limitado).
2. **Reporting Measures** del BCL — `Add Output Variable` y `Create CSV Output`. Vía recomendada en el taller. Procedimiento en [[../procedures/Solicitar-Output-Variables-Measures]].
3. Editar el IDF directamente (`Output:Variable,*,<nombre>,Timestep;`).

El catálogo de qué pedir se descubre leyendo el RDD post-simulación. Catálogo de variables comunes en [[../concepts/Variables-Output-EnergyPlus]].

## Materiales de ventana — categorías propias

Las ventanas no usan los mismos objetos de material que los muros. E+ tiene categorías propias:

| Objeto | Para qué |
|--------|----------|
| `WindowMaterial:Glazing` | Una capa de vidrio caracterizada espectralmente (transmitancia, reflectancia frontal/trasera, conductividad) |
| `WindowMaterial:Gas` | Una capa de gas entre vidrios (aire, argón, kriptón) |
| `WindowMaterial:SimpleGlazingSystem` | Una "ventana completa" caracterizada por **U-factor + SHGC + Visible Transmittance** — recomendado para el taller |
| `WindowProperty:FrameAndDivider` | El marco (framing) — espesor, conductividad, divisores |

Detalle en [[../concepts/Ventanas]]. Procedimiento de uso en [[../procedures/Agregar-Ventanas-OpenStudio]].

## Superficies de sombramiento

E+ tiene un objeto `Shading:*` para aleros, parteluces, vecinos, vegetación. Estos objetos:

- Bloquean radiación directa y difusa.
- **No** transfieren calor.
- **No** obstruyen el viento.

Detalle en [[../concepts/Superficies-de-Sombramiento]].

## Capa límite atmosférica

E+ ajusta la T del EPW (medida típicamente a 10 m) a la altura del centroide de la zona térmica antes de calcular convección. Por eso hay variables `Site:*` (clima crudo) y variables `Outdoor:*` (ajustadas). Detalle en [[../concepts/Capa-Limite-Atmosferica]].

## Archivo EPW y TMY

El archivo de clima es un **EPW** (Energy Plus Weather) — texto plano con datos horarios o sub-horarios de:

- Temperatura del aire, humedad relativa
- Velocidad y dirección del viento
- Radiación global, directa, difusa
- Lluvia, presión atmosférica
- Ubicación geográfica (lat, lon, elevación, uso horario)

El contenido típico es un **TMY** (Typical Meteorological Year) — ver [[../concepts/TMY]] para construcción y limitaciones (no es promedio, suaviza anomalías, pierde efecto de cambio climático).

**Repositorios:**

- **OneBuilding.org** — colección global.
- Construcción local desde estación meteorológica (Temixco) — pierde el carácter "típico".
- Año típico solar para Temixco (Jesús Quiñones, ANES).

## Versionado

- Energy Plus saca **un release cada 6 meses** (dos por año).
- Antes la nomenclatura era 1, 2, 3, ... 9.
- Desde hace varios años la nomenclatura es **por año**: 25.1, 25.2, 26.1, 26.2.
- Cada versión de Open Studio viene atada a una versión específica de Energy Plus.

**Importante:** mantenerse actualizado porque cada versión:

- Corrige errores
- Agrega nuevos features

El profesor revisa los release notes de cada versión nueva.

## Documentación oficial

Documentos disponibles como PDFs (~miles de páginas):

| Documento | Uso |
|-----------|-----|
| **Input Output Reference** (~2952 pp.) | Referencia de objetos de entrada y variables de salida. **Uso principal en el curso.** |
| **Engineering Reference** | Ecuaciones, correlaciones y métodos numéricos. **Uso principal en el curso.** |
| Getting Started | Visión general |
| Auxiliary Programs | Documentación del archivo de clima (EPW), entre otros |
| EMS Application Guide | Energy Management System — para programar comportamientos personalizados (no se usa en este curso) |

> **Filosofía del profesor:** no hay que aprenderse la documentación de memoria — hay que aprender a **consultarla**. Estima que conoce ~30% de Energy Plus después de 15+ años, especializado en sistemas naturalmente ventilados.

## Cobertura del curso

En este curso se usa solo una fracción de Energy Plus. **No se usan:**

- Ventilación natural (limitación del curso, no del programa)
- Cargas térmicas internas
- Geometrías complejas (limitación de Open Studio)
- Cambio de fase
- Sistemas electromecánicos avanzados

Estos temas se cubren en **Energía en Edificaciones** (siguiente materia).

## Sobre IA y Energy Plus

> **Advertencia explícita del profesor:** las IA **alucinan mucho** sobre Energy Plus — inventan objetos que no existen, dan física incorrecta, mezclan versiones. No usar IA para preguntar sobre Energy Plus. Para análisis de datos / código Python sí es útil.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción al motor y a sus archivos
- [[../classes/002-ConceptosBasicosBalancesCalor]] — módulos, restricciones, balance de superficie, EPW/TMY
- [[../classes/003-MiPrimeraSimulacion]] — balance interior, mezclado perfecto, suposiciones de radiación interior, tipos de superficie y `h_c`
- [[../classes/004-InterpretandoMensajesConstructionSets]] — `.err`, warm-up, shadow update, salidas SQL/CSV/HTML, Site/Source factors
- [[../classes/005-AnalisisSimulacionesPython]] — RDD, catálogo de variables, T operativa, capa límite, postprocesamiento con `iertools`
- [[../classes/006-DosZonasTermicasVentanasAleros]] — materiales de ventana (Glazing y SimpleGlazingSystem), framing, superficies de sombramiento

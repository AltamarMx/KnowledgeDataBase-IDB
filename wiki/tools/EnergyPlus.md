# EnergyPlus

Motor de cálculo (kernel) para simulaciones energéticas de edificaciones. Software libre desarrollado por el Departamento de Energía de EE.UU. Es uno de los dos grandes programas de simulación energética de código abierto (el otro es TRNSYS, versión europea).

**Sitio web:** energyplus.net

## Características

- Resuelve balances de energía y masa dependientes del tiempo
- Modela transferencia de calor: conducción, convección, radiación
- Lee dos archivos de entrada: **IDF** (edificación) y **EPW** (clima)
- Interfaz propia muy básica (seleccionar IDF, EPW y ejecutar)
- Editor de IDF incluido
- Releases cada 6 meses (ej. 26.1, 26.2)

## Documentación

| Documento | Páginas | Uso |
|-----------|---------|-----|
| **Input/Output Reference** | ~2,952 | Qué significan las entradas/salidas, opciones, objetos |
| **Engineering Reference** | ~1,800 | Ecuaciones, correlaciones, métodos numéricos |
| Getting Started | — | Introducción general |
| Auxiliary Programs | — | Descripción de archivos de clima (EPW), etc. |

Los dos documentos principales del curso son Input/Output y Engineering Reference. No hay que memorizarlos, sino aprender a **consultarlos**.

## Archivos

- **IDF** (Input Data File) — texto plano con objetos que definen la edificación
- **EPW** (Energy Plus Weather) — datos climáticos horarios + ubicación geográfica

## Archivos de salida

| Archivo | Extensión | Contenido |
|---------|-----------|-----------|
| **RDD** | `.rdd` | Diccionario de variables de salida disponibles para esa simulación |
| **SQL** | `.sql` | Base de datos con todos los resultados (ruta estable: `run/eplusout.sql`) |
| **ERR** | `.err` | Warnings y errores de la simulación |
| **CSV** | `.csv` | Resultados exportados (requiere measure Create CSV Output) |

### Variables de salida (RDD)

El archivo RDD lista las variables que EnergyPlus puede reportar para una simulación específica. Si un componente no fue modelado, sus variables no aparecen. Las variables se organizan por alcance:

- **Site** — clima (del EPW): `Site Outdoor Air Drybulb Temperature`, radiación solar
- **Zone** — zona térmica: `Zone Mean Air Temperature`, `Zone Operative Temperature`
- **Surface** — por superficie (inside/outside): temperaturas superficiales, radiación incidente, coeficientes convectivos

**Cuidado:** distinguir entre variables en Watts (rate) y Joules (gain).

## En el curso

- No se usa directamente (se accede a través de Open Studio)
- Se estudia la física y documentación para entender qué hace Open Studio "detrás"
- Cuando Open Studio no puede hacer algo, se exporta el IDF y se trabaja en Energy Plus
- El profesor estima conocer ~30% de las capacidades de Energy Plus (enfoque en edificaciones naturalmente ventiladas)
- Los resultados se analizan desde el SQL usando ear_tools en Python

## Nota sobre IA

El profesor advierte que la IA (LLMs) tiene muy mala comprensión de Energy Plus: alucina con objetos que no existen y su entendimiento de la física es deficiente. No se recomienda consultarla para dudas de simulación.

## Warm-up Period

EnergyPlus inicia todas las temperaturas a 23°C y repite el primer día de simulación hasta converger a un estado oscilatorio permanente (criterio ~0.1°C). Ver [[Warm-up-Period]].

## Cálculo de sombras

Por defecto, las máscaras de sombramiento se recalculan **cada ~20 días**. Se puede forzar cálculo diario pero es más lento. Esta simplificación hace que EnergyPlus no sea ideal para iluminación natural — Radiance es preferible para eso.

## Aparece en

- [[001-IntroduccionTallerIDB]] — Presentación del ecosistema y documentación
- [[004-InterpretandoMensajesConstructionSets]] — Archivo ERR, warm-up, cálculo de sombras, SQL, site source factors
- [[005-AnalisisSimulacionesPython]] — Variables de salida (RDD), SQL, documentación Input/Output Reference
- [[006-DosZonasTermicasVentanasAleros]] — Materiales de vidrio (Glazing vs Simple Glazing), superficies de sombramiento, marcos
- [[007-CasoBaseAleros]] — Radiación solar incidente por superficie (Surface Outside Face Incident Solar Radiation Rate Per Area)
- [[008-ShadingVentanas]] — Sunlit Fraction, quirk de radiación en sub-superficies, algoritmo de sombramiento (Engineering Reference)

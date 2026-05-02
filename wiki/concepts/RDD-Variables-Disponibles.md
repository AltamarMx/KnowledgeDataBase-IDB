---
title: RDD — Report Data Dictionary
type: concepto
tags: [concepto, energyplus, output, rdd, variables]
aliases: [rdd, eplusout.rdd, report data dictionary, variables disponibles]
clases: [005]
updated: 2026-05-02
---

# RDD — Report Data Dictionary

## Qué es

Archivo `eplusout.rdd` que **lista todas las variables que la simulación puede reportar** al output. Lo genera Energy Plus al ejecutar — no antes. Se ubica en el folder `run/` del OSM (mismo lugar que `.err`, `.sql`, `.csv`).

> Es el catálogo personalizado de variables disponibles para **esa simulación específica**. No se puede pedir nada que no esté en el RDD.

## Por qué el RDD depende de la simulación

E+ **solo expone variables que tienen sentido** para los objetos presentes en el modelo:

- Sin equipos HVAC → no aparecen variables de aire acondicionado.
- Sin ventanas → no aparecen variables de ventana.
- Sin schedules de iluminación → no aparecen variables de iluminación.

Esto convierte al RDD en una **herramienta de validación**: si el usuario "agregó un AC" pero no ve variables de AC en el RDD, algo en la configuración del HVAC no quedó.

## Estructura de cada línea

Cada línea del RDD tiene la forma:

```
Output:Variable,*,<Nombre exacto de la variable>,<frecuencias disponibles>;  !- <unidades>
```

Ejemplo:

```
Output:Variable,*,Site Outdoor Air Drybulb Temperature,Hourly; !- [C]
Output:Variable,*,Zone Mean Air Temperature,Zone Timestep; !- [C]
Output:Variable,*,Surface Outside Face Incident Solar Radiation Rate per Area,Zone Timestep; !- [W/m2]
```

Componentes:

- **`*`** — wildcard del objeto (zona, superficie, etc.). Se reemplaza al pedir la variable por el nombre específico.
- **Nombre exacto** — debe copiarse **idéntico** al pedirla (espacios, mayúsculas, sin coma final). Errores de un caracter rompen la solicitud.
- **Frecuencia disponible** — la mayoría está en `Zone Timestep` (cada paso temporal); algunas variables auxiliares solo aparecen en `Hourly` o `Detailed`.
- **Unidades** — entre corchetes después de `!-`.

## Familias típicas de variables

Buscar por prefijo en el RDD ayuda a navegar. Detalle del catálogo en [[Variables-Output-EnergyPlus]].

| Prefijo | Familia |
|---------|---------|
| `Site:` | Clima del EPW (T exterior, radiación, viento) |
| `Zone:` | Variables a nivel de zona térmica (T media del aire, T operativa, ganancias) |
| `Surface Outside Face:` | Cara exterior de cada superficie (T, radiación incidente, h_c) |
| `Surface Inside Face:` | Cara interior de cada superficie |
| `Surface:` (sin sufijo) | Variables de la superficie como un todo |

## El asterisco — wildcard

Cuando se pide una variable con `*` como objeto, E+ devuelve esa variable para **todos** los objetos compatibles (todas las superficies, todas las zonas).

Ventaja: no hay que listarlas una por una.

Riesgo: **explosión de columnas** en simulaciones grandes. Un edificio con 200 superficies + `Surface Outside Face Temperature *` = 200 columnas, cada una con ~52,560 puntos.

> **Recomendación**: para análisis específicos de una superficie, **darle nombre descriptivo a esa superficie** (ej. `Techo`, `MuroNorte`) y pedir la variable para ese nombre, no `*`.

Para asignar un nombre específico a una superficie en Open Studio: pestaña Spaces → Surfaces → columna `Name` → escribir nombre sin acentos/eñes/espacios.

## Lectura típica del RDD

1. Tras `Run`, **Show Simulation** → abrir `eplusout.rdd` con un editor de texto (Notepad / TextEdit).
2. Buscar (`Ctrl+F`) por keyword: `temperature`, `radiation`, `convection`, `solar`.
3. Verificar que las variables esperadas están listadas.
4. Copiar el nombre exacto al configurar el measure de output (ver [[../procedures/Solicitar-Output-Variables-Measures]]).

## RDD vs documentación oficial

El RDD lista **qué variables existen** en esta simulación. Para entender **qué significa** cada variable, consultar la **Input/Output Reference** de Energy Plus (Ctrl+F con el nombre de la variable). Detalle en [[../tools/EnergyPlus]].

## Otros archivos diccionario relacionados

- **`.mdd`** (Meter Data Dictionary) — análogo al RDD pero para **medidores** (consumos agregados por uso final). Usado cuando se modela HVAC y se quiere desglosar consumo.
- **`.eio`** — informe de inicialización con resúmenes de la edificación al arrancar.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — primera lectura del RDD para descubrir qué variables están disponibles

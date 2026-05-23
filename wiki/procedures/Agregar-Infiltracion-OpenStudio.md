---
title: Agregar Infiltración en Open Studio
type: procedimiento
tags: [procedimiento, openstudio, infiltracion, ach, schedule, space-type, proyecto-final]
aliases: [agregar infiltración, configurar ACH, infiltración constante, cambios de aire openstudio]
clases: [014]
updated: 2026-05-22
---

# Agregar Infiltración en Open Studio

Procedimiento completo para configurar [[../concepts/Infiltracion-Cambios-Aire|cambios de aire constantes]] (0.5 ACH típicos) en un modelo. Demostrado en [[../classes/014-InfiltracionFloorspaceWindowLBNL]].

## Resumen

7 pasos:

```
Schedule (fraccional 0-1)
        │
        ▼
Space Type (contenedor de cargas)
        │
        ▼
Arrastrar "Space Infiltration Design Flow Rate" desde Library
        │
        ▼
Editar: cambiar a "Air Changes per Hour", poner coeficientes [1, 0, 0, 0]
        │
        ▼
Asignar Schedule al Space Type → Loads
        │
        ▼
Asignar Space Type a los espacios deseados
        │
        ▼
Verificar en RDD + solicitar variable al output
```

## Paso 1 — Crear Schedule fraccional

`Schedules → Schedule Sets → New`:

1. **Type**: `Fractional` (rango 0-1). Alternativa equivalente: `Dimensionless` con valores también 0-1.
2. **Name**: descriptivo (e.g. `ventilacion_constante` o `ventilacion_nocturna`).
3. **Default Day Profile**: poner el valor que se quiere a lo largo del día.
4. **Importante**: para que pandas/E+ tomen el cambio, **dale Enter** al valor — si no se confirma, no queda guardado.

### Para infiltración constante

- Un solo valor: `1` en todo el día.
- Sin cambios por fin de semana ni estación.

### Para ventilación nocturna

Schedule típico (24 h):

| Hora | Valor |
|---|---|
| 00:00 – 06:00 | 1 (ventilando) |
| 06:00 – 22:00 | 0 (sin ventilación) |
| 22:00 – 24:00 | 1 (ventilando) |

> "Que no les pase lo que me pasó: le quise poner un 2 y la fracción va entre 0 y 1. Te marca un error descriptivo."

## Paso 2 — Crear Space Type

`Space Types → Space Types Tab → +`:

1. **Name**: descriptivo (e.g. `cuarto_ventilado`).
2. No es obligatorio que tenga otras cargas; este Space Type solo cargará la infiltración.

> "El tipo de espacio es como una función en Python — lo defines y no tiene efecto hasta que lo aplicas a un espacio."

Detalle en [[../tools/Open-Studio#space-types]].

## Paso 3 — Arrastrar Infiltration desde Library

Dentro del Space Type creado:

1. Pestaña inferior: **`Loads`**.
2. Panel lateral: **`Library`**.
3. Filtrar/buscar: aparecen dos opciones de infiltración:
   - `Space Infiltration Design Flow Rate` ← **usar este**
   - `Space Infiltration Effective Click Leakage Area` ← más físico, requiere medición experimental
4. **Arrastrar** `Space Infiltration Design Flow Rate` al panel `Loads` del Space Type.

> "Open Studio no me deja crear uno nuevo desde cero. Tengo que agarrar algo que ya existe y modificarlo."

### Por qué hay que arrastrar

Open Studio no permite crear objetos de infiltración vacíos por la GUI — solo desde la biblioteca. El objeto arrastrado se duplica internamente para que se pueda editar sin afectar la biblioteca.

## Paso 4 — Editar el objeto de infiltración

Doble-clic en el objeto arrastrado:

| Campo | Valor para ACH constante |
|---|---|
| **Name** | Renombrar (e.g. `ventilacion_constante_05`) |
| **Design Flow Rate Calculation Method** | `Air Changes per Hour` |
| **Air Changes per Hour** | `0.5` (o el valor deseado) |
| **Flow per Space Floor Area** | (vacío) |
| **Flow per Exterior Surface Area** | (vacío) |
| **Constant Coefficient (A)** | `1` |
| **Temperature Coefficient (B)** | `0` |
| **Velocity Coefficient (C)** | `0` |
| **Velocity Squared Coefficient (D)** | `0` |

Recordar la ecuación enriquecida ([[../concepts/Infiltracion-Cambios-Aire#la-ecuación-enriquecida-de-energy-plus]]):

$$
Q_{inf}(t) = Q_{diseño} \cdot S(t) \cdot \left[ A + B\,|\Delta T(t)| + C\,v(t) + D\,v(t)^2 \right]
$$

Con `[A,B,C,D] = [1, 0, 0, 0]` y schedule en 1, el flujo es **prácticamente constante** (varía en el tercer decimal por cambios de densidad del aire).

## Paso 5 — Asignar el Schedule al Space Type

En el Space Type:

1. Pestaña `Loads`.
2. Fila del objeto de infiltración recién agregado.
3. Columna `Schedule` → arrastrar el Schedule creado en el Paso 1.

Si no se asigna Schedule, E+ asume `Always On = 1` por default (puede funcionar pero es mejor explícito).

## Paso 6 — Asignar el Space Type a los espacios

`Spaces → My Model`:

1. Para cada Space al que se quiere agregar infiltración:
   - Columna `Space Type` → arrastrar el Space Type creado.
2. Verificar visualmente que el Space Type aparece en cada Space.

> "Aquí también pueden definir perfiles de ventilación, ocupación, iluminación, entrada/salida de personas y nada más lo asignan. ASHRAE tiene un montón de esas cosas para oficinas chicas, medianas, grandes, hospitales — sobre todo porque tienen ya cargas térmicas lumínicas incluidas."

## Paso 7 — Solicitar variables al output y verificar

### Vía 1 — Pedir variables manualmente

1. `Output Variables → Add Variable`.
2. Buscar: `Zone Infiltration` (aparecen 6 variables relacionadas — ver tabla abajo).
3. Seleccionar las que interesen, **Timestep** como frecuencia.

### Vía 2 — Pedir todas (Wildcard)

`Add Output Variable` con Key Value `*` y nombre `Zone Infiltration *` solicita todas. ⚠️ Cuidado con el wildcard si hay shading — puede generar `Mir-FACE` ([[../concepts/Algoritmo-Sombreamiento#mirror-surfaces-mir-face]]).

### Variables disponibles

| Variable | Unidades | Para qué |
|---|---|---|
| `Zone Infiltration Standard Density Volume Flow Rate` | m³/s | Flujo volumétrico con densidad estándar |
| `Zone Infiltration Outdoor Density Volume Flow Rate` | m³/s | Flujo con densidad exterior real |
| `Zone Infiltration Mass Flow Rate` | kg/s | Flujo másico |
| `Zone Infiltration Air Change Rate` | 1/h | ACH directo — útil para verificar |
| `Zone Infiltration Sensible Heat Gain Energy` | J | Energía sensible que entra (puede ser negativa) |
| `Zone Infiltration Total Heat Gain Energy` | J | Sensible + latente |

### Verificación de la prueba de fuego

> "Lo primero que me dice si está bien implementado es si está en el RDD."

`Show Simulation → RDD` → buscar `Zone Infiltration`. Si las variables aparecen ahí, el objeto está implementado correctamente. Si no aparecen, el Space Type **no** está asignado a ningún Space (Paso 6) o el objeto se perdió.

### Análisis en Python

```python
from iertools.read import read_sql
import matplotlib.pyplot as plt
from dateutil.parser import parse
import pandas as pd

base = read_sql("../osm/008_ach/run/eplusout.sql", alias=True).data

fig, ax = plt.subplots(figsize=(12, 3))
f1 = parse("2006-01-01")
f2 = f1 + pd.Timedelta(days=2)

ach_col = [c for c in base.columns if "Air Change Rate" in c][0]
ax.plot(base[ach_col], label="ACH")
ax.set_xlim(f1, f2)
ax.set_ylabel("ACH (1/h)")
ax.legend()
```

Si todo está bien, la curva es **casi una recta** al valor configurado.

## Anti-patrones a evitar

| Anti-patrón | Consecuencia |
|---|---|
| **Schedule con valores > 1** | E+ falla con error de validación. Recordar: fraccional = 0-1. |
| **Coeficientes [A,B,C,D] todos en 0** | El flujo resultante es 0 — ninguna infiltración aunque el ACH de diseño sea 0.5. |
| **Coeficientes [A,B,C,D] = [0, 0.0001, 0, 0]** | Flujo dependiente solo de ΔT — "físicamente más real" pero descalibra comparaciones con el caso base. |
| **No asignar Space Type a los Spaces** | El objeto existe en el modelo pero **no se activa**. La variable no aparece en el RDD. |
| **Copiar el OSM en el Finder** | Se pierde el folder hermano con measures asociados. Usar `File → Save As`. Ver [[../concepts/Caso-Base#cómo-no-copiar-el-caso-base]]. |

## Aplicación al proyecto final 2026-2

El [[../concepts/Caso-Base|caso base]] del proyecto incluye **0.5 ACH constante** (decisión tomada en clase 014 tras consulta con Miriam).

Como **estrategia bioclimática**:

- **Ventilación nocturna** (pendiente confirmación de Miriam): schedule con valor 1 entre 20:00-6:00, 0 resto del día; ACH de diseño = 1 → resulta en 1 ACH solo de noche.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — demostración en vivo + bug confesional del profesor

## Ver también

- [[../concepts/Infiltracion-Cambios-Aire]] — concepto y ecuación enriquecida de E+
- [[../concepts/Schedules]] — schedules fraccionales
- [[../concepts/Caso-Base]] — caso base del proyecto final 2026-2
- [[Debuggear-Simulacion-OpenStudio]] — qué hacer cuando la simulación falla con error de validación

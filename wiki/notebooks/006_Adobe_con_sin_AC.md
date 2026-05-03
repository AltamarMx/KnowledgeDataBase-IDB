---
title: 006 — Adobe con y sin AC en EnerHabitat
type: notebook
notebook: 006_Adobe_con_sin_AC
fuente: raw/notebooks/006_Adobe_con_sin_AC.ipynb
fecha_ingesta: 2026-05-02
tags: [notebook, enerhabitat, adobe, paramétrico, absortancia, ac, campeche]
aliases: [006 EnerHabitat, libreta adobe AC, paramétrico EnerHabitat]
clase_relacionada: 011
---

# 006 — Adobe con y sin AC en EnerHabitat

## Metadatos

- **Notebook:** `006_Adobe_con_sin_AC.ipynb`
- **Fuente:** `raw/notebooks/006_Adobe_con_sin_AC.ipynb`
- **Clase relacionada:** [[../classes/011-EnerHabitatParte2]]
- **Objetivo:** Demostrar el flujo paramétrico de [[../tools/EnerHabitat|EnerHabitat]] con la **API real verificada** (versión 0.1.9). Resuelve un muro de adobe con/sin AC e itera sobre la absortancia.

## Resumen — la libreta que **sí** funciona

Es la libreta que el profesor intentó hacer en vivo en la clase 011 y que falló por el bug de pandas 3.0. Ya con el fix aplicado en `enerhabitat 0.1.9` el flujo corre limpio.

**Importante**: esta libreta **revela la API real** del paquete, que difiere de lo documentado en clases anteriores por errores de transcripción y memoria del profesor:

| Lo que se transcribió en clase | Lo que realmente es |
|--------------------------------|----------------------|
| `Wall` | **`System`** |
| `absorptance` | **`absortance`** (sin "p" — calco del español) |
| `set_day(month=5)` | **`location.meanDay(month=5, year=Y)`** |
| `tsa()` | **`Tsa()`** (T mayúscula) |
| `solve_ac()` | **`solveAC()`** (camelCase) |
| `T_int`, `To` | **`Ti`, `Ta`** |

Las correcciones se reflejan en [[../tools/EnerHabitat]] y [[../procedures/Usar-EnerHabitat-Python]].

## Imports

```python
import enerhabitat as eh
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
```

> **Nota**: el alias usado por convención del taller es `eh`. No tiene parámetros adicionales en la API.

## Flujo verificado — caso sin AC

```python
epw_file = "../epw/MEX_CAM_Campeche-Ignacio.766961_TMYx.epw"

wall = eh.System(eh.Location(epw_file))
wall.azimuth    = 90
wall.tilt       = 90
wall.absortance = 1
wall.layers     = [("adobe", 0.30)]
wall.location.meanDay(month=5, year=2026)
wall.Tsa()

sinac = wall.solve()
sinac = pd.concat([sinac, wall.Tsa().asfreq("10min")], axis=1)
```

### Paso a paso

1. **Crear el sistema**: `eh.System(eh.Location(epw_file))` — envuelve la geolocalización en el sistema constructivo.
2. **Configurar geometría**: `azimuth`, `tilt`, `absortance`. Convención de azimut: 0=Norte, 90=Este, 180=Sur, 270=Oeste.
3. **Definir capas**: lista de tuplas `(material_str, espesor_m)`. Material debe estar en el `materials.ini` del subdirectorio.
4. **Día representativo**: `wall.location.meanDay(month, year)`. Calcula el día promedio del mes en el EPW del location.
5. **Calcular Tsa**: `wall.Tsa()`. **Obligatorio antes de `solve()`** — es el forzamiento exterior.
6. **Resolver**: `wall.solve()` para caso sin AC. Devuelve un DataFrame de 144 filas (24h × 6 timesteps por hora).

### Por qué hay que concatenar Tsa al output

```python
sinac = pd.concat([sinac, wall.Tsa().asfreq("10min")], axis=1)
```

El DataFrame que retorna `solve()` **no incluye Tsa** por default. Hay que concatenarla manualmente. Comentario del notebook explica:

> "Tsa is a function of color, tilt, orientation, month and location, so it must be recomputed whenever any of those inputs change."

`asfreq("10min")` alinea la frecuencia (Tsa puede venir a otra frecuencia interna).

## Estructura del DataFrame retornado

```
[144 rows x 13 columns]
```

13 columnas — la mayoría son **datos derivados del EPW + cálculo solar**:

| Columna | Significado | Unidad |
|---------|-------------|--------|
| `Ti` | T interior (la solución del cálculo) | °C |
| `zenith` | Ángulo cenital del sol | grados |
| `elevation` | Altura solar (negativa de noche) | grados |
| `azimuth` | Acimut solar | grados desde norte |
| `equation_of_time` | Ecuación del tiempo | min |
| `Ta` | T ambiente del aire exterior (del EPW) | °C |
| `Ig` | Radiación global horizontal | W/m² |
| `Ib` | Radiación directa normal | W/m² |
| `Id` | Radiación difusa horizontal | W/m² |
| `Tn` | T de neutralidad (Humphreys-Nicol mensual) | °C |
| `DeltaTn` | Banda de confort (delta) | °C |
| `Is` | Radiación incidente sobre la superficie | W/m² |
| `Tsa` | T sol-aire | °C |

### Diferencias críticas vs `iertools`

| `iertools.read_sql` | `enerhabitat` |
|----------------------|---------------|
| `To` | **`Ta`** |
| `Ti_<zona>` | **`Ti`** (single — un solo wall) |
| Series anuales | **Series de 1 día** (144 filas) |

### `DeltaTn = 1.25 °C` — modelo de Morillón

```
Tn       29.397
DeltaTn   1.25
```

El `DeltaTn` reportado **no es** el ±3.5 °C de [[../concepts/Confort-Adaptativo|Humphreys-Nicol/ASHRAE 55]]. Es el **delta de Morillón** — modelo adaptativo mexicano más estrecho.

> Mencionado por el profesor: "vamos a usar la temperatura de neutralidad más el intervalo, la amplitud que propuso un Morillón. Es una amplitud que varía dependiendo de la amplitud de la oscilación de cada sitio."

Para ciudades cálidas con poca oscilación diaria (como Campeche), el delta es chico (~1.25 °C). Para climas secos con alta oscilación, el delta es mayor.

## Caso con AC — `solveAC()`

```python
wall = eh.System(eh.Location(epw_file))
wall.azimuth    = 90
wall.tilt       = 90
wall.absortance = 1
wall.layers     = [("adobe", 0.30)]
wall.location.meanDay(month=5, year=2026)
wall.Tsa()

conac = wall.solveAC()
conac = pd.concat([conac, wall.Tsa().asfreq("10min")], axis=1)

print(wall.cooling_energy, wall.heating_energy)
# Output: 210255.91356566016 0.0
```

### Setpoint usado

`solveAC()` aplica el setpoint en el **límite superior de confort**: `Tn + DeltaTn`. Para Campeche en mayo:

- `Tn` = 29.4 °C
- `DeltaTn` = 1.25 °C
- **Setpoint cooling** = 30.65 °C

Esto significa que el AC solo enfría cuando la T interior **sale del rango adaptativo**. En climas cálidos como Campeche, esto sigue siendo bastante de tiempo.

### Atributos de energía

Después de `solveAC()`:

| Atributo | Significado | Unidad |
|----------|-------------|--------|
| `wall.cooling_energy` | Energía total de enfriamiento del día representativo | J/(m²·día) |
| `wall.heating_energy` | Energía total de calefacción | J/(m²·día) |

Para Campeche en mayo con adobe negro (α=1):

- `cooling_energy` = **210,255 J/(m²·día)**
- `heating_energy` = **0** — Campeche nunca llega al límite inferior de confort (clima cálido)

## Estudio paramétrico — el correcto

```python
calentamiento = []
enfriamiento  = []
absortancia   = []

for absortance in np.linspace(0.01, 1, 10):
    wall = eh.System(eh.Location(epw_file))
    wall.azimuth    = 90
    wall.tilt       = 90
    wall.absortance = absortance         # ← varía la absortancia (no el espesor)
    wall.layers     = [("adobe", 0.3)]   # ← espesor fijo
    wall.location.meanDay(month=5, year=2026)
    wall.Tsa()

    conac = wall.solveAC()
    conac = pd.concat([conac, wall.Tsa().asfreq("10min")], axis=1)

    absortancia.append(absortance)
    enfriamiento.append(wall.cooling_energy)
    calentamiento.append(wall.heating_energy)
```

### Diferencias con la versión que falló en la clase 011

En clase 011 el profesor en vivo escribió por error:

```python
wall.layers = [("adobe", absortance)]   # ❌ alpha es ESPESOR aquí, no absortancia
```

→ Variaba el espesor del adobe, no su color → resultados contraintuitivos.

La versión correcta del notebook 006 separa correctamente:

```python
wall.absortance = absortance      # ← color, varía
wall.layers     = [("adobe", 0.3)]  # ← espesor fijo
```

> **Lección de la clase 011**: confiar en la física. Si el resultado paramétrico es contraintuitivo, revisar el código primero.

### Crear nuevo `wall` en cada iteración

Cada paso del loop crea **un nuevo `wall`**:

```python
wall = eh.System(eh.Location(epw_file))
```

Razón: el solver mantiene estado interno. Reusar el mismo `wall` cambiando solo `absortance` puede dejar valores residuales del cálculo anterior. La estrategia segura es **crear desde cero** cada iteración. Costo: ~1 segundo extra de instanciación por iteración, despreciable.

## Plot del paramétrico

```python
fig, ax = plt.subplots()
ax.scatter(absortancia, enfriamiento, label="enfriamiento")
ax.scatter(absortancia, calentamiento, label="calentamiento")
ax.legend()
```

### Por qué `scatter` y no `plot`

- **`scatter`** marca cada punto sin conectarlos con línea — útil cuando los puntos son discretos (10 valores de absortancia, no una serie continua).
- **`plot`** los conectaría con línea — sugiere continuidad. Visualmente similar pero con interpretación distinta.

Para 10 puntos `scatter` es suficiente. Para 100 puntos `plot` se ve mejor.

### Resultado esperado en Campeche

- **Enfriamiento ↑** con α (más absorción solar → más calor → más AC).
- **Calentamiento = 0** todo el rango de α (Campeche es muy cálido).

Patrón coherente con la lección clave del bioclimático: **colores claros** reducen drásticamente la carga de cooling en climas cálidos. Si α va de 0.01 (blanco perfecto) a 1.0 (negro), el cooling escala casi lineal.

## EPW usado — Campeche

```
"../epw/MEX_CAM_Campeche-Ignacio.766961_TMYx.epw"
```

- **`MEX_CAM`** = México, Campeche.
- **`Campeche-Ignacio`** = aeropuerto Ignacio Alfredo Bárrenos.
- **`TMYx`** = TMY de cualquier rango (no especifica años).

Clima de Campeche:

- Cálido húmedo (península de Yucatán).
- Pico térmico en abril-mayo (pre-monzón).
- Sin invierno frío — ideal para diseño bioclimático sin calefacción.
- **Solo extremo cálido** — el caso ideal según el consejo del profesor en clase 002.

## Patrones para reusar

1. **Recrear `wall` en cada iteración** del loop paramétrico — evita estado residual.
2. **`absortance` para color, `layers` para espesor** — no confundirlos.
3. **`Tsa()` antes de `solve()`** siempre.
4. **`pd.concat` con `asfreq("10min")`** para tener `Tsa` en el output.
5. **`scatter` para puntos discretos**, `plot` para series continuas.
6. **`np.linspace(0.01, 1, 10)`** para evitar α=0 (irreal — sería un material que ni absorbe ni refleja).

## Limitaciones observadas

- **Heating energy = 0** en clima cálido — la métrica no agrega información cuando el clima es de un solo extremo.
- **Solo 10 puntos de α** — para análisis fino conviene `np.linspace(0.01, 1, 50)` o más.
- **Solo un mes (mayo)** — para el año completo hay que iterar también sobre meses, lo cual triplica el tiempo de cómputo.
- **Sin gráfica de la temperatura sol-aire** — el día solo se reporta como `cooling_energy` agregado.

## Clase relacionada

- [[../classes/011-EnerHabitatParte2]] — clase donde el flujo se intentó en vivo y falló por el bug pandas 3.0
- [[../classes/010-EnerHabitatParte1]] — introducción a la herramienta
- [[../classes/009-AireAcondicionadoSetPoints]] — concepto de setpoint en el límite de confort

## Herramienta

- [[../tools/EnerHabitat]] — paquete (con la API actualizada de este notebook)

## Procedimiento

- [[../procedures/Usar-EnerHabitat-Python]] — flujo formal (actualizado con la API real)

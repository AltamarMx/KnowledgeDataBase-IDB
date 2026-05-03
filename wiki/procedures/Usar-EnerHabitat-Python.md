---
title: Usar EnerHabitat (paquete Python)
type: procedimiento
tags: [procedimiento, enerhabitat, python, paquete, primeras-decisiones, scripting]
aliases: [enerhabitat python, paquete enerhabitat, pip install enerhabitat]
clases: [010, 011]
updated: 2026-05-02
---

# Usar EnerHabitat (paquete Python)

Procedimiento para usar el paquete Python de [[../tools/EnerHabitat]] — vía recomendada cuando se necesita más control que la web app: orientaciones no cardinales, estudios paramétricos, integración con notebooks.

> **Versión 0.1.9 (clase 011):** flujo verificado. Bug previo (`assignment is read only` por pandas 3.0 que hizo inmutables los resultados) corregido. La documentación se actualizó tras el fix.

## Pre-requisitos

- Entorno Python con uv listo. Ver [[Setup-Entorno-Python-uv]].
- Para el ejemplo: un EPW descargado y un archivo `materials.ini` (de la base local de materiales).

## 1. Crear proyecto y agregar dependencias

```bash
uv init test_enerhabitat
cd test_enerhabitat

uv add enerhabitat pandas matplotlib jupyter
```

`enerhabitat` se instala desde **PyPI**. Trae como dependencia transitiva `pvlib` (usado para proyección de radiación solar).

> Recordatorio: cualquiera puede publicar en PyPI sin revisión. EnerHabitat es publicado por el grupo IER. La cadena de confianza es relevante — verificar que el paquete viene del grupo correcto.

## 2. Conseguir el archivo `materials.ini`

EnerHabitat necesita una base de datos local de materiales. La que usa la web app:

1. Ir al repo GitHub `enerhabitat/enerhabitat` o `enerhabitat/web-app`.
2. Descargar `materials.ini` (en la raíz o subfolder de configuración).
3. Colocarlo en la raíz del proyecto Python.

Estructura del archivo (texto plano, formato INI):

```ini
[tabique_recocido]
conductivity = 0.7
density = 1400
specific_heat = 1000

[concreto_alta_densidad]
conductivity = 2.0
density = 2400
specific_heat = 1000

[EPS]
conductivity = 0.035
density = 45
specific_heat = 1500

# ... más materiales
```

Para **agregar materiales propios**: editar el INI y agregar una sección con propiedades térmicas. El cambio aplica solo a la copia local — no afecta la web pública.

## 3. Conseguir el EPW

Descargar de OneBuilding o usar uno propio. Detalle en [[Descargar-EPW-OneBuilding]]. Colocar en un subfolder `EPW/` del proyecto.

## 4. Estructura básica del código (API verificada en 0.1.9)

> **Importante**: la API real, verificada con el [[../notebooks/006_Adobe_con_sin_AC|notebook 006]], difiere de lo que se transcribió en clase. Las correcciones se reflejan abajo.

```python
import enerhabitat as eh
import pandas as pd
import matplotlib.pyplot as plt

# 1. Crear el sistema (envuelve location + EPW)
wall = eh.System(eh.Location("EPW/MX_Cuernavaca.epw"))

# 2. Configurar la geometría
wall.azimuth    = 90      # Norte=0, Este=90, Sur=180, Oeste=270
wall.tilt       = 90      # 90 = muro vertical, 180 = techo horizontal
wall.absortance = 0.3     # ← "absortance" SIN p (calco del español "absortancia")

# 3. Definir las capas (ext → int)
wall.layers = [
    ("adobe", 0.30),       # tupla (nombre del material, espesor en m)
]

# 4. Día representativo del mes — método de location, no de wall
wall.location.meanDay(month=4, year=2026)

# 5. Calcular la temperatura sol-aire (Tsa con T MAYÚSCULA)
wall.Tsa()                 # se almacena en wall internamente

# 6. Resolver la transferencia de calor
result = wall.solve()      # sin AC — la T flota
# o
result = wall.solveAC()    # con AC (camelCase) — setpoint adaptativo
```

### Diferencias críticas con la transcripción de clase

| Lo transcrito en clase | API real verificada |
|-------------------------|---------------------|
| `enerhabitat.Wall(...)` | **`eh.System(eh.Location(epw_file))`** |
| `wall.absorptance = α` | **`wall.absortance = α`** (sin p) |
| `wall.set_day(month=4)` | **`wall.location.meanDay(month=4, year=Y)`** |
| `wall.tsa()` | **`wall.Tsa()`** (T mayúscula) |
| `wall.solve_ac()` | **`wall.solveAC()`** (camelCase) |
| Output con `T_int` | **`Ti`** (interior) |
| Output con `To` | **`Ta`** (ambiente) |

Detalle del descubrimiento en [[../notebooks/006_Adobe_con_sin_AC]].

### `materials.ini` auto-detectado

EnerHabitat busca `materials.ini` en el **subdirectorio donde corre la libreta** automáticamente. Si está ahí, no hay que pasar la ruta explícitamente. Si no se encuentra, hay que pasar la ruta como argumento.

> Detalle no documentado oficialmente — descubierto en clase 011. Conviene poner `materials.ini` en la raíz del proyecto Python para que se auto-detecte.

### `solve()` vs `solveAC()`

| Método | Resultado | Variables clave que se generan |
|--------|-----------|---------------------------------|
| `wall.solve()` | T interior flota libre | DataFrame con `Ti` (serie temporal del día) |
| `wall.solveAC()` | T interior constante en setpoint adaptativo | DataFrame con `Ti` constante; atributos `wall.cooling_energy`, `wall.heating_energy` (J/(m²·día)) |

Para climas cálidos: el setpoint adaptativo se coloca en el **límite superior de confort**. Para climas fríos: el setpoint actual no es óptimo (issue documentado en GitHub).

### Concatenar Tsa al output del solve

El DataFrame que retorna `solve()` o `solveAC()` **no incluye Tsa** por default. Hay que concatenarla manualmente:

```python
result = pd.concat([result, wall.Tsa().asfreq("10min")], axis=1)
```

> Comentario del notebook 006 lo explica: "Tsa is a function of color, tilt, orientation, month and location, so it must be recomputed whenever any of those inputs change."

### Columnas del DataFrame de output

13 columnas para un día (144 filas a 10 min):

| Columna | Significado |
|---------|-------------|
| `Ti` | T interior (la solución del cálculo) |
| `Ta` | T ambiente exterior (del EPW) |
| `Tsa` | T sol-aire (al concatenar con `pd.concat`) |
| `Tn` | T de neutralidad (Humphreys-Nicol mensual) |
| `DeltaTn` | Banda de confort — **modelo de Morillón** (ej. 1.25 °C en Campeche), no Humphreys 3.5 |
| `Is` | Radiación incidente sobre la superficie |
| `Ig`, `Ib`, `Id` | Radiación global, directa, difusa del EPW |
| `zenith`, `elevation`, `azimuth` | Posición solar |
| `equation_of_time` | Ecuación del tiempo |

## 4b. Configuración global de coeficientes convectivos

```python
enerhabitat.config.h0 = 13   # h_c exterior (W/m²K) — default 13
enerhabitat.config.h1 = 8.6  # h_c interior — default 8.6
```

> **Bug observado en clase 011**: cambios a `config.h0` **después** de llamar `tsa()` no aplican retroactivamente. Configurar **antes** de llamar `tsa()` y `solve()`.

> **"Susto feliz" del primer time step**: si miras solo el primer valor del día, los cambios a $\alpha$ o $h_c$ **no afectan** $T_{sa}$ porque $I = 0$ a las 00:00. Mirar el día completo, especialmente horas con sol. Detalle en [[../concepts/Temperatura-Sol-Aire]].

## 5. Comparar dos sistemas constructivos

```python
import enerhabitat as eh

epw_file = "EPW/MX_Cuernavaca.epw"

# Sistema 1 — tabique blanco
wall_1 = eh.System(eh.Location(epw_file))
wall_1.azimuth    = 90
wall_1.tilt       = 90
wall_1.absortance = 0.3                       # blanco
wall_1.layers     = [("tabique_recocido", 0.14)]
wall_1.location.meanDay(month=4, year=2026)
wall_1.Tsa()
res_1 = wall_1.solve()
res_1 = pd.concat([res_1, wall_1.Tsa().asfreq("10min")], axis=1)

# Sistema 2 — tabique rojo
wall_2 = eh.System(eh.Location(epw_file))
wall_2.azimuth    = 90
wall_2.tilt       = 90
wall_2.absortance = 0.7                       # rojo
wall_2.layers     = [("tabique_recocido", 0.14)]
wall_2.location.meanDay(month=4, year=2026)
wall_2.Tsa()
res_2 = wall_2.solve()
res_2 = pd.concat([res_2, wall_2.Tsa().asfreq("10min")], axis=1)

# Comparar — usar Ti (no T_int) y Tsa (no T_sa)
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(res_1["Ti"],  label="blanco (α=0.3)", color="blue")
ax.plot(res_2["Ti"],  label="rojo (α=0.7)",   color="red")
ax.plot(res_1["Tsa"], label="Tsa blanco",     color="blue", linestyle="--")
ax.plot(res_2["Tsa"], label="Tsa rojo",       color="red",  linestyle="--")
ax.legend()
ax.set_ylabel("T (°C)")
```

Resultado esperado: la `Tsa` del rojo es **mucho mayor** (más absorción solar). La T interior del rojo también es mayor, pero la diferencia se atenúa por la masa del tabique.

## 6. Estudios paramétricos — caso de uso central

> "Esto sí es lo que no puedo hacer en la web app. Aquí parametrizo y resuelvo todo de una."

Patrón básico: iterar sobre absortancia comparando consumo AC.

```python
import numpy as np
import enerhabitat as eh

epw_file = "EPW/cuernavaca.epw"
absortancias = np.linspace(0.01, 1.0, 10)   # 10 puntos
consumo_cool = []
consumo_heat = []

for alpha in absortancias:
    # Crear nuevo wall en cada iteración (evita estado residual)
    wall = eh.System(eh.Location(epw_file))
    wall.azimuth    = 90
    wall.tilt       = 90
    wall.absortance = alpha                  # ← variar absortance (no espesor)
    wall.layers     = [("concreto", 0.15)]   # espesor fijo
    wall.location.meanDay(month=4, year=2026)
    wall.Tsa()
    wall.solveAC()

    # cooling_energy y heating_energy son atributos escalares (J/(m²·día))
    consumo_cool.append(wall.cooling_energy)
    consumo_heat.append(wall.heating_energy)

fig, ax = plt.subplots()
ax.scatter(absortancias, consumo_cool, label="enfriamiento")
ax.scatter(absortancias, consumo_heat, label="calentamiento")
ax.set_xlabel("Absortancia α")
ax.set_ylabel("Energía (J/(m²·día))")
ax.legend()
```

> **Nota**: `wall.cooling_energy` y `wall.heating_energy` son **escalares**, no series temporales. No usar `.sum()` — ya están agregados al día representativo.

Resultado esperado en clima cálido (Cuernavaca):

- **Enfriamiento ↑** con α (más absorción → más calor → más AC).
- **Calentamiento ↓** con α (la absorción reduce necesidad de calefacción).

### Anti-patrón observado en clase 011

> "Todo lo que les dije está mal. Ahí no va la absortancia, aquí va el espesor."

```python
# ❌ MAL — alpha aquí es ESPESOR, no absortancia
wall.layers = [("adobe", alpha)]

# ✅ BIEN
wall.absorptance = alpha
wall.layers      = [("adobe", 0.30)]   # espesor fijo
```

Lección: si los resultados son contraintuitivos, **revisar el código**. La física no miente.

### Anti-patrón observado: pegar T sol-aire entre walls distintos

> "La temperatura sol-aire le pertenece a ese muro y solo a ese muro."

Si cambias color, orientación, lugar o período, la T sol-aire cambia. Solo se puede compartir entre walls que tienen **idénticos** todos esos parámetros (solo difieren en sistema constructivo). Detalle en [[../concepts/Temperatura-Sol-Aire]].

### Anti-patrón observado: confiar en resultados raros

> "Si yo no tengo una idea de la física, me puedo sobreexplicar resultados raros para justificarlos. Es bien peligroso."

Si un resultado contradice la predicción cualitativa de la física:

1. Predecir cualitativamente **antes** de correr.
2. Si difiere, **revisar el código** primero (no la física).
3. Solo en último caso cuestionar la física.

Aplicable a cualquier análisis numérico — no solo EnerHabitat.

## 7. Programación orientada a objetos — versatilidad

EnerHabitat usa OOP — varios `Wall` pueden compartir un `Location`, o cada uno puede tener su propia `Location`:

```python
# Mismo clima, distintas orientaciones
loc = enerhabitat.Location(epw_file="EPW/temixco.epw")

walls = {
    orient: enerhabitat.Wall(loc) for orient in ["N", "S", "E", "W"]
}

azimuts = {"N": 0, "E": 90, "S": 180, "W": 270}
for o, w in walls.items():
    w.azimuth = azimuts[o]
    w.tilt    = 90
    # ... mismas capas y absortancia
    w.set_day(month=4)
    w.tsa()

# Comparar las 4 orientaciones
```

Ventaja: estructura limpia, código claro, fácil de extender (agregar inclinaciones intermedias, etc.).

## 8. Variables de salida del `solve()`

(Aproximado — confirmar con la documentación actual del paquete.) Típicamente:

| Variable | Significado |
|----------|-------------|
| `T_int` | T del aire interior, serie temporal del día |
| `T_sa` | T sol-aire, serie temporal del día |
| `T_amb` | T ambiente del aire exterior |
| `IS` | Radiación incidente sobre la superficie |
| `FD` | Factor de decremento ingenuo |
| `FD_sa` | Factor de decremento sol-aire |
| `time_lag` | Tiempo de retraso (h) |
| `q_in`, `q_out` | Energía entrante/saliente por ciclo |
| `T_neut`, `T_sup`, `T_inf` | Banda de confort adaptativo |

## 9. Validar con el repo `validation`

Para confirmar que el flujo funciona: ver el repo `enerhabitat/validation` en GitHub. Tiene notebooks de ejemplo que **comparan EnerHabitat vs Energy Plus** — replicar las mismas condiciones en E+ y verificar que los resultados coinciden (dentro del margen aceptable).

> "Hicimos la simulación con E+ y con EnerHabitat. Deberían dar lo mismo. Es una talachita pero no es imposible."

Es el patrón de validación de software: dos métodos independientes que convergen al mismo resultado.

## Trampas comunes

| Síntoma | Causa probable |
|---------|----------------|
| `Material X not found` | El nombre en `layers` no coincide con la sección en `materials.ini`. Verificar typo (mayúsculas/minúsculas) |
| `assignment is read only` | Versión del paquete < 0.1.9. Actualizar (`uv add enerhabitat --upgrade`) |
| EPW no carga | Path incorrecto, archivo corrupto, lat/lon fuera de rango |
| Resultados implausibles | Capas en orden incorrecto (interior↔exterior); revisar. O usaste `wall.layers` cuando querías cambiar `wall.absorptance` |
| FD sol-aire > 1 | Algo está mal — el límite termodinámico se rompió. Verificar absortancia y geometría |
| Cambio en `α` o `h_c` no se ve | Mirando el primer time step (radiación = 0). Mirar el día completo |
| `config.h0` no aplica | Configurado **después** de `tsa()` — configurar antes |

## Cuándo NO usar este paquete

Si solo necesitas un análisis rápido de 1-2 sistemas constructivos: la **web app** es más rápida (sin setup). Ver [[Usar-EnerHabitat-Web]].

Si necesitas:

- **Análisis anual** (no de un día representativo).
- **Edificación completa** con varias zonas.
- **Cálculos energéticos serios** para reportar consumo.

→ usar **Energy Plus**, no EnerHabitat. Ver [[../tools/EnergyPlus]] y [[Crear-Primera-Simulacion-OpenStudio]].

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción al paquete (con bug en la demo)
- [[../classes/011-EnerHabitatParte2]] — API verificada en 0.1.9, patrón paramétrico, anti-patrones observados

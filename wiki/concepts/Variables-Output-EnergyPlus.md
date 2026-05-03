---
title: Variables de Output de Energy Plus
type: concepto
tags: [concepto, energyplus, output, variables, postprocesamiento]
aliases: [output variables, variables de salida, output:variable]
clases: [005, 007, 008, 009]
updated: 2026-05-02
---

# Variables de Output de Energy Plus

## Familias de variables

E+ organiza las variables de salida por el **objeto** al que se asocian. El primer término del nombre indica la familia:

| Prefijo | Aplica a | Ejemplos |
|---------|----------|----------|
| **Site** | El sitio / clima (input del EPW) | T exterior, radiación, viento |
| **Zone** | Una zona térmica (volumen de aire) | T media del aire, T operativa, ganancias |
| **Surface Outside Face** | Cara exterior de una superficie | T superficial, radiación incidente, h_c |
| **Surface Inside Face** | Cara interior de una superficie | T superficial interior, ganancias por radiación |
| **Surface** (sin sufijo) | La superficie como un todo | Construction, área |
| **Output** | Outputs agregados de la simulación | Run periods, errores |

## Variables `Site:*` (clima del EPW)

Todas tienen frecuencia mínima horaria (el EPW es horario).

| Variable | Significado | Unidades |
|----------|-------------|----------|
| `Site Outdoor Air Drybulb Temperature` | T de bulbo seco exterior | °C |
| `Site Outdoor Air Dewpoint Temperature` | T de rocío | °C |
| `Site Outdoor Air Wetbulb Temperature` | T de bulbo húmedo | °C |
| `Site Outdoor Air Relative Humidity` | Humedad relativa | % |
| `Site Wind Speed` | Velocidad del viento (a 10 m, ref. estación) | m/s |
| `Site Wind Direction` | Dirección del viento | grados desde el norte |
| `Site Direct Solar Radiation Rate per Area` | Radiación solar **directa** (en plano normal al sol) | W/m² |
| `Site Diffuse Solar Radiation Rate per Area` | Radiación solar **difusa** (sobre plano horizontal) | W/m² |
| `Site Solar Altitude Angle` | Altura solar | grados |
| `Site Solar Azimuth Angle` | Acimut solar | grados desde el norte |

> **Importante: no hay variable "global solar radiation"**. La global = directa proyectada al horizontal + difusa. Para tenerla en una superficie horizontal: pedir `Surface Outside Face Incident Solar Radiation Rate per Area` sobre el techo (siempre que no tenga sombreamiento). Truco mencionado por el profesor.

### Site vs Outdoor — capa límite atmosférica

Algunas variables tienen versión `Site:*` (a 10 m, valor "crudo" del EPW) y `Outdoor:*` ajustado a la altura del centroide de la zona térmica. E+ ajusta porque la T del aire varía con la altura. Detalle en [[Capa-Limite-Atmosferica]].

## Variables `Zone:*` (zona térmica)

| Variable | Significado | Unidades |
|----------|-------------|----------|
| `Zone Mean Air Temperature` | T promedio del aire de la zona ([[Mezclado-Perfecto]]) | °C |
| `Zone Air Temperature` | Casi idéntica a Mean Air Temp; solo difiere en simulaciones con modelos de aire detallado | °C |
| `Zone Operative Temperature` | [[Temperatura-Operativa]] (promedio T_aire + T_radiante) | °C |
| `Zone Mean Radiant Temperature` | T radiante de la zona | °C |
| `Zone Air Relative Humidity` | HR del aire interior | % |
| `Zone Total Internal Radiant Heating Rate` | Suma de ganancias radiantes de fuentes internas (proyectores, luminarias) | W |
| `Zone Windows Total Heat Gain Rate` | Calor neto que **entra a la zona** por todas las ventanas (después de transmitancia/reflectancia/absortancia) | W |
| `Zone Air System Sensible Cooling Rate` | Carga de enfriamiento (con HVAC) | W |
| `Zone Ideal Loads Zone Total Cooling Energy` | Energía total de enfriamiento (sensible + latente) — para integral mensual con `resample().sum()` | J |
| `Zone Ideal Loads Zone Total Heating Energy` | Análoga para calefacción | J |
| `Zone Ideal Loads Zone Sensible Cooling Rate` | Potencia de enfriamiento sensible | W |
| `Zone Ideal Loads Zone Sensible Heating Rate` | Potencia de calefacción sensible | W |
| `Zone Thermostat Cooling Setpoint Temperature` | Setpoint de cooling aplicado en cada paso (verifica que el schedule llegó al IDF) | °C |
| `Zone Thermostat Heating Setpoint Temperature` | Análogo | °C |

## Variables `Surface Outside Face:*` (cara exterior)

| Variable | Significado | Unidades |
|----------|-------------|----------|
| `Surface Outside Face Temperature` | T de la cara exterior | °C |
| `Surface Outside Face Incident Solar Radiation Rate per Area` | Radiación solar **incidente** sobre la cara (dir. proyectada + difusa) | W/m² |
| `Surface Outside Face Incident Beam Solar Radiation Rate per Area` | Solo la directa proyectada | W/m² |
| `Surface Outside Face Incident Sky Diffuse Solar Radiation Rate per Area` | Difusa que viene del cielo | W/m² |
| `Surface Outside Face Incident Ground Diffuse Solar Radiation Rate per Area` | Difusa **reflejada por el piso** | W/m² |
| `Surface Outside Face Absorbed Shortwave Radiation Rate per Area` | Lo que se absorbe (incidente × α) | W/m² |
| `Surface Outside Face Convection Heat Transfer Coefficient` | h_c exterior | W/m²K |
| `Surface Outside Face Convection Heat Gain Rate per Area` | Flujo convectivo exterior | W/m² |
| `Surface Outside Face Net Thermal Radiation Heat Gain Rate per Area` | Balance LWR neto exterior | W/m² |
| `Surface Outside Face Sunlit Fraction` | Fracción de la superficie con sol directo (0-1) — captura sombreamiento sobre la radiación directa | adimensional |
| `Surface Outside Face Sunlit Area` | Área absoluta soleada | m² |

## Variables `Surface Inside Face:*` (cara interior)

| Variable | Significado | Unidades |
|----------|-------------|----------|
| `Surface Inside Face Temperature` | T de la cara interior | °C |
| `Surface Inside Face Convection Heat Transfer Coefficient` | h_c interior | W/m²K |
| `Surface Inside Face Convection Heat Gain Rate per Area` | Flujo convectivo interior | W/m² |
| `Surface Inside Face Net Surface Thermal Radiation Heat Gain Rate per Area` | LWR interior neto entre superficies del cuarto | W/m² |
| `Surface Inside Face Solar Radiation Heat Gain Rate per Area` | Onda corta que llega al interior (de ventanas + luminarias, distribuida — ver [[Radiacion-Interior-Distribuida]]) | W/m² |

## Convención de nombres en el output (CSV / SQL)

Cuando se reporta una variable, las columnas combinan **objeto + variable + unidades**:

```
NORTE:Zone Mean Air Temperature [C]
TECHO:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]
Environment:Site Outdoor Air Drybulb Temperature [C]
```

El **objeto** queda en mayúsculas antes de los dos puntos. Por eso conviene poner nombres en mayúsculas y sin acentos a zonas y superficies (ver [[../concepts/Espacio-vs-ZonaTermica]]).

## ⚠️ Frecuencias mezcladas — antipatrón

Cuando se solicitan output variables con **frecuencias distintas** (ej. una a `Timestep` y otra a `Hourly`), `iertools.read_sql` alinea todo al índice de mayor resolución. Las celdas donde la variable de menor resolución no tiene dato → **NaN**.

Síntoma observado en el [[../notebooks/003_EDA|notebook 003]]:

```
                     ESTE:Zone Air Temperature (C)    Ti_ESTE
2006-01-01 00:10:00                            NaN  24.692687
...
```

**Casi todos los valores de la columna en NaN** salvo el último de cada hora. Causa: esa variable se solicitó a `Hourly` mientras `Ti_ESTE` se solicitó a `Timestep`.

**Buena práctica**: configurar **todas las output variables con la misma frecuencia** (típicamente `Timestep`). Si una se quiere con menor resolución, hacer `df.col.resample("h").mean()` después de cargar. Ver [[../procedures/Solicitar-Output-Variables-Measures]].

## `Zone Mean Air Temperature` vs `Zone Air Temperature` — qué pasa con el alias

Las dos variables tienen significado **muy similar** pero `iertools` solo aplica alias automático a `Zone Mean Air Temperature`:

| Variable | Alias |
|----------|-------|
| `Zone Mean Air Temperature` | `Ti_<zona>` (alias automático) |
| `Zone Air Temperature` | `<ZONA>:Zone Air Temperature (C)` (sin alias) |

En el [[Mezclado-Perfecto|modelo de mezclado perfecto]] (siempre activo en el taller) las dos son **prácticamente idénticas**. Si pediste ambas:

- Usar `Ti_<zona>` (Mean) para análisis.
- La sin alias está casi siempre vacía (NaN) por la frecuencia mezclada — ver sección anterior.

## Diferencia "Heat Gain Rate" vs "Heat Gain"

E+ reporta dos sabores de algunas variables:

| Sabor | Unidades | Interpretación |
|-------|----------|----------------|
| `... Heat Gain Rate` | W (potencia instantánea) | Lo que está pasando en ese paso temporal |
| `... Heat Gain` | J (energía acumulada) | Suma sobre el paso temporal (rate × Δt) |

> Cuidado: graficar Joules vs el rato hace que parezcan números muy grandes/distintos. Para análisis a paso fino casi siempre se usa **Rate** (W).

## Cómo descubrir qué hay disponible

Dos vías:

1. **Leer el RDD** de la simulación (`eplusout.rdd`) — lista lo que **esta** simulación puede reportar. Ver [[RDD-Variables-Disponibles]].
2. **Documentación oficial**: Input/Output Reference (Ctrl+F por keyword: `temperature`, `radiation`, `convection`).

## Cómo pedirlas

Procedimiento detallado en [[../procedures/Solicitar-Output-Variables-Measures]]. Resumen:

- La pestaña **Output Variables** de Open Studio expone solo un subconjunto.
- Para acceso completo se usan **measures** del BCL: `Add Output Variable` (uno por variable que se quiera) + `Create CSV Output` (opcional, útil para verificar).
- El nombre debe copiarse **idéntico** al RDD.

## Incidente exterior vs entrada a la zona

Dos variables que se confunden:

| Variable | Mide |
|----------|------|
| `Surface Outside Face Incident Solar Radiation Rate per Area` (sobre la ventana) | Radiación que **llega** a la cara exterior del vidrio. **No** considera transmitancia ni absortancia. |
| `Zone Windows Total Heat Gain Rate` | Calor neto que **entra** a la zona vía todas las ventanas. **Sí** considera transmitancia/reflectancia/absortancia. |

Para evaluar **el efecto del vidrio en sí** (cambiar de simple a doble vidrio): usar la segunda.

> "La radiación incidente sobre la superficie está bien para sombramiento. Si quisiera evaluar el cambio de transmitancia, me iría a la radiación que ingresa a la zona térmica."

## ⚠️ Nota crítica — sombreamiento en sub-superficies

**`Surface Outside Face Incident Solar Radiation Rate per Area` NO refleja el sombreamiento sobre sub-superficies (ventanas y puertas)**.

Si comparas la radiación incidente sobre una ventana en dos simulaciones (con y sin alero), las series son **idénticas** — esto **no** significa que el alero falle, significa que esa variable reporta la radiación bruta antes del sombreamiento.

**Para auditar sombreamiento sobre ventanas:**

- Pedir `Surface Outside Face Sunlit Fraction` y observar cuándo cae a 0.
- O pedir radiación incidente sobre el **muro padre** (en muros opacos sí refleja sombreamiento).
- O pedir `Zone Windows Total Heat Gain Rate` (la radiación que entra a la zona después de todos los efectos).

Detalle en [[Sunlit-Fraction]] y procedimiento en [[../procedures/Auditar-Sombreamiento-Ventanas]].

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — recorrido del RDD y selección de variables clave para el primer análisis
- [[../classes/007-CasoBaseAleros]] — distinción incidente vs entrada a la zona, cuándo usar cada una
- [[../classes/008-ShadingVentanas]] — `Sunlit Fraction` para auditar sombreamiento; nota crítica sobre sub-superficies
- [[../classes/009-AireAcondicionadoSetPoints]] — variables de Ideal Air Loads (energía cooling/heating)

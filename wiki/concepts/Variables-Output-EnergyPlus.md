---
title: Variables de Output de Energy Plus
type: concepto
tags: [concepto, energyplus, output, variables, postprocesamiento]
aliases: [output variables, variables de salida, output:variable]
clases: [005, 007]
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

Para evaluar **sombreamiento**: usar la primera (lo que el alero bloquea está relacionado con la incidente).

Para evaluar **el efecto del vidrio en sí** (cambiar de simple a doble vidrio): usar la segunda.

> "La radiación incidente sobre la superficie está bien para sombramiento. Si quisiera evaluar el cambio de transmitancia, me iría a la radiación que ingresa a la zona térmica."

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — recorrido del RDD y selección de variables clave para el primer análisis
- [[../classes/007-CasoBaseAleros]] — distinción incidente vs entrada a la zona, cuándo usar cada una

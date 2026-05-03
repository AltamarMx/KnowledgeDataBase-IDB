---
title: Auditar el efecto del sombreamiento sobre ventanas
type: procedimiento
tags: [procedimiento, sombreamiento, ventanas, sunlit-fraction, debugging, validacion]
aliases: [auditar alero, validar sombreamiento, sunlit fraction]
clases: [008]
updated: 2026-05-02
---

# Auditar el efecto del sombreamiento sobre ventanas

Procedimiento para verificar que un alero, parteluz o estrategia de sombramiento sobre una ventana **realmente bloquea radiación**. Necesario porque la variable obvia (`Surface Outside Face Incident Solar Radiation Rate per Area`) **no refleja el sombreamiento en sub-superficies** — ver [[../concepts/Sunlit-Fraction]].

## Cuándo usar este procedimiento

- Comparar dos simulaciones con/sin alero y los resultados de radiación incidente sobre ventanas se ven **iguales**.
- Verificar que un parteluz vertical efectivamente cubre los ángulos azimutales de mañana o tarde.
- Cuantificar el % de radiación efectiva bloqueada para reportar en el proyecto final.

## 1. Pedir las variables correctas

En Open Studio, configurar measures de output ([[Solicitar-Output-Variables-Measures]]):

| Variable | Para qué | Key Value |
|----------|----------|-----------|
| `Surface Outside Face Sunlit Fraction` | Fracción de la superficie al sol directo | nombre de la ventana (ej. `vNorte`) |
| `Surface Outside Face Incident Solar Radiation Rate per Area` | Radiación bruta (sin sombreamiento) | nombre de la ventana |
| `Site Direct Solar Radiation Rate per Area` | Referencia (radiación directa exterior) | `*` (Environment) |
| `Site Diffuse Solar Radiation Rate per Area` | Referencia (radiación difusa) | `*` |
| `Site Solar Altitude Angle` | Para filtrar período diurno | `*` |

> Para auditar **muros opacos** (no ventanas): la radiación incidente sobre muros **sí** refleja sombreamiento — no es necesario usar Sunlit Fraction. La excepción es solo para sub-superficies.

## 2. Cargar las dos simulaciones en Python

```python
from iertools.read import read_sql

base    = read_sql("../OSM/005_caso_base/run/eplusout.sql", alias=False).data
alero   = read_sql("../OSM/006_protecciones/run/eplusout.sql", alias=False).data
```

> Sin alias para mantener nombres originales. Los nombres específicos (`VNORTE:...`) son inestables — verificar con `df.columns`.

## 3. Inspeccionar la Sunlit Fraction

Patrón visual esperado:

| Caso | Comportamiento |
|------|----------------|
| **Sin protección, día despejado** | SF = 1 cuando hay sol directo sobre la ventana, 0 cuando no |
| **Con protección efectiva** | SF = 0 durante períodos extendidos del día — el alero bloquea |
| **Con protección parcial** | SF entre 0 y 1 cuando el sol incide oblicuo y el alero solo cubre parte |
| **Día completamente nublado** | SF = 0 todo el día (no hay rayo directo que medir) |

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 6))

# Panel superior — Sunlit Fraction de los dos casos
SF_col = "VNORTE:Surface Outside Face Sunlit Fraction []"
ax[0].plot(base[SF_col],  label="Sin alero",  color="green", linestyle="-")
ax[0].plot(alero[SF_col], label="Con alero",  color="green", linestyle="--")
ax[0].set_ylabel("Sunlit Fraction")
ax[0].set_ylim(0, 1.1)
ax[0].legend()

# Panel inferior — radiación incidente bruta (referencia, las dos series IDÉNTICAS por la sub-superficie)
IS_col = "VNORTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]"
ax[1].plot(base[IS_col],  label="Sin alero",  color="orange", linestyle="-")
ax[1].plot(alero[IS_col], label="Con alero",  color="orange", linestyle="--")
ax[1].set_ylabel("I incidente bruta (W/m²)")
ax[1].legend()
```

Resultado esperado:

- **Panel inferior** (radiación bruta): las dos curvas se **superponen exactamente** — confirma el "bug" no-bug.
- **Panel superior** (SF): la línea sólida (sin alero) llega a 1; la dasheada (con alero) cae a 0 durante períodos del día → demuestra que el alero **sí funciona**.

## 4. Calcular la radiación efectiva sobre la ventana

La radiación que efectivamente entra al balance térmico de la ventana es:

$$
I_{efectiva} = I_{directa} \cdot \cos\theta \cdot SF + I_{difusa} \cdot F_{s\to sky} + I_{reflejada} \cdot F_{s\to ground}
$$

Aproximación simplificada (asumiendo $\theta$ pequeño y factores de vista no afectados):

```python
# Aproximación: directa modulada por SF + difusa sin atenuar
SF_base  = base["VNORTE:Surface Outside Face Sunlit Fraction []"]
SF_alero = alero["VNORTE:Surface Outside Face Sunlit Fraction []"]
I_b      = base["Environment:Site Direct Solar Radiation Rate per Area [W/m2]"]
I_d      = base["Environment:Site Diffuse Solar Radiation Rate per Area [W/m2]"]

I_eff_base  = I_b * SF_base  + I_d  # aproximación gruesa
I_eff_alero = I_b * SF_alero + I_d
```

> Nota: esta es **simplificación**. El cálculo exacto de E+ proyecta la directa sobre la ventana (multiplica por `cos θ` con θ función del acimut) y modifica los factores de vista. Para auditoría de orden de magnitud, la aproximación basta.

## 5. Métrica del bloqueo

Reducción de radiación efectiva integrada en el período de interés:

```python
import numpy as np

dt = 10/60  # paso de 10 min en horas

E_base  = (I_eff_base  * dt).sum()  # W·h/m² al año
E_alero = (I_eff_alero * dt).sum()
reduccion = (E_base - E_alero) / E_base * 100

print(f"Energía solar incidente sin alero:  {E_base:>8.0f} W·h/m²")
print(f"Energía solar incidente con alero:  {E_alero:>8.0f} W·h/m²")
print(f"Reducción:                          {reduccion:.1f}%")
```

Para una ventana al sur con alero PF=1 bien dimensionado, espera **30-60% de reducción**. Para ventanas E/W con alero horizontal, **<20%** (sol oblicuo no se cubre) — confirma que orientaciones E/W son difíciles de proteger.

## 6. Validar con criterios físicos

### Criterio 1 — SF debería ser 0 en pico solar

Si tu protección tiene `PF ≥ 1` y la ventana mira al sur, la SF al **mediodía solar** del solsticio de verano debería ser **exactamente 0** (el sol está alto, el alero lo bloquea).

```python
# Filtrar días alrededor del solsticio
solsticio = (alero.index.month == 6) & (alero.index.day.between(19, 23))

# Sunlit Fraction al mediodía solar (~13:00 horario civil en MX)
mediodia = alero.index.hour == 13
SF_mediodia = SF_alero[solsticio & mediodia]

print(f"SF promedio mediodía solsticio: {SF_mediodia.mean():.3f}")
# Esperado: cercano a 0 si el alero está bien
```

### Criterio 2 — SF simétrica si la geometría es simétrica

Para una ventana sur con un alero **simétrico** (sin parteluces o con parteluces simétricos a izquierda y derecha), la SF debería ser **simétrica respecto al mediodía solar**.

Si no lo es, hay asimetría en la geometría (un parteluz solo a un lado, una sombra de un edificio vecino, etc.). En la clase 008, el profesor observó asimetría en una ventana norte y dedujo que se debía a un parteluz solo en un lado.

### Criterio 3 — En noches y días nublados, SF = 0

`SF` solo aplica a la **directa**. De noche o con nubes pesadas no hay directa → SF = 0 incluso sin protección. Esto **no significa** protección — significa que no hay nada que bloquear.

Filtrar al período diurno con sol directo:

```python
solar_alt = base["Environment:Site Solar Altitude Angle [deg]"]
I_directa = base["Environment:Site Direct Solar Radiation Rate per Area [W/m2]"]

de_dia_con_sol = (solar_alt > 0) & (I_directa > 50)  # W/m² mínimo
SF_efectiva = SF_alero[de_dia_con_sol]
```

## 7. Diseño iterativo del alero

Patrón de iteración para diseñar un alero efectivo:

1. **Versión inicial** del alero (PF = 0.5).
2. Correr y graficar SF.
3. **Si SF > 0** durante períodos cálidos → alargar alero (PF mayor).
4. **Si SF = 0** todo el invierno → acortar alero (PF menor — está sobre-bloqueando ganancia útil).
5. Iterar hasta encontrar el balance.

Para extender el alero más allá del ancho de la ventana (limitación de Open Studio), ver el workaround manual en [[Agregar-Aleros-OpenStudio]].

## Trampas comunes

| Síntoma | Causa |
|---------|-------|
| Las dos series de radiación incidente son idénticas | **Esperado** en sub-superficies — el sombreamiento no se aplica ahí. Pedir SF |
| SF = 0 todo el día y sin alero | Probablemente la ventana no recibe sol directo (orientación norte en hem. norte, o edificio vecino bloquea) |
| SF distinta de 0 con alero "obvio" | Alero sub-dimensionado o de mismo ancho que la ventana (limitación de Open Studio) — ver [[Agregar-Aleros-OpenStudio]] |
| Ambas SF son 0 todo el día | Día nublado en el EPW — escoger otro día con sol claro para auditar |
| Diferencia muy pequeña en `T` interior | Ventana muy chica (poco efecto absoluto), o radiación dominada por difusa, o T del aire enmascara efecto radiativo local — usar [[Temperatura-Operativa]] |

## Clases relacionadas

- [[../classes/008-ShadingVentanas]] — descubrimiento del rol de la Sunlit Fraction

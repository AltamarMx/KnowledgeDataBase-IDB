---
title: Schedules (Horarios)
type: concepto
tags: [concepto, schedules, horarios, cargas, setpoints, energyplus, openstudio]
aliases: [schedule, horario, schedules energyplus, schedule:compact]
clases: [009, 014]
updated: 2026-05-22
---

# Schedules (Horarios)

## Qué son

**Horarios** que controlan cuándo y con qué intensidad ocurre algo en una simulación de Energy Plus. Cualquier objeto que varía en el tiempo necesita un schedule:

- **Setpoints** de heating y cooling — qué T mantener cada hora.
- **Ocupación** — cuántas personas hay cada hora.
- **Iluminación** — qué fracción de la potencia eléctrica está activa.
- **Equipos** — análogo para electrodomésticos, computadoras.
- **Apertura de ventanas** — fracción del área de ventana abierta (en simulaciones con ventilación natural).
- **Operación de HVAC** — cuándo está prendido el AC.

Sin un schedule, los objetos que dependen del tiempo no pueden definirse.

## Tipos de schedule

E+ requiere asignar un **tipo** al crear cada schedule. El tipo determina las unidades, los límites y cómo se interpretan los valores:

| Tipo | Valores | Para qué |
|------|---------|----------|
| **Temperature** | Continuos, °C, sin límites por default | Setpoints de heating/cooling |
| **Fraction** (`Dimensionless 0-1`) | 0 a 1 | Ocupación parcial, apertura de ventanas, dimming de luces |
| **On/Off** (`Dimensionless 0-1` con valores discretos) | 0 o 1 | HVAC encendido/apagado |
| **Number of People** (`Dimensionless 0-N`) | Continuo, 0 a N | Ocupación absoluta de personas |
| **Power** (`Dimensionless`) | Watts u otra unidad | Cargas eléctricas absolutas |
| **Activity Level** | W/persona | Metabolismo de ocupantes (sentado, parado, ejercicio) |

Algunos tipos tienen **límites superior e inferior**; otros no. Open Studio muestra `lower limit non` y `upper limit non` cuando no hay restricción.

> **Trampa**: el `lower limit` y `upper limit` del tipo de schedule **no son** los valores del schedule mismo. Son los límites de validez. El valor del schedule se define con la línea negra en la interfaz visual.

## Resolución temporal

E+ permite hasta **60 timesteps por hora** (1 minuto). Esto es:

- El máximo de granularidad del schedule.
- El mismo límite que para todo el cálculo dependiente del tiempo de E+.

Eventos que duran **menos de un minuto** no se pueden simular con su tiempo real — hay que repartirlos en un minuto o promediarlos.

> "Si cocinas 10 segundos, asumes que ese tiempo se reparte en un minuto."

## Anatomía de un schedule en Open Studio

La interfaz visual de Open Studio muestra:

```
┌─────────────────────────────────────────┐
│ Schedule type: Temperature              │
│ Lower limit: non    Upper limit: non    │
│                                         │
│ Default profile (always — fallback)     │
│  ┌──────────────────────────────────┐  │
│  │   25 °C ━━━━━━━━━━━━━━━━━━━━━━━ │  │
│  │         00h    08h    18h    24h │  │
│  └──────────────────────────────────┘  │
│                                         │
│ + Run Period (overrides default in...)  │
│ + Holidays (overrides for...)           │
└─────────────────────────────────────────┘
```

Componentes:

- **Default profile** — el horario base; aplica todos los días si no hay overrides.
- **Run periods** — sobreescritura para un rango de fechas (ej. vacaciones).
- **Holidays** — sobreescritura para días específicos.
- **Days of week** — distinguir lunes-viernes vs. fines de semana.

Los overrides se **encimas** al default — si hay un Run Period activo, manda sobre el default.

### Editar el default profile

1. Doble click sobre la línea negra → **inserta un segmento** en ese punto del día.
2. Mover el cursor sobre el segmento → **cambia el valor** (no escribir el valor con teclado primero — escribir mientras el cursor está sobre el segmento).
3. Doble click otra vez sobre el corte → lo elimina.
4. La barra inferior permite hacer zoom: cada hora / 15 min / 1 min.

> "Si pones esto se mueve, pero no se está moviendo el valor. Cuando yo me pongo encima, ahí escribo el valor."

Procedimiento detallado en [[../procedures/Crear-Schedule-Temperatura]].

## Schedule Set

Análogo al Construction Set ([[Construction-Set]]) pero para horarios. Permite agrupar schedules de varios tipos y asignar el set como default a la edificación.

Útil cuando hay muchas zonas con el **mismo patrón de uso** (oficina típica, salón de clase típico, etc.).

## Templates de ASHRAE / Estados Unidos

ASHRAE publica schedules tipo para distintos usos (escuelas, oficinas, hospitales, residencias). Open Studio incluye estos templates y se pueden importar directamente.

> "Lo fantástico de Estados Unidos y de ASHRAE es que ya tienen tipos de schedules y de cargas térmicas para un montón de casos."

Para México **no hay equivalente oficial** — el grupo del IER trabaja en caracterizar el consumo eléctrico de viviendas mexicanas (tesis de Eric Iván) para construir schedules propios.

## Uso en el taller

El taller usa schedules **mínimamente**:

- En el alcance "sin AC, sin cargas internas": no se necesitan schedules.
- Cuando se incorpora AC ideal (clase 009): se requieren schedules de heating y cooling setpoint, aunque sean **constantes todo el día/año**.

Los proyectos finales no incluyen ocupación, luces ni equipos — el cascarón sin cargas internas es suficiente para evaluar estrategias bioclimáticas pasivas.

## Visualización en Python

Cuando se piden output variables de un schedule (`Schedule Value` con el nombre del schedule), las series temporales pueden inspeccionarse para validar:

```python
fig, ax = plt.subplots()
ax.plot(df["heating_setpoint"], label="heating")
ax.plot(df["cooling_setpoint"], label="cooling")
ax.set_ylabel("Setpoint (°C)")
ax.legend()
```

Permite verificar que el schedule llegó al IDF como esperabas (caso real: una zona horaria mal configurada puede desplazar todo el schedule).

## Schedule fraccional para infiltración

Caso específico — el schedule para infiltración ([[Infiltracion-Cambios-Aire]]) **debe ser tipo Fractional (0-1)**. Si se asigna un valor `> 1`, E+ falla con error de validación.

El valor efectivo de la infiltración es:

$$
ACH_{efectivo}(t) = ACH_{diseño} \cdot S(t)
$$

Patrón típico de **ventilación nocturna**:

| Hora | Valor del schedule |
|---|---|
| 00:00 – 06:00 | 1 (ventilando) |
| 06:00 – 22:00 | 0 |
| 22:00 – 24:00 | 1 |

Con `ACH_diseño = 1`, este schedule produce 1 ACH solo de noche. Detalle en [[../procedures/Agregar-Infiltracion-OpenStudio]].

> "Que no les pase lo que me pasó: le quise poner un 2 y la fracción va entre 0 y 1." — clase 014

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — introducción al concepto y demo en vivo de la interfaz visual
- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — schedule fraccional 0-1 para infiltración; bug confesional del profesor (fraction=2); ventilación nocturna como estrategia

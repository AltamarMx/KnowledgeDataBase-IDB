---
title: Temperatura Operativa
type: concepto
tags: [concepto, confort, temperatura, radiacion, energyplus]
aliases: [operative temperature, temperatura operativa, top, temperatura de operacion]
clases: [005, 007]
updated: 2026-05-02
---

# Temperatura Operativa

## Definición

Temperatura "equivalente" que percibe un cuerpo en una zona, combinando el efecto de la **temperatura del aire** y la **temperatura radiante**:

$$
T_{op} = \frac{h_c \, T_{aire} + h_r \, T_{rad}}{h_c + h_r}
$$

donde $T_{rad}$ es la **temperatura radiante media** (T efectiva del intercambio LWR con las superficies que rodean al cuerpo), y $h_c$, $h_r$ son los coeficientes convectivo y radiativo. En primera aproximación con $h_c \approx h_r$:

$$
T_{op} \approx \frac{T_{aire} + T_{rad}}{2}
$$

Es la temperatura que **mediría un termómetro de globo** correctamente calibrado.

## Por qué importa

Un termómetro convencional mide solo la **temperatura del aire**. Pero el confort de una persona depende también del **intercambio radiativo** con las superficies a su alrededor. Casos donde la diferencia es grande:

- **Sol incidente directo**: parado al sol, $T_{op}$ puede ser 10 °C arriba del $T_{aire}$.
- **Pared muy caliente o muy fría enfrente**: el cuerpo intercambia LWR con esa pared y siente más calor (o frío) del que un sensor de aire detectaría.
- **Plancha de concreto** que reflejó sol todo el día: irradia LWR durante la noche, eleva $T_{op}$ aunque el aire ya esté fresco.
- **Tragafuegos a 700+ K**: ejemplo extremo del profesor — el calor sentido al pasar es radiación, no convección.

> "Si yo pongo un sensor de temperatura de aire, no se va a dar cuenta. Aquí donde hay grandes niveles de radiación, la temperatura radiante y por lo tanto la temperatura operativa van a ser muy importantes."

## Cuándo $T_{op} \approx T_{aire}$

Cuando **no hay fuente radiante asimétrica** que sobresalga del balance:

- Habitación con superficies a temperatura similar al aire.
- Sin sol directo entrando.
- Sin equipos calientes ni superficies muy frías.

En esa situación, las superficies están en equilibrio con el aire y $T_{rad} \approx T_{aire}$. Por eso muchos análisis simples asumen $T_{op} = T_{aire}$ y reportan solo $T_{aire}$.

## Cuándo se rompe la aproximación (caso del IER)

Ejemplo del profesor: la planta baja de un edificio con **plancha grande de concreto** y **absortancia baja** (color claro, no absorbe mucho la radiación solar — pero sí refleja). El concreto:

- Recibe radiación todo el día y se calienta.
- De noche emite radiación de onda larga.
- La $T_{op}$ percibida por usuarios queda **alta** aunque el $T_{aire}$ esté templado.

Es disconfort que un análisis basado solo en `Zone Mean Air Temperature` no detecta.

## Cómo obtenerla en Energy Plus

Variable disponible en el RDD:

```
Zone Operative Temperature [C]
```

Se pide igual que cualquier otra: ver [[../procedures/Solicitar-Output-Variables-Measures]].

Variables auxiliares útiles:

- `Zone Mean Radiant Temperature` — la T radiante que E+ calculó.
- `Surface Inside Face Temperature` — T de cada superficie interna (entrada al cálculo de T radiante).

## Sensores virtuales de confort

E+ permite colocar **sensores virtuales** en posiciones específicas dentro de una zona — útiles para análisis fino de confort cuando el [[Mezclado-Perfecto|mezclado perfecto]] subestima efectos locales.

Casos donde la diferencia importa:

- **Sol entrando por una ventana** sobre un escritorio específico — el ocupante recibe radiación directa que el T del aire de zona no captura.
- **Pared muy fría o muy caliente** cerca del usuario — efecto LWR local.

Configurar sensores virtuales requiere editar el IDF (no expuesto en Open Studio) — fuera del alcance del taller, pero relevante saber que existen.

> "Si es radiación directa, incidente sobre un espacio, eso puede hacer que esos sensores virtuales que pones en Energy Plus se disparen — porque el confort se basa en T operativa, y la T operativa se basa en T radiante."

## Implicaciones para diseño bioclimático

- **Análisis de confort** debe usar $T_{op}$, no solo $T_{aire}$, sobre todo cuando:
  - Hay vidrios grandes (radiación que entra cae sobre superficies internas).
  - Hay superficies muy masivas con temperaturas distintas al aire (concreto exterior expuesto).
  - Se evalúa confort en climas con alta radiación solar.
- **Estrategias** que actúan sobre la radiación (sombras, color, masa térmica) afectan $T_{op}$ vía $T_{rad}$, aunque el efecto sobre $T_{aire}$ sea modesto.

## Relación con confort adaptativo

Los modelos de [[Confort-Adaptativo]] (Humphreys-Nicol, ASHRAE 55 adaptativo) suelen estar formulados sobre $T_{op}$, no sobre $T_{aire}$. Cuando se reporta confort en horas-grado, debería usarse $T_{op}$ — la diferencia con $T_{aire}$ puede cambiar materialmente el resultado.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — definición y por qué se debe pedir como variable de output
- [[../classes/007-CasoBaseAleros]] — sensores virtuales de confort, caricatura del rayo de luz local

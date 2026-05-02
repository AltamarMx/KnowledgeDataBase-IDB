---
title: Sunlit Fraction
type: concepto
tags: [concepto, sombreamiento, ventanas, energyplus, debugging, output]
aliases: [sunlit fraction, fraccion soleada, sunlit area, surface sunlit]
clases: [008]
updated: 2026-05-02
---

# Sunlit Fraction

## Qué es

Variable de Energy Plus `Surface Outside Face Sunlit Fraction`: fracción de la superficie que **recibe radiación directa del sol** en cada paso temporal, después de aplicar todas las sombras (aleros, parteluces, edificios vecinos, vegetación, geometría propia).

Valor entre 0 y 1:

| Valor | Significado |
|-------|-------------|
| 1.0 | Toda la superficie recibe radiación directa — ningún sombreamiento |
| 0.5 | La mitad está al sol, la mitad en sombra |
| 0.0 | Toda la superficie está sombreada — la protección es completa |

Existe también `Surface Outside Face Sunlit Area` (m²) — área absoluta en lugar de fracción.

## El descubrimiento crítico — sub-superficies

> **En ventanas (sub-superficies), `Surface Outside Face Incident Solar Radiation Rate per Area` NO refleja el sombreamiento**.

Si pides la radiación incidente sobre una ventana en dos simulaciones (una con alero, una sin alero), las dos series temporales se ven **iguales**. Esto **no** significa que el alero no funcione — significa que E+ no aplica el sombreamiento a esa variable.

### Por qué E+ lo hace así

E+ calcula la **fracción de Sunlit** en cada paso temporal y la usa **al hacer el balance de calor**, no al reportar la radiación incidente:

$$
q''_{absorbida} = \alpha \left[ I_b \cdot \cos\theta \cdot SF_s + I_d \cdot F_{s\to sky} + I_g \cdot F_{s\to ground} \right]
$$

donde:

- $I_b$ = radiación directa (beam)
- $I_d$ = radiación difusa
- $I_g$ = radiación reflejada por el ground
- $SF_s$ = **Sunlit Fraction** (atenúa solo la directa)
- $F_{s\to sky}$, $F_{s\to ground}$ = factores de vista (atenúan difusa y reflejada)
- $\alpha$ = absortancia
- $\theta$ = ángulo entre rayo y normal

Por eso la radiación incidente **bruta** se reporta sin sombreamiento — el sombreamiento se aplica en otro paso del cálculo.

> "Energy Plus no calcula la radiación incidente con sombreamiento, sino más bien la fracción de sombreamiento, y luego une todo. Por eso, cuando volteamos a ver esa variable, no lo notamos."

### En muros opacos sí refleja sombreamiento

Curiosidad: para muros opacos, `Surface Outside Face Incident Solar Radiation` **sí** muestra el efecto del sombreamiento. La simplificación es **específica para sub-superficies (ventanas y puertas)**.

Por eso una vía de auditar el efecto del alero es pedir la radiación incidente sobre **el muro padre** (que sí refleja el sombreamiento) o sobre la ventana **junto con** la Sunlit Fraction.

## Cómo auditar un alero/parteluz

Procedimiento detallado en [[../procedures/Auditar-Sombreamiento-Ventanas]]. Resumen:

1. Pedir como output:
   - `Surface Outside Face Sunlit Fraction` para la ventana protegida.
   - `Surface Outside Face Incident Solar Radiation Rate per Area` (referencia bruta, sin sombreamiento).
2. En Python: graficar la Sunlit Fraction.
3. Validar:
   - **Sin protección**: `SF = 1` cuando hay sol directo, `SF = 0` cuando el sol no incide (noche, nubes, sombra propia de la edificación).
   - **Con protección efectiva**: `SF = 0` durante períodos extendidos del día — el alero bloquea el sol.
4. Calcular **radiación efectiva**:
   ```python
   I_efectiva = I_directa * SF + I_difusa * F_sky + I_ground * F_ground
   ```

   Comparación de la integral de `I_efectiva` entre casos con y sin protección → cuantifica el efecto.

## Solo afecta la radiación directa

> Importante: la Sunlit Fraction **solo aplica a la radiación directa**. La difusa y la reflejada por el ground no usan SF — se atenúan via los **factores de vista** entre la superficie y el cielo / ground / surroundings.

Implicación: una protección que reduce SF a 0 **no bloquea** el 100% de la radiación. La difusa sigue llegando (con su propio factor de vista atenuado). En un día completamente nublado, la radiación es 100% difusa → la protección casi no atenúa.

> "La fracción me diría la relación de energía o de potencia incidente sobre cualquier instante. Pero no es la radiación total bloqueada — la directa la bloquea, la difusa no."

## Ventaja para diseño bioclimático

Con la Sunlit Fraction se puede demostrar **numéricamente** que un alero está bien diseñado:

> **Criterio**: filtra los datos para `Site Solar Altitude Angle > 0` (de día). En esos períodos, si quieres protección completa, esperas `SF = 0`.

Si la SF nunca cae a 0 durante el día en períodos cálidos, el alero está sub-dimensionado. Si cae a 0 incluso en invierno, el alero está sobre-dimensionado y bloquea el sol cuando se quiere ganancia.

Este patrón — visualizar la SF en lugar de la radiación incidente — es la herramienta para iterar el diseño del alero.

## Asimetría observada — pista física

En la clase 008, al graficar la SF de una ventana norte se observa **asimetría temporal** (la mañana no es espejo de la tarde). Hipótesis del profesor: los aleros / parteluces de la simulación no son simétricos — un quebrasol al este o al oeste rompe la simetría temporal.

> Validación: diseñar un caso simétrico debería producir SF simétrica. Si no, hay un bug.

Lección: la asimetría en SF es esperable cuando los elementos de sombramiento son asimétricos. Antes de declarar un bug, verificar que la geometría sí es simétrica.

## Clases relacionadas

- [[../classes/008-ShadingVentanas]] — descubrimiento del rol de Sunlit Fraction y resolución del bug de la clase 007

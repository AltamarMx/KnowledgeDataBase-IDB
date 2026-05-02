---
title: Cálculo de Sombramientos
type: concepto
tags: [concepto, energyplus, sombras, radiacion-solar, performance]
aliases: [shadow calculation, shadow update, mascaras de sombramiento]
clases: [004]
updated: 2026-05-02
---

# Cálculo de Sombramientos

## Qué calcula E+

La proyección de **sombras sobre cada superficie** debido a:

- La trayectoria solar aparente (función de hora del año + latitud + longitud).
- Aleros, parteluces y otros elementos de sombreamiento de la propia edificación.
- Edificaciones vecinas modeladas como objetos de sombreamiento.

El resultado son **máscaras** que indican qué fracción de cada superficie está al sol y qué fracción en sombra para cada paso temporal.

## La aproximación de E+: actualización periódica

E+ **no recalcula las máscaras en cada paso temporal** — sería caro. Por default, recalcula **cada 20 días** (parámetro `Shadow Calculation Update Frequency`).

Ejemplo de la salida del `.err`:

```
Updating Shadowing Calculations, Start Date=February 03
Updating Shadowing Calculations, Start Date=February 22
...
```

Entre dos actualizaciones, E+ usa la máscara calculada en la última fecha y la aplica a todos los pasos intermedios.

## Por qué la aproximación es razonable

La trayectoria solar cambia **lentamente día a día** — la posición del sol al mediodía se desplaza pocos grados en 20 días. Para análisis bioclimático con resolución horaria a diaria, la diferencia introducida por congelar las sombras 20 días es típicamente menor que otras incertidumbres del modelo (propiedades de materiales, condiciones de frontera, etc.).

## Cuándo es problemática

### Iluminación natural (daylighting)

Para análisis fino de iluminación natural, la aproximación de 20 días pierde detalle relevante:

- Cambios día a día en niveles de iluminancia interior pueden ser percibidos por usuarios.
- Un alero que sombrea perfectamente el 21 de junio puede dejar pasar sol el 1 de julio — la diferencia importa para diseño de protecciones.

> "Por eso E+ no es el mejor programa para iluminación natural."

**Alternativa**: **Radiance** recalcula sombras cada hora usando *backward ray tracing* (técnica numérica más eficiente que el ray tracing directo, aunque sigue siendo más caro que la aproximación de E+). Es el estándar para análisis de iluminación natural.

### Diseño fino de protecciones solares

Si el objetivo es ajustar la longitud o inclinación de un alero al milímetro para una ventana específica, conviene aumentar la frecuencia de actualización a cada 5 días o cada día.

## Cómo cambiar la frecuencia

En el IDF (objeto `ShadowCalculation`):

- `Shadow Calculation Update Frequency` — cada cuántos días recalcular (default: 20).
- `Shadow Calculation Method` — `PolygonClipping` (default) o `PixelCounting` (más rápido en GPUs modernas).
- `Maximum Figures in Shadow Overlap Calculations` — límite para geometrías complejas.

En Open Studio: típicamente vía un IDF Measure, ya que la GUI no expone el objeto directamente con todas sus opciones.

## Trade-off de cómputo

E+ nació en 1996 como evolución de programas previos (BLAST, DOE-2). En esa época cada cálculo de sombramiento era costoso. Hoy las computadoras absorben el costo, pero la default de 20 días se mantiene por compatibilidad y porque para la mayoría de análisis térmicos la diferencia no se nota.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — primera mención al inspeccionar el `.err` y notar el `Shadow Calculation Update`

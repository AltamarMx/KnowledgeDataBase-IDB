---
title: Site / Source Factor
type: concepto
tags: [concepto, energia, lca, ciclo-de-vida, normativas, mexico]
aliases: [site source factor, factor sitio fuente, energia primaria]
clases: [004]
updated: 2026-05-02
---

# Site / Source Factor

## Definición

Factor de conversión entre la **energía consumida en el sitio** (`site energy`, lo que mide el medidor de la edificación) y la **energía requerida en la fuente** (`source energy` o energía primaria, lo que costó producir y transportar esa unidad hasta el sitio).

$$
E_{source} = E_{site} \times f_{site \to source}
$$

donde el factor $f$ depende del **vector energético**: electricidad de la red, gas natural, gas LP, leña, etc.

## Por qué $f \neq 1$ para electricidad

Una unidad de electricidad en el sitio "costó" varias unidades en la fuente:

1. **Eficiencia de la planta generadora**: termoeléctrica de gas natural ~50-60%; de carbón ~35-40%. Una unidad eléctrica salió de varias unidades de calor del combustible.
2. **Pérdidas en transmisión y distribución**: ~5-10% adicional.
3. **Otros costos energéticos** del ciclo: extracción, transporte de combustible, etc.

Resultado típico en EE.UU.: **~3.0-3.2** para electricidad.

> "En Estados Unidos, la electricidad tiene un factor de tres veces. Eso quiere decir que perdemos 3 veces una unidad para que llegue acá."

Para gas natural el factor es cercano a 1.0 (la pérdida principal es en distribución, ~5%).

## Para qué sirve

### Edificaciones de energía cero (Net Zero)

Una edificación net-zero en **site energy** consume tanto como genera localmente medido en el medidor. Una en **source energy** lo hace medido en energía primaria — un objetivo más exigente porque considera las pérdidas del sistema.

> Tesis mencionada: trabajo de **Nachito** (egresado del IER, originario de Acapulco) sobre las múltiples definiciones de edificaciones de energía cero — Net Zero Site, Net Zero Source, Net Zero Cost, Net Zero Emissions — y los retos de cada una.

### Análisis de Ciclo de Vida (LCA)

LCA contabiliza la energía total que costó construir y operar la edificación a lo largo de su vida útil. La operación se mide en source energy para reflejar el impacto real sobre el sistema energético.

### Comparación entre vectores

Comparar un calentador eléctrico (eficiencia 100% en el sitio, factor ~3 en fuente) contra un calentador de gas (eficiencia 80% en el sitio, factor ~1) requiere convertir todo a fuente para no engañar al diseñador.

## Estado en México

> "México **no tiene** site source factors oficiales completos. Existen los factores del **Sistema Eléctrico Nacional**, pero esos son **solo de transmisión** — nos faltan los de eficiencia de plantas. México está en pañalitos."

Implicaciones prácticas:

- Un análisis riguroso de LCA en México requiere ensamblar factores manualmente o adaptar los de EE.UU. con las salvedades del caso.
- El registro de la **CFE** (Comisión Federal de Electricidad) da el dato más fácil de consumo, pero no incluye las pérdidas completas — subestima el impacto.
- Para el caso del **viento** y otras renovables, definir el factor es no trivial (¿partir del potencial teórico máximo?). El profesor lo deja como pregunta abierta.

## Dónde aparecen en Open Studio

El reporte HTML que genera E+ tras la simulación incluye una sección **Site and Source Energy** con factores típicos de EE.UU. precargados. En el reporte se ven los factores aplicados — útil para auditar el supuesto pero **no se debe asumir como dato local**.

En el alcance del curso este reporte se pasa por encima — el taller analiza temperaturas, no energía consumida.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — primera mención al recorrer el reporte HTML y ver la sección Site / Source

---
title: Estudio Paramétrico
type: concepto
tags: [concepto, metodologia, estudio-parametrico, bioclimatico, comparacion]
aliases: [estudio parametrico, parametric study, sensitivity analysis, sensibilidad]
clases: [007, 012]
updated: 2026-05-08
---

# Estudio Paramétrico

## Qué es

Conjunto de simulaciones que **difieren en una sola dimensión** del modelo (un parámetro o decisión de diseño), todas las demás iguales. Comparar los resultados aísla el efecto de la dimensión variada — el principio de "todo lo demás igual" trasladado a simulación.

Para evaluar diseño bioclimático, esta es la metodología natural: en lugar de adivinar qué estrategia mejora más el confort, **se simula cada una** y se comparan los resultados.

## Estructura típica

| Componente | Cantidad | Rol |
|------------|----------|-----|
| **[[Caso-Base]]** | 1 | Referencia sin estrategias |
| **Variantes individuales** | N | Caso base + cada estrategia probada por separado |
| **Caso combinado** | 1 (opcional) | Caso base + todas las estrategias juntas |

Total típico para el proyecto final del taller: **5 simulaciones** (1 base + 3 estrategias + 1 combinada).

## Por qué solo cambiar una dimensión a la vez

Si dos variantes difieren en **dos dimensiones**, no se puede atribuir la diferencia de resultados a una de ellas en particular. Es el problema clásico de aislamiento de variables en experimentación.

Ejemplo:

| Caso | Color | Aleros | Resultado |
|------|-------|--------|-----------|
| Base | Negro | Sin alero | T_max = 35°C |
| Variante A | Blanco | Sin alero | T_max = 31°C ← efecto del color aislado |
| Variante B | Negro | Con alero | T_max = 32°C ← efecto del alero aislado |
| Combinado | Blanco | Con alero | T_max = 28°C ← efecto conjunto |

Comparar `Combinado` con `Base` no responde "¿qué importó más?". Sí lo responden las variantes individuales.

### Sinergia y antagonismo

`Combinado` ≠ `Variante A` + `Variante B` necesariamente:

- **Sinergia positiva**: el efecto combinado es **mayor** que la suma — las estrategias se potencian (ej. aislamiento + color claro).
- **Antagonismo**: el efecto combinado es **menor** que la suma — una estrategia anula parte de la otra (ej. masa térmica grande con ventilación nocturna intensiva).
- **Aditivo simple**: la suma es similar al combinado — las estrategias actúan independientemente.

Solo se puede saber **simulando** los tres casos.

## Estrategias bioclimáticas comunes para el taller

| Estrategia | Parámetro modificado | Variable de salida sensible |
|------------|----------------------|------------------------------|
| **Color** | Solar Absorptance del muro/techo (α: 0.7→0.3) | T superficies exteriores → T zona |
| **Sombreamiento** | Aleros, parteluces, vegetación | Radiación incidente sobre ventanas → ganancias solares |
| **Aislamiento** | Capa de aislante (EPS, lana mineral) en sistema constructivo | Conducción a través de la envolvente |
| **Masa térmica** | Espesor del muro o `InternalMass` | Atenuación y desfase térmico |
| **Orientación** | Rotación de la edificación (`North Axis` en Facility) | Distribución horaria de radiación |
| **WWR (Window-to-Wall Ratio)** | Tamaño de ventanas | Ganancia solar + pérdida por conducción |
| **Ventilación nocturna** | (Avanzado, no en taller) | Disipación nocturna de masa térmica |

## Métricas para reportar

Las métricas se eligen **antes** de ramificar. Típicas:

| Métrica | Fórmula |
|---------|---------|
| **% del año en confort** | Fracción de pasos temporales con T_op dentro de la banda de [[Confort-Adaptativo]] |
| **Grados-hora cálidos** | $\sum_t \max(T_{op}(t) - (T_{neut} + \Delta T), 0) \cdot \Delta t$ |
| **Grados-hora fríos** | Análogo para banda inferior |
| **T máxima / mínima / promedio** del aire interior | Por zona |
| **Reducción relativa** | $(M_{base} - M_{variante})/M_{base}$ |

Calcularlas para cada simulación y reportarlas en una tabla **lado a lado** con las diferencias relativas.

## Encuadre del proyecto final 2026-2

La clase 012 fija reglas adicionales sobre cómo presentar y discutir el estudio:

- **Etiquetar las estrategias por nombre** (`estrategia color`, `estrategia ventanas`, `estrategia orientación`) — no `estrategia 1, 2, 3`. La etiqueta debe comunicar al revisor sin necesidad de abrir el reporte.
- **Las estrategias deben mejorar.** Si una variante no mejora respecto al caso base, descartarla y proponer otra (no se reporta como "estrategia fallida").
- **No automatizar la corrida.** Estar mirando series temporales en cada caso — una métrica puede ocultar trade-offs (la amplitud baja pero el promedio sube, etc.).
- **Acotar a meses críticos.** Para el proyecto se evalúa sólo el mes cálido y/o el mes frío (CONUEE). Una vivienda real exige análisis anual; el proyecto no.
- **Climas extremosos**: si hay dos meses críticos, **priorizar** explícitamente y reportar la decisión — la mejora rara vez es simétrica.

Detalle completo en [[../classes/012-ProyectoFinal]].

## Workflow recomendado

1. **Construir el caso base** y revisarlo a fondo (ver [[Caso-Base]]).
2. **Definir métricas** y configurar las **output variables** que las alimentan.
3. **Congelar el caso base** — `Save As → 005_caso_base.osm`.
4. **Para cada estrategia**: `Save As → 00X_estrategia_Y.osm` desde el caso base, aplicar el cambio único, correr.
5. **Caso combinado**: `Save As → 00X_combinado.osm` desde el caso base, aplicar **todos** los cambios, correr.
6. **Análisis comparativo en Python**: ver [[../procedures/Comparar-Simulaciones-Python]].

## Trampa común — descubrir un bug en el caso base después de ramificar

Si tras correr todas las variantes se descubre un error en el caso base (ej. una pared mal orientada, un material con densidad incorrecta), hay que:

1. Corregir el caso base.
2. **Propagar el cambio a todas las variantes** (manualmente, una por una).
3. Re-correr todas las simulaciones.
4. Re-analizar.

Por eso es **crítico** revisar el caso base a fondo antes de ramificar. Detalle en [[Caso-Base]] sección "Checklist antes de ramificar".

## Anécdota — alero equivalente vs detallado

Una ex-alumna del grupo del IER (Paloma) hizo un mini estudio paramétrico para validar una **caricatura**: simular una protección complicada (rejilla con muchos listones) como un **alero equivalente** con transmitancia. Diferencia entre las dos simulaciones: **<2%**.

Conclusión: cuando una caricatura está bien construida, **el orden y magnitud del efecto se conservan** aunque la geometría se simplifique. Esto justifica usar caricaturas en estudios paramétricos sin perder validez de las comparaciones relativas. Detalle en [[Caricatura-Computacional]].

## Clases relacionadas

- [[../classes/007-CasoBaseAleros]] — introducción al concepto y aplicación al proyecto final
- [[../classes/012-ProyectoFinal]] — encuadre 2026-2: etiquetas por nombre, mejora obligatoria, mes crítico, no automatizar

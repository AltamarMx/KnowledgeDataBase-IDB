---
title: Limpiar Geometría
type: concepto
tags: [concepto, geometria, openstudio, sketchup, debugging, intersect]
aliases: [limpiar geometria, intersect surfaces, surface matching, limpiar la geometria]
clases: [006, 007]
updated: 2026-05-02
---

# Limpiar Geometría

## Qué es

Conjunto de operaciones manuales o automáticas para **resolver inconsistencias geométricas** entre superficies que deberían compartir condición de frontera, pero no lo hacen porque sus polígonos no coinciden exactamente.

> Concepto del oficio: "limpiar la geometría". No es un proceso formal de E+ — es lo que hace un modelador cuando E+ se comporta raro por superficies mal alineadas.

## Cuándo aparece el problema

Para que Open Studio convierta automáticamente una condición de frontera de **Outdoors** a **Surface** (interzona, color verde) entre dos espacios, **deben pasar dos cosas**:

1. Las superficies se **traslapan** (sus planos coinciden geométricamente).
2. Las superficies son **del mismo tamaño**.

Si solo se cumple el primer punto — los planos se tocan pero los rectángulos son distintos — Open Studio **no convierte la condición**. Las superficies quedan como Outdoors aunque físicamente colinden.

### Ejemplo típico

Dos espacios: uno chico de 3×3, otro grande de 5×5, adosados por un muro:

```
   ┌─────────┐  ┌──────┐
   │         │  │      │
   │  GRANDE │  │CHICO │
   │  5×5    │  │ 3×3  │
   │         │  │      │
   └─────────┘  └──────┘
```

El muro del chico se traslapa con **una parte** del muro del grande, pero el muro del grande sigue siendo más alto. Open Studio:

- Mantiene el muro del chico como Outdoors.
- Mantiene el muro del grande completo como Outdoors.
- **No genera la frontera Surface** entre ellos.

Resultado: el calor que sale del chico va al ambiente y el del grande también — la edificación no se acopla térmicamente como debería.

## La solución — intersect

La operación que arregla esto se llama **intersección de superficies** (Surface Intersection): proyectar la superficie pequeña sobre la grande y cortar la grande en sub-polígonos coincidentes.

Resultado:

```
   ┌─────┬───┐  ┌──────┐
   │     │   │  │      │
   │     │ A │  │CHICO │
   │  G  │   │  │ 3×3  │
   │     ├───┤  │      │
   │     │ B │  │      │
   └─────┴───┘  └──────┘
```

Ahora:

- La sub-superficie `A` del muro grande **es del mismo tamaño** que el muro del chico.
- E+ puede convertir esa pareja a Surface (interzona, verde).
- Las sub-superficies `B` y la parte del muro del chico que no se traslapa siguen como Outdoors.

## Quién lo hace en cada flujo

| Editor | Comportamiento |
|--------|----------------|
| **FloorspaceJS** (integrado en Open Studio) | Lo hace **automáticamente** en muchos casos — al unir espacios físicamente, corta superficies. No siempre — geometrías complejas pueden quedar mal. |
| **SketchUp** (con plugin Open Studio) | **Manual** — herramienta `Surface Intersect` del plugin. El usuario decide cuándo aplicar. |
| **Rhino + LadyBug** | **Manual** — comando `Project` o `Boolean Difference`. |
| **Programas BIM** (Revit, ArchiCAD) | Geometrías complejas que casi siempre requieren limpieza extensa al exportar a IDF. |

## Por qué no se ve mucho en el taller

El curso usa **cubos simples** (uno o dos espacios cuadrados con alturas iguales). FloorspaceJS resuelve la intersección automáticamente. La limpieza se vuelve relevante cuando:

- Los espacios tienen **alturas distintas** (el techo del chico se traslapa con el muro del grande).
- Hay **múltiples pisos** con plantas distintas.
- Se importa geometría de **BIM** (Revit, ArchiCAD).
- La **envolvente es compleja** (techo a doble agua, volúmenes irregulares).

> "Esto suele pasar cuando las geometrías son muy complejas en programas de BIM que tienen muchos elementos. Lo que vamos a hacer nosotros es tratar de no llegar a ese nivel de complejidad para que no tengamos que limpiar las geometrías."

## Cómo detectar superficies sin limpiar

Síntomas en Open Studio:

- En **Render By → Boundary Conditions**, ves **azul** (Outdoors) donde esperabas **verde** (Surface).
- En la pestaña Surfaces, dos superficies que deberían ser una pareja `Outside Boundary Condition: Surface` quedan como `Outdoors`.
- En los resultados, la simulación muestra **enfriamiento exterior** en una zona que debería estar acoplada con su vecina.

## Cómo forzarlo manualmente en Open Studio

Si FloorspaceJS no lo resolvió y no se quiere ir a SketchUp:

1. Pestaña **Spaces → Surfaces**.
2. Identificar la superficie afectada por nombre (`Face N`).
3. Cambiar manualmente la columna `Outside Boundary Condition` a `Surface`.
4. En la columna `Outside Boundary Condition Object`, seleccionar la cara contraria.
5. Cambiar el `Outside Boundary Condition Object` de la otra superficie también.

Funciona si las dos superficies son del mismo tamaño. Si no, hay que cortarlas primero (o re-dibujar).

## Estrategia preventiva

- **Diseño modular**: dimensionar espacios con alturas que coincidan cuando sea razonable.
- **FloorspaceJS junta primero, separa después**: dibujar un solo gran rectángulo y dividirlo internamente con paredes — generalmente sale más limpio que dibujar dos rectángulos separados que se juntan.
- **Verificar visualmente** con Render By Boundary tras cada cambio de geometría.

## Caso de la clase 006

El profesor demostró el caso opuesto: dos cubos de **alturas distintas** (2.5 m y 5 m), y FloorspaceJS **resolvió automáticamente** la intersección — el muro del cubo bajo se traslapa con la parte inferior del muro del cubo alto, y FloorspaceJS cortó el polígono alto en dos sub-superficies. La parte de abajo quedó como Surface (verde) con el cubo bajo; la parte de arriba quedó como Outdoors (azul). Demostración limpia de cuándo el editor "limpia" sin intervención.

## Bug recurrente — FloorspaceJS rehace el modelo y rompe metadatos

> Cuando se hace un cambio geométrico en FloorspaceJS, Open Studio **rehace el modelo internamente** y puede perder:
>
> - **Nombres custom** de superficies (`Techo` → `Face 9`).
> - **Condición de frontera** específica (piso `Adiabatic` → `Ground`).
> - Asignaciones individuales que sobreescribían el Construction Set.

Síntoma típico observado en la clase 007: tras agregar aleros, las dos simulaciones (con y sin aleros) producen radiación incidente igual sobre el techo — porque el nombre `Techo` se perdió y la variable solicitada por nombre específico no encontró la superficie.

**Detección**: tras cualquier cambio geométrico, verificar:

1. Pestaña **Spaces → Surfaces** y **Sub Surfaces** — los nombres custom siguen ahí.
2. La condición de frontera del piso sigue siendo `Adiabatic`.
3. Tras correr, contar variables en el SQL — debe coincidir con las solicitadas.

Detalle del bug en [[Mensajes-EnergyPlus]] sección "Bugs recurrentes".

## Clases relacionadas

- [[../classes/006-DosZonasTermicasVentanasAleros]] — demostración de FloorspaceJS cortando automáticamente al hacer dos cubos de alturas distintas
- [[../classes/007-CasoBaseAleros]] — bug observado: cambios geométricos borran nombres custom y rompen variables solicitadas

# Agregar Ventanas en Open Studio

Procedimiento para agregar ventanas y protecciones solares a una simulación existente.

## Prerrequisitos

- Simulación funcionando (geometría + materiales + Construction Set + EPW)
- Guardar como nueva versión antes de hacer cambios

## Paso 1: Agregar ventana en FloorSpaceJS

1. Ir al editor de geometría (Geometry → Editor)
2. Reducir el grid a **0.25m** para mayor precisión en la colocación
3. Cambiar a modo **Component**
4. Seleccionar o crear un componente tipo **Window**
5. Configurar propiedades: height, width, sill height (antepecho)
6. Hacer clic sobre el muro donde se quiere la ventana
7. Repetir para cada ventana

**Tip:** todas las ventanas del mismo componente son idénticas. Para diferentes tamaños, crear nuevos componentes.

## Paso 2: Agregar overhangs y fins (opcional)

En las propiedades del componente de ventana:
- **Overhang Projection Factor:** profundidad horizontal / altura de la ventana (ej. 1 = misma profundidad que altura)
- **Fin Projection Factor:** profundidad vertical / ancho de la ventana (ej. 0.5)

## Paso 3: Merge y verificar geometría

1. Hacer **Merge with Current OSM**
2. Ir al **3D View** para verificar que las ventanas aparecen (transparentes)
3. Verificar en **Render by Boundary** que las condiciones de frontera no se alteraron

## Paso 4: Material de vidrio

1. Ir a **Construction** y crear una nueva construction para ventana
2. Usar el vidrio **Clear 3mm** de la librería de EnergyPlus (ya viene incluido)
3. Arrastrar el material a la construction

## Paso 5: Asignar en Construction Set

1. Ir al **Construction Set**
2. En **Exterior Sub Surface Construction** → Fixed Window, asignar la construction de ventana
3. Verificar en **Sub Surfaces** que todas las ventanas tienen sistema constructivo asignado

## Paso 6: Correr y verificar

1. Guardar (Save As con nuevo número de versión)
2. Correr la simulación
3. Revisar el archivo ERR
4. Error común: sub-superficie sin construction asignada

## Aparece en

- [[006-DosZonasTermicasVentanasAleros]] — Demostración completa del procedimiento

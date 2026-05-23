---
title: Espacio vs Zona Térmica
type: concepto
tags: [concepto, openstudio, modelado, nomenclatura]
aliases: [space vs thermal zone, espacio openstudio, thermal zone]
clases: [003, 006, 014]
updated: 2026-05-22
---

# Espacio vs Zona Térmica

## Por qué hay dos conceptos

En Open Studio existen **dos entidades distintas** que para casos sencillos parecen redundantes:

| Entidad | Qué es | Existe en E+ directo |
|---------|--------|----------------------|
| **Espacio** (`Space`) | Volumen geométrico delimitado por superficies. Puede agrupar tipo de uso, cargas internas tipo (Space Type), schedules. | **No** — es un concepto de Open Studio para facilitar plantillas. |
| **Zona térmica** (`Thermal Zone`) | Volumen de aire donde E+ resuelve la temperatura aplicando el balance. Una zona puede contener uno o más espacios. | **Sí** — es el objeto que E+ realmente simula. |

## Mapeo en el curso

En el taller cada **espacio se mapea 1:1 con una zona térmica** — se ignora la capa de Space Types (no se modelan cargas internas tipo "centro de cómputo").

A pesar del mapeo 1:1, **espacio y zona térmica son objetos distintos en Open Studio** y deben definirse por separado:

1. Crear el espacio (en el editor FloorspaceJS, dibujando la planta).
2. Crear la zona térmica (en la pestaña Thermal Zones del Open Studio Application).
3. Asignar la zona térmica al espacio (drag-and-drop desde la columna Thermal Zone en la pestaña Spaces).

## Convención de nombres del profesor

Espacios y zonas térmicas no pueden compartir el mismo nombre. La convención sugerida (refinada en clase 014):

| Tipo | Nombre | Ejemplo |
|------|--------|---------|
| Espacio | Sufijo `_E` o `_S` (de *Space*) — convención moderna | `recamara_E`, `estancia_E` |
| Espacio | Prefijo `S:` (de *Space*) — variante antigua | `S:Norte`, `S:Sur` |
| Zona térmica | Nombre del lugar tal cual | `recamara`, `estancia`, `Norte` |

> "Acuérdense que el espacio y las zonas térmicas no se pueden llamar de la misma manera. Pónganle `_E` o `_S` de Space." — clase 014

Ambas convenciones funcionan. La de sufijo es más moderna porque resiste mejor el ordenado alfabético en listados.

**Reglas de nomenclatura adicionales:**

- **Sin acentos, sin eñes, sin espacios** en nombres. Python/E+ pueden fallar con caracteres no-ASCII.
- **Sin `Thermal Zone 1`, `Thermal Zone 2`** — en una semana no se sabrá qué es cada cosa.
- Una palabra preferentemente; si necesitas dos, **CamelCase** o **guion-medio**.
- Para las zonas térmicas: pegado o con guion medio. **No espacios** — algunos scripts del grupo fallan al parsear espacios.

## Por qué Open Studio expone Spaces

La capa de espacios permite definir **Space Types** reutilizables: un "salón de clase tipo" con cargas (personas, equipos, iluminación) y schedules predefinidos, y luego asignar ese tipo a 30 espacios que comparten esa configuración. Es útil en proyectos comerciales con muchas zonas similares (oficinas, hospitales, escuelas). En el curso no se aprovecha porque no se modelan cargas internas.

## Alturas — heredables y sobreescribibles

En FloorspaceJS la altura del volumen se controla en dos niveles:

| Nivel | Default | Cómo se fija |
|-------|---------|--------------|
| **Story** | `Floor to Ceiling Height` global del piso (default 2.43 m) | Panel Stories del editor |
| **Space** | Hereda del Story | Cada Space puede sobreescribirla individualmente |

Si se cambia la altura en el **Story**, todos los Spaces de ese nivel que no la hayan sobreescrito heredan el cambio. Si se cambia en un **Space** específico, solo ese Space se modifica.

Esto permite modelos con espacios de **alturas distintas** en el mismo piso (ej. una sala doble altura junto a habitaciones normales). FloorspaceJS resuelve la geometría: si los techos quedan a alturas distintas, **corta automáticamente** los muros que ahora se traslapan parcialmente. Ver [[Limpiar-Geometria]].

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — primera vez que se distinguen explícitamente
- [[../classes/006-DosZonasTermicasVentanasAleros]] — alturas distintas por Space, corte automático de superficies traslapadas
- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — convención sufijo `_E`/`_S` para distinguir Space de ThermalZone; uso de Space Types como contenedores de cargas (infiltración)

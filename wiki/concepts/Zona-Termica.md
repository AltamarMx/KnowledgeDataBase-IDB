# Zona Térmica

Volumen de aire delimitado por superficies donde EnergyPlus puede resolver la temperatura interior. Es el concepto fundamental para plantear cualquier simulación energética.

## Definición

Una zona térmica es un espacio cerrado (o que se puede modelar como cerrado) donde:
1. Se puede definir el volumen de aire
2. La temperatura interior es diferente a la del exterior
3. El aire al interior se puede considerar a una temperatura uniforme (well-mixed)

## Pregunta clave para identificar una zona térmica

> "¿Si me paro aquí, voy a sentir una temperatura diferente a la del exterior?"

- **Sí** → probablemente es una zona térmica
- **No**, o no puedo delimitar el volumen → no es zona térmica
- **A veces sí, a veces no** → evaluar si la mayor parte del tiempo se comporta diferente

## Ejemplos

| Espacio | ¿Zona térmica? | Razón |
|---------|----------------|-------|
| Cuarto cerrado | Sí | Volumen definido, temperatura diferente al exterior |
| Pasillo abierto | No | Misma temperatura que el exterior |
| Cubo de escaleras | Generalmente no | Difícil delimitar volumen, temperatura similar al exterior |
| Cafetería cerrada | Posiblemente | Si tiene poca ventilación y se siente diferente al exterior |
| Cafetería muy ventilada | No | Se comporta como el exterior |

## Importancia del volumen

Sin volumen definido no se puede hacer el balance de masa: la misma cantidad de energía impacta más a un volumen pequeño que a uno grande. Si no se puede definir el volumen, no se puede calcular cómo sube y baja la temperatura.

## En EnergyPlus

- Cada zona térmica se delimita por superficies (muros, techo, piso)
- Las superficies tienen sistemas constructivos y condiciones de frontera
- EnergyPlus resuelve el balance de energía y masa para cada zona
- Se puede tener múltiples zonas térmicas conectadas entre sí

## Espacios vs Zonas térmicas en Open Studio

- **Espacio** = concepto de Open Studio (volumen geométrico, puede tener tipo de uso). No existe en EnergyPlus.
- **Zona térmica** = concepto de EnergyPlus (donde se resuelve la temperatura).
- En casos simples: un espacio → una zona térmica.
- Los nombres deben ser diferentes y descriptivos (Norte, Cocina, no ThermalZone1).
- No usar espacios en nombres de zonas (causa problemas en Python).

## Zonas con diferentes alturas

En FloorSpaceJS, cada Space puede tener una altura floor-to-ceiling independiente. Si dos espacios con diferente altura comparten una pared, FloorSpaceJS corta automáticamente la superficie: la parte inferior queda con condición Surface/Interzone, la parte superior queda como Outdoor.

## Aparece en

- [[002-ConceptosBasicosBalancesCalor]] — Definición y ejemplos detallados
- [[003-MiPrimeraSimulacion]] — Distinción espacio/zona térmica, creación práctica en Open Studio
- [[001-IntroduccionTallerIDB]] — Mencionada como concepto básico
- [[006-DosZonasTermicasVentanasAleros]] — Zonas con diferentes alturas, intersección automática de superficies

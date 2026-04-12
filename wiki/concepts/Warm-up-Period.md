# Warm-up Period

Proceso iterativo que usa EnergyPlus para eliminar la influencia de la condición inicial arbitraria (23°C) antes de iniciar la simulación real.

## Problema

EnergyPlus inicializa todas las zonas térmicas y temperaturas de materiales a **23°C**. Este valor no corresponde al equilibrio real del edificio con su clima.

## Mecanismo

1. Toma el **primer día** del periodo de simulación (por defecto, 1 de enero)
2. Simula el día completo partiendo de 23°C
3. Registra la temperatura al final del día
4. **Repite el mismo día** usando la temperatura final como nueva condición inicial
5. Continúa hasta que la diferencia entre iteraciones sea menor al criterio de convergencia (~0.1°C)
6. Resultado: **estado oscilatorio permanente** — la solución ya no depende de la condición inicial

## Factores que afectan la convergencia

| Factor | Efecto |
|--------|--------|
| Masa térmica alta | Más iteraciones (tarda más en equilibrar) |
| Clima severo (muy frío o caliente) | Más iteraciones (mayor distancia desde 23°C) |
| Edificación pequeña/liviana | Converge rápido (~3 días) |

## Implicación para comparar simulaciones

- Todas las simulaciones que se quieran comparar deben **empezar en la misma fecha**
- La edificación "recuerda" el clima de días anteriores por efecto de [[Masa-Termica]]
- El warm-up está atado al primer día de simulación — cambiar la fecha de inicio cambia el resultado

## Aparece en

- [[004-InterpretandoMensajesConstructionSets]] — Explicación completa con gráfica conceptual de convergencia

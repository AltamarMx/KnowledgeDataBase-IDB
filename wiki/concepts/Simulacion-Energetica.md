# Simulación Energética

Modelado numérico del comportamiento térmico y energético de una edificación a lo largo del tiempo. Resuelve balances de energía y masa a través de la envolvente arquitectónica, considerando el clima exterior como forzante.

## Ideas clave

- Energy Plus resuelve el problema de **transferencia de calor dependiente del tiempo** a través de la envolvente
- Se requieren dos archivos: un **IDF** (modelo de la edificación) y un **EPW** (datos climáticos)
- Los mecanismos de transferencia modelados incluyen: conducción, convección y radiación
- El resultado permite evaluar temperatura interior, confort térmico y consumo de energía
- La documentación técnica está en el Engineering Reference de Energy Plus (~1,800 páginas)

## Métodos numéricos

EnergyPlus resuelve la ecuación de calor en **1 dimensión** (perpendicular a cada superficie), dependiente del tiempo:

| Método | Tipo | Ventaja |
|--------|------|---------|
| **Conduction Transfer Function (CTF)** | Semi-analítico (funciones de transferencia) | Solución instantánea, muy rápida. Método por defecto |
| **Diferencias Finitas** | Numérico (discretización) | Más intuitivo, necesario para materiales con propiedades variables |

**Distinción crítica:** una "simulación dinámica" que solo usa U o R con clima variable NO es dependiente del tiempo — no considera masa térmica. EnergyPlus sí resuelve el modelo completo. Las NOM-008 y NOM-020 de México usan el modelo independiente del tiempo.

## Simplificaciones en este curso

- Sin ventilación natural
- Sin cargas térmicas internas (personas, equipos)
- Geometrías simples (cubos)
- Piso adiabático

Estas simplificaciones hacen que el resultado **no represente una edificación real**, pero el orden de efectividad de las estrategias se conserva.

## Fase de análisis de resultados

Hacer la simulación es "la parte fácil". El análisis de datos es obligatorio, especialmente para edificaciones sin aire acondicionado donde el objetivo no es minimizar energía sino evaluar confort térmico a lo largo del tiempo. El flujo incluye:
1. Configurar variables de salida (measures en Open Studio)
2. Cargar resultados desde el SQL con Python (ear_tools)
3. Verificar sistemas constructivos
4. Graficar series temporales (temperaturas, radiación)
5. Evaluar confort con modelos adaptativos

## Aparece en

- [[001-IntroduccionTallerIDB]] — Introducción al concepto y al ecosistema de simulación
- [[002-ConceptosBasicosBalancesCalor]] — Métodos numéricos, ecuaciones y restricciones (1D, superficies planas)
- [[005-AnalisisSimulacionesPython]] — Flujo completo de análisis de resultados

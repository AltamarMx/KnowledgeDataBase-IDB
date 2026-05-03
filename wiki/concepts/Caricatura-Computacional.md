---
title: Caricatura Computacional
type: concepto
tags: [concepto, metodologia, modelado, simulacion]
aliases: [caricatura, modelo simplificado, principio de modelado]
clases: [002, 003, 006, 007, 008, 009, 010, 011]
updated: 2026-05-02
---

# Caricatura Computacional

## Definición

Principio metodológico que el profesor usa para encuadrar **qué es** una simulación de Energy Plus:

> "Energy Plus es una caricatura de la vida real, pero una buena caricatura. Hay que tener muy claras las limitaciones."

Una **caricatura** preserva la física relevante para el problema y descarta el resto. La calidad del análisis depende de **saber qué se está descartando** y **por qué es defendible** descartarlo en este caso.

## Caricaturas que hace Energy Plus (catálogo)

| Caricatura | Realidad | Implicación para el modelo |
|------------|----------|----------------------------|
| Flujo de calor **1D perpendicular** a cada superficie | Hay flujo lateral, esquinas y puentes térmicos | No se capturan puentes térmicos a menos que se subdivida la superficie |
| **Material homogéneo** en cada capa de un sistema constructivo | Capas heterogéneas (trabes, refuerzos, vanos) | Se subdivide o se compensa con [[Masa-Termica]] (`InternalMass`) |
| **Mezclado perfecto** del aire de la zona | Estratificación, plumas, gradientes | Una sola temperatura por zona — ver [[Mezclado-Perfecto]] |
| **Solo líneas rectas y polígonos planos** | Geometrías curvas | No hay ventanas circulares, no hay domos — se aproximan con polígonos |
| Radiación interior **distribuida uniformemente** entre superficies | La directa proyectada debería caer en una superficie específica | El piso "se queda" con la directa por default; la difusa siempre — ver [[Radiacion-Interior-Distribuida]] |
| **Coeficiente convectivo correlacional** (no CFD) | Mecánica de fluidos compleja | Correlaciones experimentales por tipo de superficie e inclinación |
| Sombras **no obstruyen el viento** | Una pared exterior frena el viento | E+ no resuelve fluidodinámica externa — la sombra solo afecta radiación |
| **Aleros sin temperatura ni conducción** | Aleros se calientan al sol y conducen al muro | Aleros solo bloquean radiación; no participan en transferencia de calor — ver [[Superficies-de-Sombramiento]] |
| **Marco de ventana ignorado** (en el taller) | Marco metálico genera puente térmico | El "área de ventana" del modelo incluye marco como si fuera cristal — ver [[Ventanas]] |
| **Alero del mismo ancho que la ventana** (Open Studio) | Aleros reales se extienden lateralmente | Limitación de la GUI; sol oblicuo no queda cubierto sin el workaround manual |
| **Rayo de luz que pega a una persona** | El sol entrando por una ventana calienta directamente al ocupante (efecto local) | E+ distribuye uniformemente — el ocupante no se modela; usar T operativa o sensores virtuales para análisis fino |
| **Sunlit Fraction multiplicativa** | El sombreamiento debería atenuar la radiación incidente reportada | E+ aplica el sombreamiento al hacer el balance, no a la variable de output → la radiación incidente sobre ventanas se reporta sin sombreamiento. Ver [[Sunlit-Fraction]] |
| **Sombreamiento solo bloquea directa** | Una protección reduce también difusa y reflejada | E+ atenúa difusa/reflejada via factores de vista (parcialmente) — no cero. En días nublados la protección casi no atenúa |
| **HVAC Ideal Air Loads** | Los AC reales tienen capacidad limitada y eficiencia <100% | E+ Ideal Air Loads: eficiencia 100%, sin capacidad pico, sin ductos. Suficiente para evaluar efecto bioclimático relativo. Ver [[Aire-Acondicionado-Ideal]] |
| **EnerHabitat — una pared sola** | Edificios tienen 4+ superficies + ventanas + ventilación | EnerHabitat resuelve **un solo muro** con cuarto idealizado de 2.5 m y pared adiabática opuesta. Sin LWR interior, sin ventanas, sin ventilación. Útil para **primeras decisiones** comparativas. Ver [[../tools/EnerHabitat]] |
| **Día representativo / oscilatorio permanente** | Cada día real es distinto (clima cambia continuamente) | EnerHabitat usa un **día promedio del mes** repetido hasta convergencia. Suficiente para análisis comparativo, no para análisis energético anual. Ver [[Estado-Oscilatorio-Permanente]] |
| **Temperatura sol-aire** | El balance exterior tiene 3 componentes (radiación corta + convección + LWR) | EnerHabitat los encapsula en una T equivalente $T_{sa}$ y aplica un solo coeficiente convectivo equivalente. Ver [[Temperatura-Sol-Aire]] |
| **Cuarto idealizado de 2.5 m** | Cuartos reales tienen 4 muros + ventanas + ventilación + ocupación | EnerHabitat asume cuarto ficticio de 2.5 m con pared opuesta adiabática. 200 elementos de discretización en la pared, paso de 600 s, h_c exterior=13 W/m²K (NOM-008/020), h_c interior=8.6 W/m²K |

## Caricaturas que hace el curso (sobre las del programa)

Además de lo que impone E+, en el taller se simplifica:

- **Sin ventilación natural ni mecánica.** No se resuelve renovación de aire (Airflow Network).
- **Sin cargas internas** (personas, equipos, iluminación).
- **Piso adiabático.** Se evita modelar la temperatura del ground.
- **Geometrías cúbicas** simples; sin volumetrías complejas.

Ver también [[../classes/001-IntroduccionTallerIDB]].

## Por qué la caricatura sirve

Para evaluar **estrategias de diseño bioclimático** lo importante no es la temperatura absoluta sino el **orden relativo**: ¿qué estrategia mejora más el confort? Una caricatura bien construida preserva ese orden:

> "Si yo digo 'esta es la mejor estrategia' (en la simulación), en la vida real se va a seguir manteniendo. Tal vez las proporciones no, pero el orden sí."

## Cuándo la caricatura traiciona

- Cuando se quieren **valores absolutos** (temperatura exacta, demanda de AC en kWh con precisión).
- Cuando entra una variable que se descartó (ventilación natural cambia mucho los resultados; cargas internas también).
- Cuando se compara con mediciones experimentales sin replicar las condiciones.

## Narrativa computacional asociada

El principio de caricatura se acompaña de una **narrativa computacional**: cada decisión de simplificación se documenta en versiones consecutivas del archivo OSM (`001_volumetria.osm`, `002_dosZonas.osm`, …) → ver [[../procedures/Estructura-Proyecto-Simulacion]]. Cuando algo falla, el regreso a una versión anterior es práctica de desapego: no aferrarse a una versión rota.

## Validación de caricaturas — anécdota Paloma

Una alumna del grupo IER (Paloma) comparó dos simulaciones de la misma protección solar compleja: (1) cada listón dibujado en SketchUp; (2) área equivalente con transmitancia agregada. Diferencia entre las dos: **<2%** en radiación recibida.

Conclusión: **una caricatura bien construida preserva el orden y magnitud del efecto**. Justifica usar simplificaciones agresivas en estudios paramétricos sin perder validez de las comparaciones relativas.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — primera mención del principio
- [[../classes/003-MiPrimeraSimulacion]] — refuerzo y aplicación al primer modelo
- [[../classes/006-DosZonasTermicasVentanasAleros]] — caricaturas nuevas: aleros sin transferencia de calor, marcos ignorados, alero del mismo ancho que la ventana
- [[../classes/007-CasoBaseAleros]] — caricatura del rayo de luz local; anécdota Paloma de validación
- [[../classes/008-ShadingVentanas]] — Sunlit Fraction multiplicativa; sombreamiento solo bloquea directa
- [[../classes/009-AireAcondicionadoSetPoints]] — Ideal Air Loads como caricatura de HVAC
- [[../classes/010-EnerHabitatParte1]] — caricaturas de EnerHabitat (un solo muro, día representativo, T sol-aire)
- [[../classes/011-EnerHabitatParte2]] — detalles del cuarto idealizado de 2.5 m con discretización y coeficientes

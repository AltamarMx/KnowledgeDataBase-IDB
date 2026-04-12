# Condiciones de Frontera

Especificaciones matemáticas que definen el comportamiento en los límites del modelo de simulación. Determinan cómo interactúa cada superficie de la envolvente con su entorno.

## Tipos principales

1. **De temperatura** — se especifica una temperatura en el borde (ej. temperatura del suelo, temperatura de un espacio adyacente)
2. **De flujo de calor** — se especifica el flujo de calor a través del borde:
   - Flujo constante
   - Flujo variable
   - Flujo cero = **condición adiabática** (no hay transferencia de calor)
3. **Convectiva** — se especifica un coeficiente de transferencia por convección

## En EnergyPlus

La condición de frontera exterior se expresa como el [[Balance-de-Calor]]:

> I_s · α + q_LWR + q_conv = -k · ∂T/∂x |_{x=0}

Donde x=0 es la superficie exterior. Esta ecuación acopla la radiación solar, radiación de onda larga (ground, cielo, aire, alrededores) y convección con el flujo conductivo que entra al muro.

## En el curso

- El **piso se modela como adiabático** (simplificación) porque determinar la temperatura del suelo depende del clima, material, humedad y tipo de terreno
- Las superficies exteriores se conectan con el clima (archivo EPW)
- Open Studio permite revisar visualmente las condiciones de frontera de cada superficie

## Visualización en Open Studio

En el 3D View → **Render by Boundary**:

| Color | Condición | Significado |
|-------|-----------|-------------|
| Azul | Outdoor | Expuesta a sol y viento |
| Verde | Surface/Interzone | Conectada a zona adyacente |
| Rojo | Adiabatic | Flujo de calor = 0 |
| Otro | Ground | Temperatura del suelo |

## Combinaciones con Sun/Wind Exposure

En EnergyPlus, una superficie con condición **Outdoor** puede tener desactivada la exposición al sol y/o al viento:

| Caso | Sol | Viento | Ejemplo |
|------|-----|--------|---------|
| Outdoor completo | Sí | Sí | Fachada normal |
| Outdoor sin sol | No | Sí | Superficie sombreada permanentemente por edificio adyacente |
| Outdoor sin sol ni viento | No | No | Estacionamiento subterráneo (tiene aire pero no sol ni viento) |

**Aplicación más común:** pisos sobre estacionamiento subterráneo — se quita exposición al sol manteniendo convección con aire exterior.

## Condición Other Side Coefficients

Permite definir una **temperatura constante** en la cara exterior. Útil para casos especiales no cubiertos por las condiciones estándar.

## Aparece en

- [[001-IntroduccionTallerIDB]] — Explicación de los tipos y simplificaciones usadas
- [[002-ConceptosBasicosBalancesCalor]] — Ecuación del balance exterior como condición de frontera
- [[003-MiPrimeraSimulacion]] — Representación visual con colores, aplicaciones prácticas de condiciones adiabáticas
- [[004-InterpretandoMensajesConstructionSets]] — Combinaciones sun/wind exposure, Other Side Coefficients, estacionamiento subterráneo
- [[006-DosZonasTermicasVentanasAleros]] — Intersección automática con diferentes alturas, limpieza de geometría

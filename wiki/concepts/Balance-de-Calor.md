---
title: Balance de Calor
type: concepto
tags: [concepto, fisica, transferencia-calor, balance]
clases: [001, 002, 003]
updated: 2026-05-02
---

# Balance de Calor

## Definición

Aplicación del **principio de conservación de la energía** a un volumen (una zona térmica, una superficie, una capa de material). En cualquier instante, el calor que entra menos el que sale es igual al cambio en energía almacenada del volumen. Cuando se acopla con la conservación de masa, se llama **balance de energía y masa**.

En simulación de edificaciones, lo que resuelve [[../tools/EnergyPlus]] en cada paso de tiempo es el **balance de calor dependiente del tiempo** a través de la **envolvente arquitectónica** ([[Envolvente-Arquitectonica]]).

## Modelo dependiente del tiempo

La ecuación 1D de transferencia de calor en un material homogéneo:

$$
\rho \, c_p \, \frac{\partial T}{\partial t} = k \, \frac{\partial^2 T}{\partial x^2}
$$

donde ρ es densidad, $c_p$ calor específico, k conductividad térmica. El término $\partial T/\partial t$ es lo que **distingue** un modelo dependiente del tiempo de uno independiente.

Energy Plus resuelve esta ecuación **solo en la dirección perpendicular a la superficie** (1D) — ver restricciones en [[../tools/EnergyPlus]].

### Por qué importa el modelo dependiente del tiempo

Sin el término temporal, el modelo se reduce a la **resistencia térmica** ($U = 1/R$) y **desprecia la masa térmica**: la temperatura interior responde instantáneamente al exterior. Esto:

- Subestima el papel del [[Masa-Termica]] (atenuación y desfase).
- Es inadecuado para climas con oscilación diurna fuerte (la mayoría de México).

> **Red flag terminológica:** "simulación dinámica" es ambiguo — algunos lo usan para el modelo dependiente del tiempo, otros para U variable en el tiempo. Decir explícitamente: **"modelo de transferencia de calor dependiente del tiempo"**.

> **Crítica del profesor:** las **NOM-008** y **NOM-020** mexicanas usan modelos basados en U → no son adecuadas para evaluar diseño bioclimático real.

## Balance de calor en superficie exterior (opaca)

Para una superficie expuesta al exterior, en la frontera $x=0$:

$$
\underbrace{q''_{\alpha sol}}_{\text{onda corta}} + \underbrace{q''_{LWR}}_{\text{onda larga}} + \underbrace{q''_{conv}}_{\text{convección}} = -k \, \frac{\partial T}{\partial x}\bigg|_{x=0}
$$

Es la condición de frontera que alimenta la solución de la ecuación 1D dependiente del tiempo a través del sistema constructivo.

### Componente 1 — Radiación de onda corta

$$
q''_{\alpha sol} = \alpha \cdot (I_{directa,\perp} + I_{difusa})
$$

donde α es la [[Absortancia-Solar]]. La radiación directa se proyecta sobre la superficie según trayectoria solar aparente — Energy Plus se encarga de los cálculos geométricos y aplica sombreamientos si los hay.

### Componente 2 — Radiación de onda larga

$$
q''_{LWR} = q''_{ground} + q''_{sky} + q''_{air} + q''_{surroundings}
$$

Cada sub-componente sigue Stefan-Boltzmann:

$$
q''_{s \to i} = \varepsilon \, \sigma \, F_{s \to i} \, (T_i^4 - T_s^4)
$$

con ε = [[Emisividad]], σ = constante de Stefan-Boltzmann, $F_{s \to i}$ = [[Factor-de-Vista]]. Detalle de las cuatro fuentes en [[Radiacion-Onda-Larga]].

> El intercambio con el cielo (temperatura efectiva muy baja) es un **enfriador potente** — la onda larga puede ser el 60-70% del calor total en un muro.

**Linealización con coeficiente HR:**

$$
q''_{s \to i} = h_{r,i} \, (T_i - T_s)
$$

Más práctico para acoplar numéricamente con el término convectivo.

### Componente 3 — Convección

$$
q''_{conv} = h_c \, (T_{aire} - T_s)
$$

El coeficiente $h_c$ depende de:

- Inclinación de la superficie
- ΔT entre aire y superficie
- Rugosidad del material
- Velocidad del viento

Energy Plus usa **correlaciones experimentales** (no resuelve mecánica de fluidos). Hay correlaciones más adecuadas para ciertos climas — el curso usa los defaults excepto cuando se compara contra una norma que fija valores específicos.

## Balance de calor en superficie interior (opaca)

Pieza paralela al balance exterior, con tres componentes en la cara $x=L$:

$$
q''_{conv,i} + q''_{LWR,i} + q''_{SW,i} = -k \frac{\partial T}{\partial x}\bigg|_{x=L}
$$

### Componente 1 — Convección con el aire interior

$$
q''_{conv,i} = h_{c,i} \, (T_s - T_I)
$$

donde $T_I$ es la **temperatura del aire indoor** de la zona (la incógnita principal del problema). $h_{c,i}$ depende del [[Tipos-Superficie|tipo de superficie]] (Wall/Roof/Floor) y de la inclinación; un techo no tiene el mismo coeficiente que un piso.

### Componente 2 — Radiación de onda larga interior

A diferencia del exterior, la onda larga interior se intercambia **solo entre las superficies del cuarto**. La radiación de onda larga **no atraviesa el vidrio** — aunque haya ventana en el muro, las superficies internas no "ven" el cielo por LWR.

$$
q''_{LWR,i} = \sum_j \varepsilon_j \, \sigma \, F_{s \to j} \, (T_j^4 - T_s^4)
$$

con la suma sobre todas las superficies internas que ven a `s` ([[Factor-de-Vista]] = 0 para superficies paralelas que están detrás).

### Componente 3 — Radiación de onda corta interior

Viene de fuentes que emiten luz visible:

- Radiación solar directa y difusa que entra por ventanas.
- Luminarias y proyectores.

E+ aplica una caricatura: la **directa se asume al piso** por default; la **difusa se reparte uniformemente** entre todas las superficies. Detalle en [[Radiacion-Interior-Distribuida]].

## Balance de calor en zona térmica (balance de aire)

Para una [[Zona-Termica]], el balance de aire combina todos los flujos de calor que cruzan sus fronteras y los aplica a un volumen de aire que se asume **bien mezclado** ([[Mezclado-Perfecto]] — toda la zona, una sola temperatura instantánea):

- Convección con cada superficie interior — suma de $h_{c,i}\,A_i\,(T_{s,i} - T_I)$
- Conducción a través de ventanas (módulo Window)
- Radiación que entra por ventanas y se queda en el cuarto
- Cargas internas (personas, equipos, iluminación) — **no se usan en este curso**
- Intercambio de masa por ventilación / infiltración — entrada de aire con su propia temperatura y humedad; conservación de masa obliga a tratar también la humedad. **No se usa en este curso**
- Aire acondicionado (HVAC) — opcional, según objetivo

Resolver este balance produce $T_I$ — la temperatura interior, finalidad del modelo.

## Mecanismos de transferencia involucrados

- **Conducción** — a través de los materiales que componen un sistema constructivo ([[Sistemas-Constructivos]]).
- **Convección** — entre las superficies y el aire (interior y exterior).
- **Radiación** — solar (onda corta) sobre superficies exteriores; intercambio de onda larga con cielo, ground, air y surroundings; intercambio entre superficies interiores.
- **Cambio de fase** — Energy Plus puede modelarlo (PCM), pero **en este curso no se usa**.

## Por qué es central en este curso

Las estrategias bioclimáticas actúan **modificando términos del balance**:

| Estrategia | Término modificado |
|------------|--------------------|
| Color claro | α (absortancia solar) ↓ |
| Alero | $I_{directa,\perp}$ sobre ventana ↓ |
| Masa térmica | ρ·$c_p$·V ↑ → atenuación y desfase |
| Ventana doble | U de la ventana ↓ |
| Aislamiento | k del sistema constructivo ↓ |

Cuantificar el impacto = resolver el balance con y sin la estrategia y comparar.

## Forzante externo

El balance depende del clima codificado en el **archivo EPW** (ver [[TMY]]).

## Condiciones de frontera

El balance se resuelve sujeto a las [[Condiciones-de-Frontera]] de cada superficie: temperatura impuesta, flujo de calor (constante, variable o cero ⇒ adiabática), o condición convectiva combinada con radiación.

## Documentación de referencia

- **Engineering Reference** de Energy Plus — ecuaciones, correlaciones y métodos numéricos.
- **Input Output Reference** — cómo se especifican objetos.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción conceptual
- [[../classes/002-ConceptosBasicosBalancesCalor]] — ecuación 1D dependiente del tiempo, balance en superficie exterior con sus tres componentes
- [[../classes/003-MiPrimeraSimulacion]] — balance en superficie interior, balance de aire con mezclado perfecto

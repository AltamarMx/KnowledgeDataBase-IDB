---
title: 002 — Conceptos Básicos y Balances de Calor
type: clase
clase: 002
profesor: Guillermo Barrios del Valle
fuente: raw/videos/002_ConceptosBasicos.md
fecha_ingesta: 2026-05-02
tags: [clase, balance-calor, zona-termica, energyplus, epw, tmy]
aliases: [Clase 002]
---

# 002 — Conceptos Básicos y Balances de Calor

## Metadatos

- **Clase:** 002
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/002_ConceptosBasicos.md`
- **Tipo:** Clase teórica con introducción a las ecuaciones del balance de calor

## Resumen

Primera clase técnica del taller. Se define el concepto central de **zona térmica**, se contrasta el modelo **dependiente del tiempo** (lo que hace Energy Plus) contra el modelo **independiente del tiempo** (que solo usa la U y desprecia masa térmica), se recorren los **módulos de Energy Plus** que resuelven cada parte del problema (conducción, ventanas, sombreamiento, cielo, balance de aire), se enuncian las **restricciones fundamentales** del programa (flujo 1D perpendicular, solo líneas rectas) y se escribe el **balance de calor en la superficie exterior** con sus tres componentes — radiación de onda corta, radiación de onda larga, convección — igualados a la conducción.

Cierra con una introducción al **archivo EPW** y al concepto de **TMY** (Typical Meteorological Year). Anuncia que en el proyecto final cada equipo escogerá una ciudad y deberá revisar su EPW.

## Objetivos de aprendizaje

- Entender qué es una [[../concepts/Zona-Termica]] y cómo decidir si un espacio se modela como tal.
- Distinguir un modelo de transferencia de calor **dependiente del tiempo** de uno independiente, y por qué importa.
- Tener un mapa de los módulos de [[../tools/EnergyPlus]] y qué resuelve cada uno.
- Conocer las **restricciones de modelado** que impone Energy Plus.
- Plantear el balance de calor en la superficie exterior de una pared.
- Entender qué es un EPW y un TMY.

## Conceptos centrales

### Zona térmica

Volumen de aire delimitado por superficies, donde Energy Plus resuelve la temperatura del aire interior. Sin volumen no hay balance de masa, sin balance de masa no hay variación de temperatura.

**Heurística para decidir si un espacio es zona térmica:**

> ¿En este espacio se siente una temperatura diferente a la del exterior? Si la respuesta es **siempre sí**, es zona térmica. Si es **a veces sí, a veces no**, podría ser. Si es **siempre no**, no es zona térmica.

Ejemplos del instituto:
- Pasillos abiertos del edificio nuevo → **no** son zona térmica (siempre la temperatura del aire exterior).
- Cafetería actual (muy ventilada) → difícil definirla como zona térmica.
- Cafetería anterior (más cerrada, con cocina) → sí era zona térmica diferenciada.

Detalle en [[../concepts/Zona-Termica]].

### Modelo dependiente del tiempo (vs. independiente)

Energy Plus resuelve la **transferencia de calor dependiente del tiempo**: la ecuación tiene el término ∂T/∂t y considera la **masa térmica** (densidad × calor específico × espesor) además de la conductividad.

Modelos **independientes del tiempo** usan solo la **U** (inverso de la resistencia térmica). Como no hay derivada temporal, no consideran masa térmica → son inadecuados para entender la dinámica de una edificación en climas con oscilación día/noche.

> **Red flag terminológica:** "simulación dinámica" es ambiguo — algunos lo usan para el modelo dependiente del tiempo, otros para un modelo con U pero alimentado por temperatura ambiente y radiación variable. El profesor recomienda decir explícitamente **"modelo de transferencia de calor dependiente del tiempo"**.

> **Crítica a las normas mexicanas:** las **NOM-008** y **NOM-020** usan modelos con U (independientes del tiempo) → no son adecuadas para evaluar diseño bioclimático en México.

Una excepción donde el modelo independiente del tiempo es válido: **ventanas** (masa despreciable).

### Time steps

Energy Plus resuelve por intervalos de tiempo:

- Mínimo simulable: **un día**.
- Resolución típica: **horaria**.
- Resolución máxima: **cada minuto** → ~525,600 pasos/año por variable.

Cada variable solicitada al output genera una serie temporal de ese tamaño.

### Módulos de Energy Plus (panorama)

Energy Plus está compuesto por módulos que resuelven cada parte del problema. Detalle de cada módulo en [[../tools/EnergyPlus]]:

| Módulo | Qué resuelve | Uso en el curso |
|--------|--------------|-----------------|
| **Conduction Transfer Function (CTF)** | Conducción dependiente del tiempo a través de muros — solución semi-analítica con funciones de transferencia | Default — se usa |
| **Diferencias finitas** | Alternativa a CTF — se necesita para materiales con cambio de fase o conductividad variable | No se usa |
| **Window glass** | Transferencia de calor a través de ventanas (modelos sencillos y complejos, marcos, capas múltiples, intercambio radiativo onda corta y larga) | Se usa |
| **Shading** | Cálculo de obstrucciones sobre ventanas y superficies (aleros horizontales y verticales) | Se usa |
| **Sky** | Modelo de cielo — discretización de la semiesfera en parches (uno de ellos brilla = el sol, los demás difusos) | Default — se usa |
| **Day lighting** | Iluminación natural | No se usa (Radiance es mejor para esto) |
| **Air heat balance / mass balance** | Balance de energía y masa en el aire de la zona térmica | Se usa |
| **Airflow Network** | Modelo más complejo de E+ para ventilación natural (incluso con velocidad de viento cero, por diferencia de densidad) | **No se usa en el curso** — demasiado complejo |

### Restricciones fundamentales de Energy Plus

Estas restricciones limitan qué se puede modelar y obligan a hacer "caricaturas":

1. **Flujo de calor 1D perpendicular a la superficie.** No hay flujo lateral entre superficies adyacentes (la conducción esquina-a-esquina se desprecia; el acoplamiento entre muros pasa por convección con el aire interior). Implicación: los puentes térmicos en cambios de material no se capturan bien — son las "vanos".
2. **Solo líneas rectas.** No hay superficies curvas, no hay ventanas circulares. Razón: el [[../concepts/Factor-de-Vista]] de una superficie consigo misma se asume cero, lo que solo es cierto para superficies planas. Resolver curvas requeriría coordenadas cilíndricas.
3. **Material homogéneo en cada superficie.** Si necesito modelar trabes empotradas en un techo, debo:
   - Subdividir la superficie en sub-superficies con sistemas constructivos distintos, o
   - Usar el objeto **masa térmica** para sumar la inercia perdida.

> **Principio de modelado:** las simulaciones son **caricaturas**. Una buena caricatura preserva la física relevante; una mala caricatura colapsa lo importante.

## Balance de calor en la superficie exterior

Para una superficie opaca expuesta al exterior, Energy Plus iguala el flujo de calor entrante (suma de tres componentes) al flujo conducido hacia el interior:

$$
\underbrace{q''_{\alpha sol}}_{\text{onda corta}} + \underbrace{q''_{LWR}}_{\text{onda larga}} + \underbrace{q''_{conv}}_{\text{convección}} = -k\,\frac{\partial T}{\partial x}\bigg|_{x=0}
$$

**Componente 1 — Radiación de onda corta absorbida:**

$$
q''_{\alpha sol} = \alpha \cdot (I_{directa,\perp} + I_{difusa})
$$

donde α es la [[../concepts/Absortancia-Solar]] (típicamente 0.2-0.3 para colores claros, ~0.8 para oscuros). Energy Plus se encarga de proyectar la radiación directa sobre la superficie usando trayectoria solar aparente y aplica sombreamiento si lo hay.

**Componente 2 — Radiación de onda larga (4 sub-componentes):**

$$
q''_{LWR} = q''_{ground} + q''_{sky} + q''_{air} + q''_{surroundings}
$$

Cada sub-componente sigue la forma Stefan-Boltzmann:

$$
q''_i = \varepsilon \, \sigma \, F_{s\to i} \, (T_i^4 - T_s^4)
$$

donde ε es la [[../concepts/Emisividad]], σ la constante de Stefan-Boltzmann, F el [[../concepts/Factor-de-Vista]] entre la superficie y la fuente i ∈ {ground, sky, air, surroundings}.

> **Insight clave:** el cielo tiene temperatura cercana al cero absoluto → **enfría** las superficies de noche por intercambio radiativo. La onda larga puede ser el **60-70%** del calor en un muro — no es despreciable como a veces se enseña.

**Linealización con coeficiente HR:**

Se puede reescribir cada término radiativo en forma "tipo convección":

$$
q''_i = h_{r,i} \, (T_i - T_s)
$$

donde h_{r,i} se calcula a partir de las temperaturas. Esto facilita el acoplamiento numérico con el término convectivo.

**Componente 3 — Convección:**

$$
q''_{conv} = h_c \, (T_{aire} - T_s)
$$

El coeficiente convectivo h_c depende de:

- **Inclinación** de la superficie (vertical, horizontal, intermedia)
- **Diferencia de temperatura** ΔT
- **Rugosidad** del material (acero/vidrio: lisos; concreto pulido vs. concreto rugoso: difieren)
- **Velocidad del viento**

Energy Plus tiene **correlaciones experimentales** para distintos casos. **No** resuelve mecánica de fluidos. Hay correlaciones más adecuadas para ciertos climas y otras menos. En el curso se usan los defaults; para comparaciones contra normativas (NOM-008, NOM-020) se fijan los coeficientes que la norma especifica.

## Caricaturas y decisiones de modelado

El profesor da varios ejemplos de cómo construir una "caricatura" del salón actual:

- **Versión simple:** 6 superficies (4 muros + piso + techo), todos del mismo sistema constructivo (tabique blanco; piso y techo de concreto). Sin ventanas.
- **Versión intermedia:** dividir el muro grande en tres sub-superficies (trabe — ladrillo — trabe), cada una con su sistema constructivo. O simplificar a dos: trabe + ladrillo.
- **Trampa válida:** sumar dos muros idénticos con la misma orientación y sin radiación incidente en uno de ellos en una sola superficie del doble de largo. Energy Plus no nota la diferencia mientras no haya radiación o sombreamiento que distinga uno del otro.

> **Cuándo es válido sumar/simplificar:** cuando no hay radiación incidente ni sombreamiento que diferencie superficies. Si hay diferencias de carga, hay que conservar la representación.

## Archivo EPW y TMY

El archivo de clima EPW (Energy Plus Weather) tiene los datos horarios o sub-horarios de:

- Temperatura del aire
- Humedad relativa
- Velocidad y dirección del viento
- Radiación global, directa, difusa
- Presión atmosférica
- Lluvia
- Ubicación geográfica

### TMY (Typical Meteorological Year)

Año típico meteorológico — **no es un año promedio**.

Cómo se construye: para cada mes, se busca el año real cuyos datos de ese mes **se parecen más a la distribución de todos los años**. Mide distancias estadísticas (similar a R² en un fit). El año típico resultante puede ser un Frankenstein con enero de 2016, febrero de 2018, marzo de 2016, etc.

**Implicaciones:**

- El TMY **suaviza anomalías**: meses extremos quedan fuera por construcción.
- Por tanto, **pierde gran parte del efecto de cambio climático** (las anomalías cada vez son más frecuentes).
- Los datos provienen típicamente de **reanálisis** (no de estaciones meteorológicas directamente) — validados donde se puede.

Detalle en [[../concepts/TMY]].

### Dónde bajar EPWs

- **OneBuilding** — colección global de archivos EPW. Cada zip viene con varios archivos; uno es el EPW.
- Los nombres incluyen el periodo (ej. TMY-2007-2021, TMY-2009-2023). Son TMYs distintos según el rango.

### Construir un EPW propio

El grupo construye EPWs desde la **estación meteorológica de Temixco** cuando hace experimentos locales. Esto es fácil técnicamente pero pierde el carácter "típico" — un solo año con anomalía puede sobredimensionar el diseño.

Otro recurso interno: Jesús Quiñones (instituto) publicó un **año típico solar** (basado en datos solares, no meteorológico general) en ANES.

## Proyecto final — anuncio

Cada equipo escogerá una **ciudad** donde se evaluará la casa del proyecto.

**Tip estratégico del profesor:**

- **No escojan climas con doble extremo** (Monterrey, Sonora) — implican diseñar para temporada cálida **y** fría. Las soluciones se pisan: lo que mejora una empeora la otra.
- **Climas calurosos** (ej. Temixco) → solo temporada cálida → más manejable.
- **Climas fríos puros** → solo temporada fría.
- "No se dejen llevar por el romanticismo" de escoger su ciudad natal.

Tendrán que verificar que **existe el EPW** del lugar y aprender a inspeccionarlo.

## Conceptos derivados (referencias)

Conceptos nuevos introducidos o profundizados en esta clase:

- [[../concepts/Zona-Termica]] — volumen de aire delimitado por superficies
- [[../concepts/Balance-de-Calor]] — actualizada con ecuación del balance en superficie exterior
- [[../concepts/Factor-de-Vista]] — fracción de radiación que recibe una cara desde otra
- [[../concepts/Absortancia-Solar]] — fracción de radiación solar que absorbe una superficie
- [[../concepts/Emisividad]] — eficiencia radiativa relativa al cuerpo negro
- [[../concepts/Radiacion-Onda-Larga]] — intercambio radiativo con ground, sky, air, surroundings
- [[../concepts/Masa-Termica]] — densidad × calor específico × espesor; razón por la que importa el modelo dependiente del tiempo
- [[../concepts/TMY]] — año típico meteorológico

## Conexiones

- ← **Anterior:** [[001-IntroduccionTallerIDB]] — introducción y herramientas
- → **Siguiente:** _003-MiPrimeraSimulacion_ — empieza a dibujar y simular
- → Reglas (incluye asistente IA del curso): [[../REGLAS_CURSO]]
- → Materia siguiente del plan: Energía en Edificaciones (ahí se ven Diferencias finitas, Airflow Network, masa térmica con detalle, EPW propio con cambio climático)

## Recursos mencionados

- **OneBuilding.org** — repositorio global de archivos EPW.
- **Jesús Quiñones** (Instituto) — año típico solar publicado en ANES.
- **Página de Energy Plus** — para revisar la lista oficial de módulos.
- **NotebookLM** y **Gems de Gemini** — herramientas RAG para construir el asistente IA del curso (ver [[../REGLAS_CURSO]]).

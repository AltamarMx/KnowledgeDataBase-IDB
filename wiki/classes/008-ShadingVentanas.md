---
title: 008 — Shading en Ventanas
type: clase
clase: 008
profesor: Guillermo Barrios del Valle
fuente: raw/videos/008_ShadingEnVentanas.md
fecha_ingesta: 2026-05-02
tags: [clase, shading, ventanas, sunlit-fraction, debugging, energyplus, grados-hora, confort]
aliases: [Clase 008]
---

# 008 — Shading en Ventanas

## Metadatos

- **Clase:** 008
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/008_ShadingEnVentanas.md`
- **Tipo:** Clase resolutiva — depura el bug de la 007 y entrega la metodología de análisis del proyecto final

## Resumen

Clase que **resuelve el bug de la 007** y entrega el descubrimiento técnico central:

> **`Surface Outside Face Incident Solar Radiation Rate per Area` no refleja el sombreamiento sobre sub-superficies** (ventanas). E+ aplica el sombreamiento mediante `Surface Outside Face Sunlit Fraction` al hacer el balance de calor, no al reportar la radiación incidente.

El profesor consulta Claude (que primero alucina una respuesta, luego confiesa haberla inventado, luego la valida con la documentación oficial). Continúa con el **algoritmo de sombreamiento** de E+, una digresión sobre **enfriamiento radiativo al cielo**, y cierra con la **metodología completa para el proyecto final**: grados-hora de disconfort y estructura de libretas.

## Resolución del bug de la 007

### Pasos del debugging del profesor

1. **Verificar el flujo de datos en Python**: ¿la función `carga_df(f)` redefine `f` adentro? ¿estoy cargando dos veces el mismo SQL?
2. **Verificar visualmente el OSM**: las shading surfaces sí están dibujadas.
3. **Inspeccionar el IDF generado**: ¿llegaron las shading surfaces al motor? Sí están.
4. **Inspeccionar la radiación incidente sobre el muro padre** (no la ventana): ahí sí se ve diferencia entre casos. → primera pista de que el problema es la variable, no el modelo.
5. **Consultar la documentación / IA**:
   - Pregunta a Claude — primera respuesta: "es un quirk", explica el comportamiento.
   - Pregunta a Claude por la documentación específica — Claude **confiesa haberlo inventado** ("autocomplaciente").
   - Verificar manualmente en la **Engineering Reference**: la explicación de Claude era esencialmente correcta.

> "Los modelos de lenguaje suelen ser autocomplacientes — me dice 'tienes razón, no lo encontré en ningún lado, yo lo inventé'. Pero eso debería estar en el Engineering Reference, y lo que sucede es lo siguiente."

### El descubrimiento

E+ calcula el balance de calor sobre una ventana así:

$$
q''_{absorbida} = \alpha \left[ I_b \cdot \cos\theta \cdot SF + I_d \cdot F_{s\to sky} + I_g \cdot F_{s\to ground} \right]
$$

donde `SF` es la **Sunlit Fraction** (fracción de la superficie que recibe radiación directa). El sombreamiento entra **multiplicativamente** vía SF para la **directa**, y vía cambios en factores de vista para la difusa y reflejada.

La variable `Surface Outside Face Incident Solar Radiation` reporta la **radiación bruta** ($I_b$, $I_d$, $I_g$ en su forma sin atenuar por SF) sobre **sub-superficies** — por eso en ventanas no muestra el sombreamiento.

Detalle en [[../concepts/Sunlit-Fraction]] y [[../concepts/Algoritmo-Sombreamiento]].

### Cómo se audita correctamente

- Pedir `Surface Outside Face Sunlit Fraction` para la ventana.
- Validar visualmente que cae a 0 cuando el alero protege.
- Calcular radiación efectiva = $I_b \cdot SF + I_d$ (aproximación).

Procedimiento completo en [[../procedures/Auditar-Sombreamiento-Ventanas]].

> "En sub-superficies (ventanas y puertas), la radiación incidente se reporta sin descontar sombras. En muros opacos sí refleja el sombreamiento. Por eso al hacer el experimento sobre el muro padre sí se ve el efecto."

## El algoritmo de sombreamiento de E+

Tres mecanismos:

| Mecanismo | Aplica a | Cómo |
|-----------|----------|------|
| **Sunlit Fraction** | Radiación directa | Multiplicativo: $I_b \cdot SF$ |
| **Factores de vista** modificados | Difusa y reflejada del ground | Atenúan $F_{s\to sky}$ y $F_{s\to ground}$ |
| **Algoritmo de overlapping** | Múltiples sombras superpuestas | Detecta y descuenta área duplicada |

Tres algoritmos de cálculo: **Polygon Clipping** (default), **Pixel Counting**, **Imported**. Detalle en [[../concepts/Algoritmo-Sombreamiento]].

### Solo la directa se atenúa con SF

Implicación importante: una protección con `SF = 0` **no bloquea** el 100% de la radiación. La difusa sigue llegando con factor de vista al cielo no nulo.

> "El Sunlit Fraction me dice la relación de energía o de potencia incidente sobre cualquier instante. Pero no es la radiación total bloqueada — la directa la bloquea, la difusa no."

En **días nublados** (radiación 100% difusa), una protección casi no atenúa. En **días despejados**, la protección con SF baja sí reduce drásticamente la radiación efectiva.

### Warning común "many overlapping shadows"

Inocuo cuando se entiende. Aparece en geometrías complejas con muchos aleros, parteluces y vecinos. Detalle en [[../concepts/Mensajes-EnergyPlus]].

## Reflexión sobre Energy Plus y la confianza

> "10 veces hemos dicho que Energy Plus está mal. Solo en 2 de las 10 realmente lo estaba. En las otras 8 fue que no entendíamos cómo funcionaba o lo estábamos usando mal."

Los dos bugs reales que el grupo IER ha encontrado en E+:

1. **Airflow Network**: comportamiento anómalo encontrado por Miriam durante su doctorado.
2. **Diferencias finitas**: criterio de convergencia mal planteado — al refinar la discretización la solución empeora. Encontrado hace ~15 años por un estudiante. **Sigue ahí** en versiones actuales.

> "Cada vez digo menos que está mal Energy Plus. La mayoría de las veces somos nosotros."

Lección: cuando algo no salga, primero verificar:

1. **Flujo de datos** (Python).
2. **Configuración del modelo** (Open Studio).
3. **Documentación** (Engineering Reference).
4. Solo al final culpar al motor.

## Uso de IA (Claude) para debugging

> "Por supuesto, le pregunto y le planteo el problema. Le digo: ¿en qué parte está documentado eso? Y curiosamente, esto sale algo que tal vez ya hayan leído: los modelos de lenguaje suelen ser autocomplacientes."

Patrón observado:

1. La IA da una respuesta plausible.
2. Si pides la fuente, **a veces confiesa que la inventó** ("tienes razón, no lo encontré en ningún lado").
3. Pero la respuesta inventada es a menudo **direccionalmente correcta** — sirve como hipótesis para verificar en la documentación.

Recomendación: **IA como first pass**, siempre verificar con documentación oficial antes de actuar. Especialmente para Energy Plus, que tiene mucha física específica que es fácil de inventar plausiblemente.

> "Tengo la documentación, tengo una interpretación, tengo el modelo de lenguaje que admite que miente y que me autocomplace. Entonces ahora yo tengo que construir la historia verdadera."

## Asimetría observada — pista física, no bug

Al graficar la Sunlit Fraction de una ventana norte, el profesor nota **asimetría temporal**: la mañana no es espejo de la tarde. Hipótesis:

- En el solsticio de verano cerca del trópico, el sol pasa al **norte del cenit** unos días → la ventana norte recibe directa.
- Si hay un **parteluz solo en un lado** (no simétrico), la Sunlit Fraction es asimétrica.

Lección: **antes de declarar un bug, verificar si la geometría es realmente simétrica**. Si no lo es, esperar asimetría.

> "¿Tienes la certeza? Diseña un experimento simétrico. Tú deberías ver SF simétrica."

## Digresión — enfriamiento radiativo al cielo

Pregunta de la clase: ¿una edificación puede estar más fría que la T del aire exterior? Respuesta: **sí, y no se viola la termodinámica**.

Mecanismo: **intercambio LWR con el cielo** (T efectiva ~−15 °C). Una superficie con factor de vista al cielo y baja inercia térmica (lámina metálica) puede caer 3-5 °C debajo de T del aire en una noche despejada y seca.

Aplicación moderna: **películas selectivas** que reflejan el solar y emiten en la ventana atmosférica de 8-13 μm → enfriamiento pasivo incluso de día. Patente reciente (~2017, Stanford et al.).

> "Si tú le pasas un fluido, lo voy a enfriar — porque está viendo al cielo y el cielo está a una temperatura menor."

Detalle en [[../concepts/Enfriamiento-Radiativo-Cielo]].

## Filosofía — método experimental y autores recomendados

El profesor cita dos figuras como inspiración del método experimental:

| Persona | Campo | Por qué |
|---------|-------|---------|
| **Steven Brunton** | Mecánica de fluidos / dynamical systems (UWashington) | Plática rigurosa, científico de carrera |
| **James Hoffman** | Café especialty (YouTube) | Mercadotecnista que aplica método científico a café — ejemplo de transferencia de método |

Anécdota Hoffman: experimento sobre cómo la altura de vertido afecta el sabor del café. Pone termopares, varía altura, mide perturbación del lecho. "Es un cuate mercado nuevo, pero plantea experimentos genial."

Lección: la habilidad para **plantear experimentos controlados** que aíslen variables es transferible a cualquier campo, y es lo que separa la opinión del conocimiento.

## Métricas para el proyecto final — grados-hora de disconfort

> "No va a haber una sola métrica que les dé todo. Si saco promedio, pierdo la oscilación. Si saco amplitud, pierdo el centro. Si saco solo máximos, pierdo el mínimo."

La métrica recomendada: **grados-hora de disconfort**, separados en cálido y frío.

$$
GH_{cálido} = \sum_t \max\big(T_{op}(t) - T_{conf,sup}(t),\ 0\big) \cdot \Delta t
$$

$$
GH_{frío} = \sum_t \max\big(T_{conf,inf}(t) - T_{op}(t),\ 0\big) \cdot \Delta t
$$

Ventajas vs otras métricas:

- Combina **tiempo + magnitud**.
- Distingue tipo de disconfort (cálido vs frío) — un caso puede mejorar uno y empeorar el otro.
- Comparable entre simulaciones como **diferencia relativa**.

Esquema del concepto en [[../concepts/Grados-Hora-Disconfort]] (con ASCII art del cálculo).

### Tabla de reporte del proyecto final

| Caso | GH cálido | GH frío | ΔGH cálido vs base | ΔGH frío vs base |
|------|-----------|---------|---------------------|--------------------|
| Caso base | 8,500 | 200 | — | — |
| + Estrategia 1 | 6,200 | 220 | **−27%** | +10% |
| + Estrategia 2 | 7,100 | 210 | −16% | +5% |
| + Estrategia 3 | 7,800 | 180 | −8% | −10% |
| Combinado | 4,500 | 240 | **−47%** | +20% |

Lectura: el **combinado** sinergiza más que la suma de individuales. La mayoría de estrategias contra el cálido **empeoran el frío** — trade-off inevitable que se debe reportar explícitamente.

### Sobre qué temperatura calcular

Idealmente sobre [[../concepts/Temperatura-Operativa]] (captura efecto radiativo local). Si solo se pidió T del aire, es aproximación razonable salvo en casos con sol directo entrando o superficies muy calientes/frías cercanas al usuario.

## Estructura de libretas para el proyecto final

> "Si lo hacen bien, van a tener una libreta que analiza cada caso. Pero después tienen que hacer una libreta que **unifique todos los resultados**."

Estructura recomendada:

| # | Libreta | Contenido |
|---|---------|-----------|
| 001 | EDA simulación | Verificar propiedades, sistemas constructivos, materiales. Sanity de la simulación |
| 002 | EDA EPW | Cargar EPW, calcular T neutralidad mensual, zona de confort adaptativo |
| 003 | Análisis individual de cada caso | Para cada simulación: cargar SQL, calcular grados-hora, gráficas exploratorias |
| 004 | Unificación y comparación | Tomar resultados de las anteriores, armar tabla comparativa, reportar trade-offs |

Beneficio: permite **dividir el trabajo entre el equipo** (alguien la simulación, alguien el análisis, alguien la escritura).

> "Mi carta santa es que todos sepan hacer todo. Pero sé que es complicado."

## No generalizar sin estudio exhaustivo

> "Decir que algo funciona sin haberlo probado de manera exhaustiva, puede llevar a conclusiones erróneas. Y en eficiencia energética pasa un montón."

Ejemplos de generalizaciones peligrosas:

- "El aislamiento térmico siempre es bueno" → falso en climas cálidos secos con buena ventilación nocturna (atrapa calor).
- "Las ventanas dobles son mejores" → caro y de efecto pequeño en climas mexicanos sin AC.
- "Las protecciones solares ayudan en general" → en orientaciones E/W con alero horizontal puede no funcionar (sol oblicuo).

Cualquier afirmación de este tipo requiere estudio paramétrico **multi-clima, multi-orientación, multi-material**. En el proyecto final del taller las conclusiones son **específicas al caso analizado** — no se generaliza.

## Tarea / cierre del taller

No hay tarea explícita. La clase 008 cierra el ciclo de simulación + análisis. Quedan posibles temas avanzados:

- **Ventanas complejas** (multi-capa, low-E, argón) — el profesor adelanta que probablemente no se cubrirán por tiempo y porque su efecto es pequeño en climas mexicanos sin AC.
- **Aire acondicionado y setpoints** — siguiente clase (009).

> "Tampoco vamos a empezar de cero. Algo que siempre impulsa al IER es que ustedes entiendan los fenómenos. Entenderlos los va a llevar a un mejor lugar para trabajar."

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/Sunlit-Fraction]]
- [[../concepts/Algoritmo-Sombreamiento]]
- [[../concepts/Enfriamiento-Radiativo-Cielo]]
- [[../concepts/Grados-Hora-Disconfort]]

Conceptos profundizados:

- [[../concepts/Variables-Output-EnergyPlus]] — `Surface Outside Face Sunlit Fraction`, nota crítica sobre sub-superficies
- [[../concepts/Superficies-de-Sombramiento]] — algoritmo, atenuación selectiva
- [[../concepts/Confort-Adaptativo]] — métrica grados-hora
- [[../concepts/Mensajes-EnergyPlus]] — warning "many overlapping shadows"
- [[../concepts/Caricatura-Computacional]] — Sunlit Fraction multiplicativa
- [[../concepts/Radiacion-Onda-Larga]] — el cielo como sumidero a −15°C

Procedimiento nuevo:

- [[../procedures/Auditar-Sombreamiento-Ventanas]]

## Conexiones

- ← **Anterior:** [[007-CasoBaseAleros]] — el bug que esta clase resuelve
- → **Siguiente:** _009-AireAcondicionadoSetPoints_ — incorporar HVAC al modelo
- → Procedimientos clave:
  - [[../procedures/Auditar-Sombreamiento-Ventanas]]
  - [[../procedures/Comparar-Simulaciones-Python]]

## Recursos mencionados

- **Engineering Reference de Energy Plus**, sección "Shading and Sunlit Area Calculations" + "Solar Gains".
- **Claude (LLM)** — usado como first pass para entender el comportamiento; verificado con documentación.
- **Steven Brunton** — UWashington, dynamical systems. Café científico del IER lo entrevistó.
- **James Hoffman** — barista científico, YouTube.
- Patente de **Stanford et al. (~2017)** — película selectiva para enfriamiento radiativo pasivo.
- **Tesis de Miriam** — documentó un bug real en Airflow Network de E+.
- **Bug de diferencias finitas** — encontrado hace 15 años por un estudiante del grupo, sigue presente en E+ actual.

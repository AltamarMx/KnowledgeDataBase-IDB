# 002 — Conceptos Básicos y Balances de Calor

## Metadatos
- **Clase:** 002
- **Título:** Conceptos Básicos y Balances de Calor
- **Duración:** ~1h 33min
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** zona térmica, transferencia de calor 1D, balance de calor en superficie exterior, módulos de EnergyPlus, archivos EPW/TMY, caricaturas de modelado

---

## Resumen

Clase teórica central donde se explican los fundamentos físicos y matemáticos detrás de las simulaciones en EnergyPlus. Se introduce el concepto de zona térmica, se explica cómo EnergyPlus resuelve la transferencia de calor (1D, dependiente del tiempo), y se detalla el balance de calor en la superficie exterior con sus tres componentes: radiación de onda corta, radiación de onda larga y convección.

### Herramientas de IA para el curso

- **NotebookLM** (Google) — puede recibir videos de YouTube y responder preguntas indicando timestamps exactos del video
- **Gemini Gems** — el profesor creó un asistente personalizado cargado con la documentación de EnergyPlus (Input/Output + Engineering Reference) para que los estudiantes consulten. Tiene un prompt diseñado para propiciar el estudio, no solo dar respuestas.

### Zona Térmica

Un **volumen de aire delimitado por superficies** donde se puede resolver la temperatura interior. Es el concepto fundamental para plantear una simulación en EnergyPlus.

**Pregunta clave para identificar una zona térmica:**
> "¿Si me paro aquí, voy a sentir una temperatura diferente a la del exterior?"

- Si sí → probablemente es una zona térmica
- Si no, o si no puedo delimitar el volumen → no es zona térmica

**Ejemplos:**
- Pasillos abiertos, escaleras → generalmente NO son zonas térmicas (misma temperatura que exterior)
- Cuartos cerrados → SÍ son zonas térmicas
- Cafetería cerrada → podría ser zona térmica si se comporta diferente al exterior

Si no se puede definir el volumen, no se puede calcular cómo sube y baja la temperatura (el balance de masa requiere un volumen definido — mayor volumen = menor impacto por la misma energía).

### Transferencia de calor en EnergyPlus

EnergyPlus resuelve la ecuación de calor **dependiente del tiempo en 1 dimensión** (perpendicular a cada superficie):

> ∂T/∂t = k · ∂²T/∂x²

Donde k es conductividad, considerando también densidad (ρ) y calor específico (Cp) — es decir, la difusividad térmica de cada material.

**Métodos de solución:**

| Método | Descripción | Ventaja |
|--------|-------------|---------|
| **Conduction Transfer Function (CTF)** | Solución semi-analítica con funciones de transferencia (serie tipo Fourier). Método por defecto. | Solución instantánea, muy rápida |
| **Diferencias Finitas** | Discretización numérica clásica de la ecuación de calor | Más intuitiva, misma ecuación |

Los coeficientes del CTF dependen del **sistema constructivo** (materiales y su orden, de exterior a interior).

**Restricciones fundamentales:**
- **Solo 1 dimensión:** el flujo de calor es perpendicular a cada superficie. No modela flujo lateral.
- **Temperatura uniforme por superficie:** cada muro tiene una temperatura equivalente (promedio ponderado). Si la radiación pega en medio muro, EnergyPlus da un valor promedio.
- **Solo superficies planas:** no hay líneas curvas ni ventanas circulares. Razón: el factor de vista de una superficie plana consigo misma es 0; una curva se "ve a sí misma" y complicaría enormemente el cálculo radiativo.
- **No modela puentes térmicos** directamente (flujo lateral en cambios de material).

**Resolución temporal:**
- Puede resolver cada hora o cada minuto, de un día completo hasta un año completo
- Mínimo: 1 día. Máximo: 1 año (a 1 minuto = ~525,600 timesteps)

### Distinción crítica: "simulación dinámica"

- **Modelo dependiente del tiempo** (EnergyPlus, CTF, diferencias finitas) → correcto, considera masa térmica
- **Solo U o R con clima variable** → NO es dependiente del tiempo, aunque le llamen "simulación dinámica"
- La resistencia térmica (R) y su inversa (U) **no consideran la masa térmica** — surgen de eliminar la dependencia temporal de la ecuación de calor
- Las **NOM-008 y NOM-020** de México usan el modelo independiente del tiempo → inadecuadas para el clima mexicano

### Módulos de EnergyPlus (panorama)

| Módulo | Función |
|--------|---------|
| **CTF / Diferencias Finitas** | Transferencia de calor a través de muros opacos |
| **Window/Glass** | Modelos simples y complejos de ventanas (multicapa, marcos, intercambio radiativo onda corta/larga + convección, semi-vacío) |
| **Shading** | Cálculos de obstrucción solar por aleros y protecciones |
| **Daylighting** | Iluminación natural y deslumbramiento |
| **Sky Model** | Modelo de cielo como semiesfera dividida en ~156 parches (sol = parche con radiación directa) |
| **Airflow Network** | Modelo más complejo de EnergyPlus para ventilación natural (diferencia de densidad, viento) — no se usa este semestre |
| **Fotovoltaicos** | Producción de energía con diferentes modelos de paneles (single diode, etc.) |
| **HVAC Loops** | Sistemas mecánicos de calefacción/enfriamiento |
| **District Heating/Cooling** | Calentamiento/enfriamiento centralizado para múltiples edificaciones |

### Balance de calor en la superficie exterior

El balance tiene **3 componentes** que igualan al flujo conductivo que entra al muro:

#### 1. Radiación de onda corta (solar)

> q_SW = (I_directa_proyectada + I_difusa) × α

- **α** = absorptancia solar: blanco ~0.2–0.3, negro ~0.8
- EnergyPlus calcula la proyección sobre la superficie (ángulo solar) y el efecto de sombreamiento

#### 2. Radiación de onda larga

Intercambio radiativo con **4 fuentes**, cada una con ecuación de Stefan-Boltzmann y factor de vista:

| Fuente | Notas |
|--------|-------|
| **Ground (suelo)** | Temperatura del suelo, factor de vista depende de inclinación (techo = 0, muro = ~0.5) |
| **Cielo** | Temperatura cercana a 0 K → **efecto de enfriamiento**. En la noche, superficies pueden estar más frías que el aire. La radiación solar de día (~840 W/m²) domina, pero la pérdida radiativa al cielo (~-40 W/m²) es significativa |
| **Aire (atmósfera)** | A grandes distancias, el aire SÍ participa radiativamente (no es "transparente" como se asume en transferencia de calor básica) |
| **Alrededores** | Otros edificios/objetos con su propia temperatura. Depende del factor de vista |

**Coeficiente radiativo equivalente (h_r):** se transforma la ecuación de Stefan-Boltzmann (T⁴) a forma tipo Newton (h_r · ΔT) para simplificar el acoplamiento con la convección.

**Dato interesante:** existe un material que refleja la radiación visible y maximiza el intercambio radiativo de onda larga con el cielo → se enfría incluso bajo el sol directo. Sirve para enfriar fluidos pasivamente.

#### 3. Convección

> q_conv = h_conv × (T_aire - T_superficie)

El coeficiente convectivo **h_conv** depende de:
- Inclinación de la superficie (vertical vs. horizontal)
- Diferencia de temperaturas (ΔT)
- Rugosidad de la superficie (acero/vidrio = liso; concreto = rugoso → mayor h)
- Velocidad del viento

EnergyPlus usa correlaciones experimentales validadas. Se usan los valores por defecto, pero se pueden fijar (ej. para comparar con normativas como la NOM).

#### Ecuación completa del balance exterior

> I_s · α + q_LWR + q_conv = -k · ∂T/∂x |_{x=0}

Donde x=0 es la superficie exterior y x=L el espesor (superficie interior). El balance de la superficie interior se cubre en la siguiente clase.

### Archivos EPW y Año Meteorológico Típico (TMY)

- **EPW** (Energy Plus Weather) contiene datos horarios o minutales: temperatura, humedad relativa, radiación (global, directa, difusa), velocidad y dirección de viento, presión atmosférica
- **TMY** (Typical Meteorological Year) — **NO es un promedio**:
  - Para cada mes, se busca el mes que más se parece a todos los meses equivalentes de todos los años (distancias estadísticas, no promedios)
  - Diferentes meses pueden venir de diferentes años (ej. enero 2016, febrero 2018)
  - El rango de años de búsqueda se indica en el nombre del archivo
- **Fuente principal:** [One Building](https://climate.onebuilding.org/) — colección de EPW por país
- Los datos provienen de **reanálisis** (no estaciones meteorológicas directamente)
- Se puede construir un EPW propio desde una estación meteorológica (el instituto tiene una)
- **Limitación del TMY:** tiende a perder anomalías atípicas y el efecto de cambio climático

### Modelado como "caricatura"

Los modelos de edificaciones son siempre **simplificaciones** ("caricaturas") de la realidad. La clave es que tengan sentido físico:

- Se pueden **dividir superficies** para asignar diferentes sistemas constructivos (ej. separar trabe de ladrillo)
- Se pueden **combinar superficies** si no hay diferencias de radiación incidente ni son de materiales distintos
- Existe el objeto **masa térmica** para representar protuberancias o elementos no modelados en la geometría (ej. trabes que sobresalen al interior)
- En EnergyPlus no hay transferencia lateral entre superficies adyacentes — están acopladas solo por convección con el aire interior
- Las decisiones de simplificación requieren entender la física para saber qué es importante y qué se puede despreciar

### Proyecto final

- Los equipos deben **escoger una ciudad** y encontrar su EPW en One Building
- Ciudades con clima extremo dual (calor Y frío, ej. Monterrey, Sonora) = trabajo doble (hay que evaluar temporada cálida Y fría)
- Se evaluarán estrategias bioclimáticas para una casa en el clima seleccionado
- Tip: no dejarse llevar por el romanticismo de escoger su ciudad natal si tiene clima extremo

---

## Conceptos clave

- **[[Zona-Termica]]** — volumen de aire delimitado por superficies donde se resuelve la temperatura interior
- **[[Balance-de-Calor]]** — ecuación que iguala los flujos incidentes en una superficie con la conducción
- **[[Absorptancia-Solar]]** — fracción de radiación solar absorbida por una superficie (0–1)
- **[[Factor-de-Vista]]** — fracción de radiación emitida por una superficie que llega a otra
- **[[TMY]]** — año meteorológico típico construido con los meses más representativos de un periodo

Conceptos previos referenciados: [[Simulacion-Energetica]], [[Condiciones-de-Frontera]], [[Sistemas-Constructivos]], [[Envolvente-Arquitectonica]], [[Confort-Termico]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · NotebookLM · Gemini Gems · Radiance · Design Builder

## Conexiones

- **Anterior:** [[001-IntroduccionTallerIDB]] — Presentación del curso y ecosistema de software
- **Siguiente:** [[003-MiPrimeraSimulacion]] — Balance interior y primera simulación práctica
- **Materia relacionada:** Energía en Edificaciones (ventilación, cargas térmicas, geometrías complejas)

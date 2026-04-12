# 004 — Interpretando los Mensajes de Simulaciones y Construction Sets

## Metadatos
- **Clase:** 004
- **Título:** Interpretando los Mensajes de Simulaciones y Construction Sets
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** archivo ERR, warnings vs errores severos, debugging de simulaciones, construction sets, measures (OSM e IDF), warm-up period, convergencia, cálculo de sombras, SQL y CSV, site source factors, masa térmica, condiciones de frontera avanzadas

---

## Resumen

Clase centrada en desarrollar la habilidad de **leer e interpretar los mensajes que genera EnergyPlus** (archivo .err) para diagnosticar y corregir simulaciones. Se revisa una tarea de alumnos para ilustrar errores comunes, se introduce el concepto de **Construction Set** como forma eficiente de asignar sistemas constructivos, y se explica el **warm-up period** (convergencia de la condición inicial). También se cubren temas de salida de datos (SQL, CSV, reporte HTML) y conceptos como masa térmica y site source factors.

---

## 1. Estructura de archivos del proyecto OSM

- Cuando Open Studio corre una simulación, **crea una carpeta** junto al archivo .osm con todos los archivos necesarios
- **No mover el .osm dentro de esa carpeta** — si lo haces, al abrir el .osm se crea otra carpeta anidada, en un ciclo infinito
- **No guardar archivos propios** dentro de la carpeta que crea Open Studio — se borra cada vez que se vuelve a correr
- Si mueves el .osm a otra ubicación, **pierde la ruta del EPW** y hay que reasignarla
- Botón **Show Simulation** abre la carpeta de resultados

---

## 2. Archivo ERR: warnings vs errores

El archivo `.err` se genera en la carpeta de simulación y se abre con cualquier editor de texto (ej. Notepad).

| Tipo | Efecto | Acción |
|------|--------|--------|
| **Warning** | La simulación **continúa** — EnergyPlus toma un valor por defecto o señala algo "raro" | Revisar como experto si el warning afecta los resultados |
| **Severe Error** | La simulación **se detiene** | Corregir el problema e intentar de nuevo |

### Warnings comunes (ignorables en este curso)

- Warnings de **análisis de ciclo de vida** (Lifecycle Assessment) — Open Studio está diseñado para cumplir métricas ASHRAE; si no se definen salidas de consumo energético, aparecen estos warnings
- Warnings de **outputs no definidos** — por la misma razón
- Warning de **design days** no incluidos en el EPW — no afecta si no se está dimensionando equipo HVAC
- Estos warnings **aparecerán repetidamente** en todas las simulaciones del curso y está bien

### Errores severos comunes

- **Falta archivo de clima (EPW)** — no se asignó o se movió el .osm y perdió la ruta
- **Material faltante en un sistema constructivo** — si una Construction no tiene todas sus capas, EnergyPlus no puede calcular la transferencia de calor
  - Mensaje: `Missing material in property "Outside Layer"` + nombre de la construction
  - Solución: abrir la Construction y arrastrar el material faltante

**Principio clave:** una vez que sabes leer el .err, los errores se corrigen en minutos. Sin esa habilidad, puedes estar horas sin saber qué está mal.

---

## 3. Construction Sets

Un **Construction Set** es una agrupación que asigna sistemas constructivos automáticamente según el tipo y condición de frontera de cada superficie.

### Estructura del Construction Set

| Categoría | Aplica a |
|-----------|----------|
| **Exterior Surface Construction** | Muros, pisos y techos con condición Outdoor |
| **Interior Surface Construction** | Superficies con condición Surface/Interzone |
| **Ground Contact Surface Construction** | Superficies con condición Ground |
| **Adiabatic Surface Construction** | Superficies con condición Adiabatic |
| **Sub Surface** | Puertas y ventanas (también diferenciadas por condición) |

### Cómo usar un Construction Set

1. Crear el Construction Set en la pestaña correspondiente y darle un **nombre descriptivo** (ej. "casa_ladrillo_concreto" — referencia a ladrillo en muros, concreto en techos)
2. Arrastrar los sistemas constructivos desde **My Model** a cada categoría
3. Ir a **Facility** y asignar el Construction Set al edificio completo
4. Todas las superficies que coincidan con la condición recibirán automáticamente su sistema constructivo

### Ventajas

- **Escala**: un edificio puede tener 200+ superficies — imposible asignar una por una
- **Reutilización**: un Construction Set bien nombrado (ej. por norma ASHRAE, zona climática, nivel de aislamiento) se puede reusar entre proyectos
- **Sobrescritura selectiva**: si una superficie necesita un sistema diferente, se asigna manualmente y sobrescribe al del set (cambia de verde — default — a otro color)

### Verificación visual

- Las superficies con sistema asignado por el Construction Set aparecen en **verde** (default)
- Si se sobrescribe manualmente, cambia de color
- Si una condición de frontera no tiene sistema asignado en el set, esa superficie queda **sin sistema** y genera error

---

## 4. Measures (introducción)

Los Measures son scripts que modifican la simulación en dos puntos del flujo:

```
OSM → [OpenStudio Measures] → traducción a IDF → [EnergyPlus Measures] → simulación
```

1. **OpenStudio Measures** — modifican el modelo OSM antes de traducirlo a IDF
2. **EnergyPlus Measures** — modifican el IDF antes de que corra EnergyPlus

**Utilidad principal:** agregar funcionalidades que no están en la interfaz de Open Studio. Ejemplos:
- Variar el porcentaje de ventana respecto al muro (25%, 50%, 75%, 100%) → **estudios paramétricos**
- Cambiar sistemas constructivos en todas las superficies exteriores por tres opciones diferentes → tres simulaciones automáticas
- Potencial para automatizar cumplimiento normativo

---

## 5. Warm-up Period

### El problema de la condición inicial

EnergyPlus inicializa todas las zonas térmicas y materiales a **23°C**. Esta temperatura arbitraria no corresponde a la realidad del clima simulado.

### Cómo funciona el warm-up

1. Toma el **primer día** de la simulación (por defecto, 1 de enero)
2. Simula ese día completo partiendo de 23°C
3. Al final del día, registra la temperatura final (ej. 25°C)
4. **Repite** el mismo día, pero partiendo de 25°C
5. Repite hasta que la diferencia entre iteraciones sea < criterio de convergencia (~0.1°C)
6. Resultado: alcanza un **estado oscilatorio permanente** donde la condición inicial de 23°C ya no influye

### Factores que afectan la convergencia

- **Masa térmica alta** (materiales densos, gruesos) → más iteraciones para converger
- **Clima severo** (ej. Canadá en invierno) → mayor distancia de 23°C al equilibrio → más iteraciones
- **Casas pequeñas/livianas** → convergen rápido (3 warm-up days típicos)

### Implicación importante para comparaciones

- Si se van a comparar simulaciones en un mismo día, **todas deben empezar en la misma fecha**
- La edificación "recuerda" el clima de los días anteriores por efecto de masa térmica
- No es lo mismo empezar el 1 de enero que el 1 de febrero → el warm-up del primer día depende del clima de ese día

---

## 6. Cálculo de sombras

EnergyPlus calcula las **máscaras de sombramiento** por defecto **cada ~20 días** (no diariamente). La trayectoria solar aparente cambia poco en 20 días, así que es una buena aproximación.

- Se puede forzar cálculo diario pero es más lento computacionalmente
- Esta simplificación hace que EnergyPlus **no sea el mejor programa para iluminación natural** — Radiance (trazado de rayos inverso) calcula cada hora
- Herencia de cuando las computadoras eran lentas (EnergyPlus nació ~1976-79 como otro programa)

---

## 7. Resultados de la simulación

### SQL

- EnergyPlus guarda resultados en una **base de datos SQL** estructurada
- Eficiente: descompone fechas en componentes (día, mes, hora) en vez de repetir fechas completas
- Para acceder se necesita una interfaz — no se puede abrir como texto plano
- El profesor escribió un **paquete en Python** (`ear_tools`) que lee directamente del SQL, ahorrando pasos

### CSV

- Se puede generar desde el SQL pero el formato de Open Studio es "medio feo"
- Requiere pedirlo explícitamente (measure Create CSV Output)
- No confundir CSV con Excel — CSV es un archivo de texto separado por comas

### Reporte HTML

- Open Studio genera automáticamente un reporte HTML con métricas generales
- Pensado para cumplimiento de estándares ASHRAE
- Incluye site source factors, consumo por área, etc.

---

## 8. Site Source Factors

Relación entre la energía consumida en el **sitio** (edificación) y la energía requerida en la **fuente** (planta generadora), contabilizando pérdidas de generación y transmisión.

- En EE.UU., electricidad tiene factor ~3× (se pierde 2/3 de la energía entre planta y edificio)
- Usado para análisis de ciclo de vida y definiciones de edificaciones **Net Zero**
- **México no tiene site source factors** oficiales — solo factores de transmisión del sistema eléctrico nacional, faltan eficiencias de planta
- No depende de la edificación, sino de la infraestructura energética del país

---

## 9. Masa térmica (ampliación)

**Definición:** capacidad de un material para almacenar energía térmica.

**Fórmula por unidad de área:**

> Masa_térmica = ρ × c × L

Donde:
- ρ = densidad [kg/m³]
- c = calor específico [J/(kg·K)]
- L = espesor [m]
- **Unidades resultantes:** J/(m²·K)

Al multiplicar por el área del muro → **J/K** = cuánta energía se necesita para elevar la temperatura de todo el material 1 K.

**Rangos de densidad:**
- Poliestireno expandido (EPS): ~35 kg/m³
- Concreto alta densidad: ~2500 kg/m³
- Factor de ~70× → el concreto almacena ~70 veces más energía por volumen

**Implicaciones:**
- Más masa térmica → variaciones de temperatura menores (almacena y libera energía lentamente)
- Una casa sin muebles tiene menos masa térmica → mayores oscilaciones térmicas
- Las **particiones interiores** (cubículos, divisiones a media altura) agregan masa térmica adicional
- Muebles, libros, y objetos en general contribuyen masa térmica al espacio

---

## 10. Condiciones de frontera (ampliación)

### Todas las condiciones disponibles en EnergyPlus

| Condición | Descripción |
|-----------|-------------|
| **Outdoor** | Convección + radiación onda corta + radiación onda larga (cielo, ground, aire, objetos) |
| **Surface/Interzone** | Flujo entre dos zonas adyacentes |
| **Adiabatic** | Flujo de calor = 0 en la cara exterior |
| **Ground** | Temperatura del suelo |
| **Other Side Coefficients** | Temperatura constante definida por el usuario |

### Combinaciones con Sun/Wind Exposure

Se puede tener condición **Outdoor** pero desactivar la exposición al sol y/o al viento:

| Caso | Sol | Viento | Ejemplo de uso |
|------|-----|--------|----------------|
| Outdoor completo | Sí | Sí | Fachada normal |
| Outdoor sin sol | No | Sí | Superficie sombreada permanentemente por edificio adyacente |
| Outdoor sin sol ni viento | No | No | **Estacionamiento subterráneo**: tiene aire (convección) pero no sol ni viento directo |

**Aplicación más común de "no sun":** pisos con estacionamiento subterráneo — no se dibuja el estacionamiento como zona térmica sino que se le quita la exposición al sol al piso, manteniendo convección. La suposición es que la temperatura del aire del estacionamiento es la temperatura exterior.

---

## Conceptos clave

- **[[Sistemas-Constructivos]]** — Construction Sets como agrupación eficiente
- **[[Condiciones-de-Frontera]]** — combinaciones con sun/wind exposure, nuevas condiciones
- **[[Masa-Termica]]** — ρ × c × L, efecto estabilizador en temperatura interior
- **[[Warm-up-Period]]** — convergencia a estado oscilatorio permanente

Conceptos previos referenciados: [[Balance-de-Calor]], [[Zona-Termica]], [[Simulacion-Energetica]], [[Absorptancia-Solar]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · Notepad · Radiance

## Conexiones

- **Anterior:** [[003-MiPrimeraSimulacion]] — Primer cubo, asignación manual de materiales y construcciones
- **Siguiente:** [[005-AnalisisSimulacionesPython]] — Análisis de datos con Python, lectura de SQL

## Tarea asignada

- Dos zonas térmicas (dos "casitas") con medidas y alturas dadas por el profesor
- **Sin ventanas** todavía — se verán en la siguiente sesión
- Tres sistemas constructivos diferentes
- Usar Construction Set para asignar
- La simulación debe funcionar correctamente (revisar ERR)
- Pedir datos de temperatura por zona para análisis posterior con Python (144 datos/día por zona a paso temporal de 10 min → 52,560 datos/zona/año)

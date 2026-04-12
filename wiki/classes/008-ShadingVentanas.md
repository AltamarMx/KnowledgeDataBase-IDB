# 008 — Shading en Ventanas

## Metadatos
- **Clase:** 008
- **Título:** Shading en Ventanas
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** Sunlit Fraction, radiación incidente en sub-superficies vs superficies, algoritmo de sombramiento de EnergyPlus, debugging de simulaciones, grados-hora de disconfort, temperatura de neutralidad, modelo adaptativo de confort, enfriamiento radiativo nocturno

---

## Resumen

Clase que resuelve el "happy accident" de la sesión anterior: se descubre que EnergyPlus **no reporta el efecto del sombramiento en la variable de radiación incidente sobre ventanas (sub-superficies)**, sino que calcula internamente una **Sunlit Fraction** que aplica al hacer el balance de energía. Se documenta el proceso completo de debugging (verificar datos, simulación, IDF, consultar documentación). En la segunda parte se introducen las **métricas de confort térmico** para el proyecto final: temperatura de neutralidad, límites adaptativos y **grados-hora de disconfort** como área bajo la curva.

---

## 1. El "quirk" de la radiación incidente sobre ventanas

### El problema

En la clase 007 se observó que la variable `Surface Outside Face Incident Solar Radiation Rate Per Area` daba **valores idénticos** para una ventana con y sin protecciones solares. Esto no tenía sentido físico.

### La explicación

EnergyPlus trata las **sub-superficies (ventanas)** de manera diferente a las superficies (muros):

| Tipo de superficie | Radiación incidente reportada | Sombramiento |
|-------------------|-------------------------------|--------------|
| **Muro (superficie)** | Ya incluye el efecto de sombramiento | Directo en la variable |
| **Ventana (sub-superficie)** | **Sin** efecto de sombramiento | Se aplica internamente vía Sunlit Fraction |

- EnergyPlus calcula la `Surface Outside Face Sunlit Fraction` por separado
- Al hacer el balance de energía, multiplica: radiación incidente × Sunlit Fraction
- Por eso la variable de radiación incidente sobre ventanas parece no cambiar con protecciones

### Verificación

- Pedir la radiación incidente sobre el **muro padre** (no la ventana) → ahí sí se ve diferencia
- Pedir la variable `Surface Outside Face Sunlit Fraction` → muestra la fracción iluminada (0 a 1)
- Sunlit Fraction = 1 significa que el sombramiento bloquea toda la radiación **directa**

### Ecuación de ganancia solar (Engineering Reference)

La ganancia solar total sobre una superficie exterior combina:
- **Radiación directa:** absorptancia × intensidad beam × **Sunlit Fraction (Ss/S)**
- **Radiación difusa:** factores de vista con cielo y suelo
- **Radiación reflejada por el suelo**

Las protecciones solares afectan tanto la Sunlit Fraction (directa) como los factores de vista (difusa y reflejada), pero el efecto principal es sobre la directa.

---

## 2. Proceso de debugging de simulaciones

Orden recomendado para diagnosticar resultados inesperados:

1. **Verificar archivo cargado** — ¿estoy leyendo el SQL correcto? Error común: dejar la ruta hardcoded dentro de la función
2. **Verificar geometría visual** — ¿las shading surfaces aparecen en el 3D?
3. **Verificar el IDF** — ¿el archivo de entrada contiene los objetos de sombramiento?
4. **Verificar nombres de variables** — ¿coinciden los nombres de las superficies en la simulación?
5. **Diseñar experimento de control** — pedir una variable relacionada (ej. radiación sobre el muro padre)
6. **Consultar LLM con cautela** — Claude/ChatGPT pueden dar pistas, pero admiten inventar explicaciones
7. **Verificar en documentación** — Engineering Reference para la ecuación y el algoritmo real

### Sobre la fiabilidad de EnergyPlus

De ~11 veces que el grupo de investigación sospechó un error en EnergyPlus, **solo 2 fueron errores reales** del software. Las ~9 restantes eran errores del usuario o comprensión incompleta. Errores reales encontrados:
- Comportamiento anómalo en Airflow Network (encontrado por Miriam durante el doctorado)
- Criterio de convergencia mal planteado en diferencias finitas con muchos elementos

---

## 3. Algoritmo de sombramiento en EnergyPlus

Documentado en el **Engineering Reference**, sección "Shading and Sunlit Area Calculations":

- Transformación de coordenadas globales a relativas para cada superficie
- Cálculo de **overlapping** — cuando múltiples sombras se superponen, detecta la intersección para no contar doble
- **Sunlit Fraction** aplica solo a radiación directa
- Los **factores de vista** (cielo y suelo) capturan el efecto sobre difusa y reflejada
- Asimetría en Sunlit Fraction por la trayectoria solar y geometría del modelo

---

## 4. Métricas de confort térmico para el proyecto final

### Temperatura de neutralidad

- Temperatura a la cual una persona no siente ni calor ni frío
- Cambia mensualmente en función de la temperatura exterior promedio
- Base para calcular los límites de confort

### Límites adaptativos de confort

- **Límite superior:** temperatura de neutralidad + amplitud (modelo de Humphreys)
- **Límite inferior:** temperatura de neutralidad − amplitud
- La amplitud depende de la **variación climática** del sitio: climas con mayor oscilación → la gente se adapta más → franja más amplia
- Son modelos adaptativos, no mexicanos — falta investigación de confort en México

### Grados-hora de disconfort

Métrica principal para evaluar estrategias bioclimáticas:

- **Grados-hora cálidos:** área bajo la curva cuando la temperatura interior supera el límite superior
- **Grados-hora fríos:** área bajo la curva cuando la temperatura interior cae bajo el límite inferior
- Unidades: °C·h (números grandes, ej. 32,535 para un año de 8,760 horas)
- **Ventaja sobre % de tiempo:** captura no solo cuándo hay disconfort, sino **cuánto** disconfort (magnitud)

### Por qué no usar solo % de tiempo

Dos edificaciones pueden tener el mismo porcentaje de tiempo en disconfort pero una se caliente/enfría mucho más que la otra. Los grados-hora capturan esa diferencia.

### Flujo de cálculo

1. Calcular temperatura de neutralidad mensual
2. Calcular límites superior e inferior adaptativos
3. Para cada hora: si Ti > límite superior → sumar (Ti − límite) a grados-hora cálidos
4. Para cada hora: si Ti < límite inferior → sumar (límite − Ti) a grados-hora fríos
5. Comparar caso base vs estrategias: calcular reducción porcentual

---

## 5. Temperatura interior menor que exterior

Fenómeno posible por **enfriamiento radiativo nocturno**:
- El cielo tiene una temperatura equivalente de ~−15°C
- Superficies con factor de vista al cielo intercambian radiación de onda larga
- Techos de lámina metálica (alta emisividad, baja inercia térmica, conductividad ~65 W/m·K, ~1 mm espesor) se enfrían por debajo de la temperatura del aire
- Noches despejadas = mayor efecto (sin nubes que bloqueen el intercambio)
- Aplicación: películas selectivas que reflejan radiación solar pero permiten intercambio de onda larga → enfriamiento pasivo

---

## 6. Estructura del proyecto final

### 5 simulaciones

1. Caso base
2. Caso base + Estrategia 1
3. Caso base + Estrategia 2
4. Caso base + Estrategia 3
5. Caso base + todas las estrategias

### Libretas Jupyter recomendadas

1. **Exploración de datos** — verificar propiedades, sistemas constructivos
2. **Cálculo de métricas** — temperatura de neutralidad, grados-hora, con el caso base bien definido
3. **Resultados unificados** — comparación de todos los casos, gráficas finales

### Precaución sobre generalizaciones

- No generalizar resultados de un caso particular a todos los climas/orientaciones/materiales
- El aislamiento térmico no siempre es bueno (depende del clima y uso)
- Se necesitan estudios exhaustivos (múltiples orientaciones, climas, materiales, áreas de ventana)

---

## Conceptos clave

- **[[Superficies-de-Sombramiento]]** — Sunlit Fraction, algoritmo de sombramiento, overlapping
- **[[Confort-Termico]]** — grados-hora de disconfort, temperatura de neutralidad, modelo adaptativo
- **[[Ventanas]]** — radiación incidente en sub-superficies, quirk de EnergyPlus

Conceptos previos referenciados: [[Balance-de-Calor]], [[Temperatura-Operativa]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · IDF Editor · SunPath (Andrew Marsh)

## Conexiones

- **Anterior:** [[007-CasoBaseAleros]] — Caso base y análisis comparativo que aquí se completa
- **Anterior:** [[005-AnalisisSimulacionesPython]] — Flujo de análisis con Python que aquí se extiende

## Tarea

- No se dejó tarea nueva — continúan los dos ejercicios asignados en la clase 007

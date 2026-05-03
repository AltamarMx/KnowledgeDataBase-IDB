---
title: 010 — EnerHabitat (Parte 1)
type: clase
clase: 010
profesor: Guillermo Barrios del Valle
fuente: raw/videos/010_EnerHabitat_Parte1.md
fecha_ingesta: 2026-05-02
tags: [clase, enerhabitat, herramienta-IER, sol-aire, factor-de-decremento, oscilatorio-permanente]
aliases: [Clase 010]
---

# 010 — EnerHabitat (Parte 1)

## Metadatos

- **Clase:** 010
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/010_EnerHabitat_Parte1.md`
- **Tipo:** Clase mixta — presentación de la herramienta del IER + teoría detrás + demo Python que falla por documentación incompleta

## Resumen

Presentación de **EnerHabitat**, herramienta del grupo IER para evaluación rápida de sistemas constructivos. Tres bloques principales:

1. **Tour de la web app** — cargar EPW, definir geometría/materiales, comparar sistemas, leer métricas (FD, FD sol-aire, tiempo de retraso, energía).
2. **Teoría detrás** — temperatura sol-aire como condición de frontera "equivalente", estado oscilatorio permanente como objetivo del cálculo, métricas adimensionales.
3. **Demo del paquete Python** que falla por documentación incompleta — lección sobre la importancia de sesiones de prueba con usuarios reales.

> "Va dos clases que no me sale. Es otra clase frustrante. Pero la lección es que las sesiones de prueba sirven para mejorar la herramienta."

Cierre con repositorios GitHub, distribución vía PyPI, y discusión sobre confianza en paquetes Python.

## Qué es EnerHabitat

Herramienta del IER que **evalúa sistemas constructivos** de muros o techos opacos bajo distintos climas mexicanos. Detalle completo en [[../tools/EnerHabitat]].

### Dos interfaces

| Interfaz | Tecnología | Cuándo |
|----------|------------|--------|
| **Web app** | Python + Shiny | Acceso rápido — `enerhabitat.com` |
| **Paquete Python** | PyPI (`pip install enerhabitat`) | Estudios paramétricos, scripting |

El paquete es la **fuente única de verdad** — la web es una capa encima del paquete.

### Para qué sirve y para qué NO

> "Es una herramienta para tomar primeras decisiones."

Para qué SÍ:

- Comparar 2-3 sistemas constructivos rápido.
- Evaluar [[../concepts/Posicion-Aislante|posición del aislante]].
- Cuantificar impacto del color (caso COMEX).
- Estudios académicos del impacto de cada parámetro.

Para qué NO:

- Cálculos energéticos reales (kWh anuales).
- Análisis de confort completo.
- Edificios complejos.

> "No lo deben usar para hacer cálculos reales de energía, de emisiones, de confort. Está muy limitado."

## Tour de la web app

### Carga del EPW

Por default trae Temixco. Funcionalidad **nueva** (no estaba en la versión vieja): subir un EPW propio. Antes el grupo IER tenía que cargarlo manualmente en el servidor — ahora el usuario tiene control directo.

> "Antes nos pedían 'oye, sube el EPW de mi ciudad'. Ahora le damos control al usuario."

### Visualización del clima

Auto-graficado al cargar el EPW:

- **Temperatura del aire exterior** del mes seleccionado.
- **Radiación global, difusa y directa** sobre plano horizontal.
- **Zona de confort** según [[../concepts/Confort-Adaptativo|Humphreys-Nicol]] — la línea verde delgada.

### Selección del mes

Dropdown — uno a la vez (no anual). Ver el mes más cálido para evaluar peor caso, el más frío para el otro extremo.

### Definir geometría

- **Tipo**: Muro (90°) o Techo (180°). Solo dos opciones — sin inclinaciones intermedias.
- **Orientación**: Norte, Sur, Este, Oeste. Solo cardinales.
- **Absortancia** (0-1).

### Capas del sistema constructivo

Botón **+ Capa** para agregar; orden de exterior a interior. Material desde dropdown (base local) + espesor.

Ejemplo demo: tabique 14 cm con absortancia 0.7 vs tabique 14 cm con absortancia 0.3 (blanco).

### Modo HVAC

Toggle Sin AC / Con AC.

- **Sin AC**: T interior flota; reporta FD, tiempo de retraso, T máxima/mínima.
- **Con AC**: T interior constante; reporta energía cooling y heating.

### Cálculo y métricas

~5 segundos. Reporta:

| Métrica | Significado |
|---------|-------------|
| **Factor de Decremento (FD)** | $\Delta T_i / \Delta T_o$ — ingenuo, puede ser > 1 |
| **FD sol-aire** | $\Delta T_i / \Delta T_{sa}$ — siempre < 1, **métrica recomendada** |
| **Tiempo de retraso** | Desfase entre pico exterior e interior |
| **Energía transmitida** | Bug — sale 0. No usar |

Procedimiento detallado en [[../procedures/Usar-EnerHabitat-Web]].

## Teoría — la PDE que resuelve

EnerHabitat resuelve la PDE 1D dependiente del tiempo:

$$
\rho c_p \frac{\partial T}{\partial t} = -k \frac{\partial^2 T}{\partial x^2}
$$

con condiciones de frontera:

### Frontera exterior ($x = 0$)

$$
q''_{LWR} + q''_{conv} + I \cdot \alpha = -k \frac{\partial T}{\partial x}\bigg|_{x=0}
$$

Tres componentes (radiación de onda corta absorbida + convección + LWR) **igualados a la conducción** que entra al muro. Igual estructura que [[Balance-de-Calor]] en E+.

### Frontera interior ($x = L$)

$$
-k \frac{\partial T}{\partial x}\bigg|_{x=L} = q''_{conv,interior}
$$

**Solo flujo convectivo** — sin radiación de onda larga (no hay otras superficies con quién intercambiar) ni de onda corta (sin ventanas, sin luces).

### Condición inicial

$$
T(x, t=0) = \overline{T}_{aire,día}
$$

T uniforme en todos los nodos = T promedio del día representativo.

## Estado oscilatorio permanente

> "Resuelvo el día varias veces hasta que los perfiles de T en dos días consecutivos son iguales."

EnerHabitat **repite el mismo día** hasta que la T al inicio = la T al final → **estado oscilatorio permanente**. Esto elimina la dependencia de la condición inicial arbitraria — análogo conceptual al [[../concepts/Warm-up-Period]] de E+.

Diferencia: en E+ el warm-up es **pre-cálculo**, después arranca la simulación real con días distintos. En EnerHabitat el oscilatorio permanente **es el cálculo principal** — se reporta el último día convergido.

Detalle en [[../concepts/Estado-Oscilatorio-Permanente]].

### Día representativo

Cada paso temporal del día representativo = **promedio** del valor a esa hora a través de todos los días del mes seleccionado. Captura el comportamiento promedio sin extremos.

## Temperatura sol-aire

Concepto físico clave de la clase y de EnerHabitat:

$$
T_{sa} = T_{aire} + \frac{\alpha \cdot I}{h_c} + \Delta T_{LWR}
$$

donde $\Delta T_{LWR} = 0$ para muros (vertical) y $-3.4$ °C para techos horizontales (variación lineal entre 0° y 90° de inclinación).

Cualidad: **temperatura equivalente** que combina los tres mecanismos (radiación de onda corta + convección + LWR) en una sola variable. Permite simplificar la condición de frontera exterior a un solo coeficiente convectivo "equivalente":

$$
q''_{exterior} = h_c (T_{sa} - T_{superficie})
$$

Detalle en [[../concepts/Temperatura-Sol-Aire]].

> "La temperatura sol-aire combina T + radiación. Por eso vemos esos picos cuando hay radiación directa."

### Forma típica

| Orientación | Patrón de $T_{sa}$ |
|-------------|---------------------|
| Norte | Plana, cerca de $T_{aire}$ |
| Sur (hem. norte) | Pico al mediodía |
| Este | Pico de mañana |
| Oeste | Pico de tarde |

Codifica la orientación — un patrón inesperado es pista de error. Detalle en [[../concepts/Trayectoria-Solar]].

## Métricas — Factor de Decremento

> "Una falla del FD ingenuo es que no considera la radiación solar — solo la T."

### FD ingenuo

$$
FD = \frac{\Delta T_i}{\Delta T_o}
$$

donde $\Delta T_o$ = amplitud de la T del aire exterior. **Puede ser > 1** con alta absortancia (la radiación calienta la pared más que la T del aire). **No** es buena métrica con clima soleado.

### FD sol-aire (recomendado)

$$
FD_{sa} = \frac{\Delta T_i}{\Delta T_{sa}}
$$

donde $\Delta T_{sa}$ = amplitud de la T sol-aire. **Siempre 0-1** porque $T_{sa}$ es el límite termodinámico real.

> "Si le sale mayor que uno, algo está mal."

### Tiempo de retraso

Desfase temporal entre el pico de $T_{sa}$ (exterior) y el pico de $T_i$ (interior). Mayor = más inercia. Combinada con FD_sa caracteriza el sistema constructivo.

Detalle de las dos métricas en [[../concepts/Factor-de-Decremento]].

### Comparación con NOM

> "Las NOMs hablan de potencia, no de energía. Y resuelven un punto en el tiempo. Es absurdo pensar que con un punto puedo hacer un cálculo anual."

NOM-008 / NOM-020 usan U = 1/R como métrica — modelo independiente del tiempo, ignora la masa térmica. FD sol-aire **sí captura** la masa. Por eso es métrica más rica para evaluar sistemas en climas mexicanos con oscilación diaria.

## Demo Python — bug y lección

El profesor intenta hacer una demo en vivo del paquete `enerhabitat`. Setup:

```bash
uv init test_enerhabitat
cd test_enerhabitat
uv add enerhabitat pandas matplotlib jupyter
uv run jupyter notebook
```

```python
import enerhabitat

location = enerhabitat.Location(epw_file="EPW/cuernavaca.epw")
wall = enerhabitat.Wall(location)
wall.azimuth     = 90
wall.tilt        = 90
wall.absorptance = 0.6
wall.layers      = [("tabique_recocido", 0.14)]
wall.set_day(month=3)
wall.tsa()
wall.solve()  # <-- falla aquí
```

Error: `assignment is read only`.

### Diagnóstico en vivo

El profesor revisa la documentación y descubre que:

- La documentación oficial **no menciona** el parámetro `energy=True` que activa el AC.
- Los ejemplos del repo `validation` (de Miriam) usan un patrón ligeramente distinto al documentado.
- La API ha cambiado entre versiones sin actualizar la documentación.

Decide no resolver en vivo y cerrar la clase con la lección.

### Lección sobre testing

> "La lección es que las sesiones de prueba con usuarios reales son indispensables. Hoy ustedes fueron mi sesión de prueba. Yo me sé el flujo de memoria — pero la documentación está incompleta y eso se descubre solo cuando alguien que no es yo intenta usarla."

Aplicable a cualquier herramienta que se desarrolla:

- Documentación que parece clara para el autor puede ser opaca para usuarios.
- API changes deben acompañar updates a la documentación.
- Sesiones de prueba con usuarios reales **antes de publicar** evitan estos casos.

Procedimiento aproximado del paquete (con caveats) en [[../procedures/Usar-EnerHabitat-Python]].

## Repositorios GitHub

Organización **`enerhabitat`** en GitHub:

| Repo | Para qué |
|------|----------|
| `enerhabitat` | El paquete Python (publicado en PyPI) |
| `web-app` | Aplicación web Shiny (corre local también) |
| `validation` | Scripts que comparan EnerHabitat vs Energy Plus |

### Validación contra Energy Plus

Patrón para validar el paquete: replicar las mismas condiciones en E+ y comparar resultados.

> "Crear todas las condiciones en E+ para que se parezca a EnerHabitat — un cuarto idealizado de 2.5 m con pared adiabática. Es una talachita pero no es imposible. Y entre esas crear un EPW que tenga la misma forma."

Es el patrón estándar de validación de software: dos métodos independientes deben converger al mismo resultado. Detalle en `enerhabitat/validation`.

## PyPI — distribución y confianza

> "Cualquiera puede publicar en PyPI sin revisión. Confiamos en pandas y numpy porque son grandes y tienen procesos internos. Pero instalar paquetes pequeños es peligroso — yo publiqué EnerHabitat y nadie revisó."

Lecciones sobre el ecosistema Python:

- **PyPI no revisa** los paquetes — cualquiera puede publicar.
- Los **paquetes establecidos** (pandas, numpy, matplotlib) tienen procesos internos de revisión.
- **Paquetes pequeños** requieren confianza en el autor / institución.
- EnerHabitat es publicado por el grupo IER — la cadena de confianza es relevante.

Aplicable también a `iertools` ([[../tools/iertools]]) — paquete del mismo grupo.

## Reuso de paquetes existentes

EnerHabitat **reutiliza** en lugar de reinventar:

- **`pvlib`** — proyección de radiación solar sobre superficies orientadas. Estándar de la industria solar.
- **`numpy`** — cálculo numérico, optimizado.
- **`pandas`** — series temporales.
- **`shiny`** — framework web Python.

> "¿Para qué rehacemos las cosas si pvlib ya lo hace? Por eso reutilizamos pvlib."

Filosofía aplicable a cualquier desarrollo de software libre: confiar en la infraestructura comunitaria.

## Algoritmo numérico

EnerHabitat usa:

- **Discretización espacial** — volúmenes de control con medios nodos en superficies.
- **Esquema temporal** — semi-implícito (más robusto que explícito; el explícito tiene problemas de inestabilidad).
- **Solución del sistema lineal** — TDMA (Tridiagonal Matrix Algorithm) optimizado con NumPy.

> "Está optimizado con [Numba] y se pasó a NumPy para que funcione más rápido."

## Caso COMEX

Caso real de uso histórico:

> "COMEX usó EnerHabitat para hacer cálculos de que sus pinturas son buenas. Nos pidieron que diéramos de alta más materiales con valores de absortancia. Nos extendieron una carta."

Aplicación: cuantificar el impacto del **color (absortancia)** del exterior con sistemas constructivos típicos en climas mexicanos. Útil para validar afirmaciones de marketing de la industria de pinturas.

## Limitaciones — recapitulación

EnerHabitat asume:

| Suposición | Realidad |
|------------|----------|
| Una sola pared (no edificación) | Edificios tienen 4+ superficies |
| Cuarto idealizado de 2.5 m con pared adiabática opuesta | Cuartos reales tienen 4 muros + techo + piso |
| Sin ventanas, sin ventilación | Casi todas tienen |
| Día representativo del mes | Cada día es distinto |
| Estado oscilatorio permanente | Días reales no son iguales |

Es una **caricatura útil** para primeras decisiones — para análisis serio → Energy Plus.

> "EnerHabitat es una caricatura, muy buena, pero caricatura."

Detalle en [[../concepts/Caricatura-Computacional]].

## Próxima clase

> "La próxima clase voy a llegar con el bug arreglado y haremos el ejercicio de Python. Si me da tiempo dejaré tarea relacionada con EnerHabitat."

EnerHabitat Parte 2 = clase 011.

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/Temperatura-Sol-Aire]]
- [[../concepts/Estado-Oscilatorio-Permanente]]
- [[../concepts/Factor-de-Decremento]]

Conceptos profundizados:

- [[../concepts/Caricatura-Computacional]] — EnerHabitat como caricatura
- [[../concepts/Posicion-Aislante]] — herramienta primaria para evaluar
- [[../concepts/Confort-Adaptativo]] — EnerHabitat reutiliza Humphreys-Nicol
- [[../concepts/Warm-up-Period]] — comparación con oscilatorio permanente

Herramienta nueva:

- [[../tools/EnerHabitat]]

Procedimientos nuevos:

- [[../procedures/Usar-EnerHabitat-Web]]
- [[../procedures/Usar-EnerHabitat-Python]]

## Conexiones

- ← **Anterior:** [[009-AireAcondicionadoSetPoints]] — HVAC en E+
- → **Siguiente:** [[011-EnerHabitatParte2]] — continuación con la demo Python corregida (próxima ingesta)
- → Procedimientos clave:
  - [[../procedures/Usar-EnerHabitat-Web]]
  - [[../procedures/Usar-EnerHabitat-Python]]

## Recursos mencionados

- **enerhabitat.com** — web app del IER.
- **PyPI** — Python Package Index, donde está publicado el paquete.
- **GitHub `enerhabitat/*`** — organización con los repos del proyecto.
- **`pvlib`** — paquete reutilizado para proyección de radiación solar.
- **COMEX** — caso histórico de uso.
- **Fernando** ("Alex") — egresado del IER que lideró la re-implementación con OOP.
- **Tesis de Miriam** — incluye scripts de validación contra E+.
- **Tesis de Eric Iván** — caracterización del consumo eléctrico de viviendas mexicanas (mencionada de paso).

---
title: 007 — Caso Base y Aleros
type: clase
clase: 007
profesor: Guillermo Barrios del Valle
fuente: raw/videos/007CasoBaseAleros.md
fecha_ingesta: 2026-05-02
tags: [clase, caso-base, aleros, comparacion, python, ear-tools, proyecto-final, debugging]
aliases: [Clase 007]
---

# 007 — Caso Base y Aleros

## Metadatos

- **Clase:** 007
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/007CasoBaseAleros.md`
- **Tipo:** Clase práctica con bug no resuelto al final — "happy accident" pedagógico

## Resumen

Clase enfocada en el **flujo de comparación caso base vs variante** y el **workflow del proyecto final** (5 simulaciones: base + 3 estrategias + combinada). Tres bloques principales:

1. **Trampa de la automatización** (XKCD) y workflow del proyecto final.
2. **Crear caso base + variante con aleros**: bug recurrente del piso adiabático que se revierte a Ground; nombrar superficies clave (`vNorte`, `vOeste`, `Techo`); renombrado custom de columnas.
3. **Análisis comparativo en Python**: función de carga `carga_df(f)`, diccionario `{viejo: nuevo}` para nombres, list comprehensions sobre columnas, plot con convención **color = ubicación, estilo = caso**.

Cierre con un bug no resuelto en vivo — las dos simulaciones (con y sin aleros) producen resultados casi idénticos. Profesor: "lo primero que voy a revisar es mi flujo de datos antes de culpar a Energy Plus."

> "Hoy creo que no debí haber dado clases — esto está siendo muy caótico. Me dio el modo pre-vacacional."

A pesar del caos, contenido valioso: el bug ilustra el método de debugging y refuerza el principio de revisar primero el flujo antes de cuestionar el motor.

## Trampa de la automatización (XKCD)

El profesor abre con el cómic **XKCD "Automation"** (https://xkcd.com/1319/):

> Pensamos que automatizar nos ahorra tiempo. Pero a menudo escribir el script toma más tiempo del que ahorra — y a veces no termina.

Aplicación al taller:

- **Sí automatizar**: cosas que harás muchas veces. `ear_tools` es la única automatización que el profesor mantiene porque la usa todos los días.
- **No automatizar**: cosas one-shot. El bug del 29-feb en `ear_tools` sigue sin parchearse — "empezar a considerar todas las opciones que el usuario puede tener" es la fuente de complejidad.

Cuando se decide automatizar, conviene:

- Empezar con la solución manual primero (entender el problema).
- Convertirlo en función pequeña reusable (`carga_df(f)`).
- Solo después escalar a paquete.

## Bug recurrente — piso adiabático que se revierte a Ground

Síntoma: tras un cambio geométrico en FloorspaceJS, al hacer Run → severe `Construction <piso> missing material assignments`.

Causa: el cambio geométrico hace que Open Studio "rehaga" el modelo internamente. En el proceso:

- La condición de frontera del piso se revierte de `Adiabatic` a `Ground`.
- Como el slot `Ground Contact → Floor` del Construction Set está **vacío**, el piso queda sin construction → severe.

### Workaround del profesor

Sobre-definir el slot `Ground Contact → Floor` en el Construction Set con un sistema constructivo válido (ej. `Concreto_15cm`). Esto:

- Evita el severe cuando la condición se revierte.
- **Pero introduce un riesgo silencioso**: si la condición termina como Ground (no Adiabatic), E+ usa la temperatura del ground por default (~18 °C) — la simulación corre pero **el resultado es incorrecto sin error**.

> "Esa es la importancia de revisar el `.err`. No me había pasado que algo cambiara, pero ahora la otra es que para que no falle, lo cambio a adiabático otra vez. Verde quiere decir que está definido por default."

**Buena práctica**: tras cualquier cambio geométrico, revisar:

1. El `.err` (no debería haber `Ground` warnings inesperados).
2. La pestaña Spaces → Surfaces, columna `Outside Boundary Condition` del piso (debe ser `Adiabatic`).
3. Confirmar el render por boundary (rojo = adiabatic).

Detalle en [[../procedures/Debuggear-Simulacion-OpenStudio]].

## Workflow del proyecto final — 5 simulaciones

> "Para el proyecto final el plan es: 3 estrategias individuales + caso base + caso combinado = **5 simulaciones**."

Estructura:

| OSM | Contenido |
|-----|-----------|
| `005_caso_base.osm` | Modelo de referencia, sin estrategias |
| `006_estrategia_1.osm` | Caso base + estrategia 1 (ej. color) |
| `007_estrategia_2.osm` | Caso base + estrategia 2 (ej. aleros) |
| `008_estrategia_3.osm` | Caso base + estrategia 3 (ej. aislamiento) |
| `009_combinado.osm` | Caso base + las 3 estrategias |

> "Si hacen un cambio de última hora a su modelo, y ese cambio está en el caso base, lo van a tener que pasar a todas. Por eso es súper importante que cuiden que su caso base esté súper bien revisado **antes de ramificar**."

Checklist completo del caso base en [[../concepts/Caso-Base]]. Concepto general de estudio paramétrico en [[../concepts/Estudio-Parametrico]].

## NO copiar OSM en el Explorador

> Trampa: tomar `005_caso_base.osm` con `Ctrl+C/V` en Finder/Explorer.

Resultado: solo se duplica el `.osm` — **el folder hermano no se copia**, y con él se pierden los measures (Add Output Variable, Create CSV Output) configurados.

**Vía correcta**:

1. Abrir Open Studio con el caso base.
2. `File → Save As → 006_estrategia_X.osm`.
3. Open Studio crea el folder hermano con todos los measures.

> "Es algo que cuando tengan ya más trabajo, va a arruinarles su simulación — todos los measures los van a perder."

Refuerzo de la regla en [[../procedures/Estructura-Proyecto-Simulacion]].

## Variables a pedir para análisis bioclimático

El profesor recomienda como **mínimo**:

| Variable | Por qué |
|----------|---------|
| `Site Outdoor Air Drybulb Temperature` (`TO`) | Referencia obligatoria — los modelos de confort dependen del clima exterior |
| `Zone Mean Air Temperature` (por zona) | Variable principal del análisis |
| `Surface Outside Face Incident Solar Radiation Rate per Area` (sobre cada ventana) | Para evaluar el efecto de aleros |
| `Surface Outside Face Incident Solar Radiation Rate per Area` (sobre el techo) | Proxy de radiación global horizontal — referencia |

Variables adicionales útiles:

- `Zone Operative Temperature` — captura el efecto radiativo local que el T del aire no detecta. Recomendado cuando hay ventanas grandes o sol incidente directo en el interior.
- `Zone Windows Total Heat Gain Rate` — radiación que **entra a la zona** después de transmitancia/reflectancia/absortancia (distinta a la incidente sobre el exterior).

> "La radiación incidente sobre la ventana, ya hay una fracción que se refleja y que se absorbe — transmitancia, absortancia, reflejancia. Pero como vamos a hacer un estudio de **sombramiento**, la radiación incidente sobre la superficie está bien."

Catálogo completo en [[../concepts/Variables-Output-EnergyPlus]].

## Nombrar superficies clave

Para que las variables de output sean legibles, asignar nombres descriptivos a las sub-superficies y al techo:

1. Pestaña **Geometry → 3D View**, identificar la superficie (ej. `Face 7`).
2. Verificar orientación con la línea **verde** (norte) y **roja** (este).
3. Pestaña **Spaces → Sub Surfaces** → renombrar `Face 7` a `vNorte`, `Face 14` a `vOeste`, etc.
4. Pestaña **Spaces → Surfaces** → renombrar el techo a `Techo`.

Las variables solicitadas con `Key Value` específico (no `*`) usarán estos nombres → en el SQL aparecen como columnas `VNORTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]`.

> **Bug observado**: cambios geométricos pueden borrar los nombres custom y revertirlos a `Face N`. Si una variable no aparece en el SQL después del Run, primero verificar que el nombre de la superficie sigue siendo el esperado.

## Variantes y métricas — definir antes de ramificar

> "Si voy a calcular una **métrica** de fracción de radiación, tengo que decir 'voy a calcular la fracción de radiación' y como es fracción quiere decir que va dividida por otra. Entonces tendría yo que calcular dos variables. Por eso siempre van a hacer su caso base, sacan **todas las métricas**, se aseguran que tienen, y entonces ahí aplican las estrategias bioclimáticas."

Distinguir:

- **Variable**: dato directo de la simulación (T, radiación).
- **Métrica**: cálculo derivado (fracción, grados-hora, % en confort).

Las métricas pueden requerir varias variables. Si se ramifica antes de tener todas las variables, hay que reconfigurarlas en cada variante — error humano garantizado.

## Análisis comparativo en Python

Procedimiento detallado en [[../procedures/Comparar-Simulaciones-Python]].

### Función de carga reutilizable

> Anti-patrón: copiar la celda de carga 5 veces para 5 simulaciones — fuente de errores.

```python
NOMBRES = {
    "VNORTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]": "IS_vNorte",
    "VOESTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]": "IS_vOeste",
    "TECHO:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]":  "IS_techo",
    # ...
}

def carga_df(f):
    df = read_sql(f, alias=False).data
    df.rename(columns=NOMBRES, inplace=True)
    return df
```

### Bug confesional del profesor

> "Un error común: yo no recibo esta `f`. Está definida adentro y aunque la reciba siempre va a ser la misma. No saben cuántas veces me ha pasado y a veces no me doy cuenta — es lo peligroso."

```python
def carga_df(f):
    f = "../OSM/005_caso_base/run/eplusout.sql"  # ❌ ignora el parámetro
    return read_sql(f).data
```

### Construir el diccionario sin escribir todo

```python
df = read_sql(F).data
nombres = {col: col for col in df.columns}  # dict comprehension
print(nombres)
```

Pegar la salida en una celda nueva, dejar solo las columnas de interés, editar los valores.

> Tip: `df.rename` **no falla** si una llave del diccionario no existe — útil para reusar el mismo diccionario en variantes con columnas distintas, peligroso porque enmascara typos.

### Filtrado de columnas con list comprehension

```python
cols_T  = [c for c in df.columns if c.startswith("T_")]
cols_IS = [c for c in df.columns if c.startswith("IS_")]
```

Por esto se eligen prefijos consistentes (`T_`, `IS_`) — facilitan iteración.

### Plot con convención color/estilo

> "**No hagan dobles ejes Y.** Si tienen 6 líneas, color = ubicación, estilo = caso. Las soluciones suelen ser bien sencillas — la gente quiere irse a algo elaborado."

Patrón:

| Aspecto visual | Significado |
|----------------|-------------|
| **Color** | Variable comparable (rojo = este, azul = oeste) |
| **Línea sólida** | Caso base |
| **Línea dasheada** | Variante |
| **Negro punteado** | T exterior (referencia universal) |

```python
ax[0].plot(base.T_este,   label="este (base)",  color="red", linestyle="-")
ax[0].plot(alero.T_este,  label="este (alero)", color="red", linestyle="--")
```

## Lectura de la trayectoria solar a partir de los datos

Mirando las series temporales de radiación incidente sobre ventanas norte y oeste, se infiere la posición del sol sin etiquetas:

| Patrón observado | Inferencia |
|------------------|------------|
| Joroba baja y plana, simétrica al mediodía | **Norte** (solo difusa) |
| Pico de mañana, cae a difusa después | **Este** |
| Difusa hasta el mediodía, pico de tarde | **Oeste** |
| Pico simétrico al mediodía, máximo absoluto | **Horizontal (techo)** |

> "Ahí se ve clarísimo cuál es la superficie oeste — porque le pega el sol en la tarde."

Aplicación a validación: si una superficie etiquetada como "Norte" muestra un pico de tarde en el simulador, hay un error de orientación o de etiquetas. **La trayectoria solar no miente** — un patrón inesperado es señal de bug.

Detalle en [[../concepts/Trayectoria-Solar]].

### Implicación para el diseño bioclimático

> "Por eso no queremos ventanas este/oeste. Las ventanas este/oeste son bien difíciles de proteger — el sol viene bajo y oblicuo. Queremos ventanas norte y sur."

- **Sur** (hem. norte): sol alto al mediodía → alero corto efectivo.
- **Norte**: solo difusa → no requiere sombreamiento.
- **Este/Oeste**: sol bajo y oblicuo → alero horizontal no protege; se requieren parteluces o estrategias adicionales.

## Anécdota — Paloma y la cafetería del IER (validación de caricatura)

Hace ~7 años, una alumna llamada Paloma (hoy hace simulaciones desde la **secundaria** en algún lugar del mundo, "imagínate") simuló una protección solar compleja del IER:

1. **Versión detallada**: cada listón de la rejilla dibujado en SketchUp.
2. **Versión simplificada**: una superficie equivalente con **transmitancia** que reproduce el área abierta entre listones.

Resultado de la comparación: diferencia **<2%** en radiación recibida en la superficie de medición.

> "Bueno, dijimos: entonces hacemos estas con toda la confianza. En Energy Plus, sí puedes simular si se pierde porque hay muchas interacciones, pero **no mucho**."

Conclusión: las **caricaturas bien construidas** preservan el orden y magnitud del efecto bioclimático. Justifica usar simplificaciones (alero equivalente, transmitancia agregada) en estudios paramétricos sin perder validez de las comparaciones relativas.

Refuerzo en [[../concepts/Caricatura-Computacional]].

## Caricatura adicional — el rayo de luz que pega a una persona

> "Si yo tengo un rayo de luz que ingresa, calienta todo el espacio en Energy Plus. Pero si ese rayo de luz me pega a mí, mi temperatura va a ser diferente."

Recordatorio del principio de [[../concepts/Radiacion-Interior-Distribuida]]: E+ distribuye la radiación que entra **uniformemente en la zona** — no concentra el efecto sobre un ocupante específico.

Para evaluar **disconfort radiativo local** (ej. una persona sentada cerca de una ventana al sol):

- Pedir `Zone Operative Temperature` o `Zone Mean Radiant Temperature`.
- Configurar **sensores virtuales de confort** en E+ (no se ve en el taller, pero es la herramienta para análisis fino).

> "El confort se basa en temperatura operativa. Y la temperatura operativa se basa en temperatura radiante. Entonces si es radiación directa incidente sobre un espacio, eso puede hacer que esos sensores virtuales se disparen."

Detalle en [[../concepts/Temperatura-Operativa]].

## El bug no resuelto al final de la clase

Dos simulaciones (caso base sin aleros vs variante con aleros) producen resultados **casi idénticos** en T interior y radiación. El profesor sospecha:

1. **Primero verificar el flujo de datos**: ¿estoy cargando el SQL correcto en cada variable? ¿la función `carga_df` recibe paths distintos?
2. **Segundo, verificar que el cambio llegó al IDF**: hay precedente de cambios en FloorspaceJS que no se traducen al IDF correctamente.
3. **Solo al final culpar a Energy Plus** — es lo menos probable.

> "Es bien fácil desconfiar de Energy Plus, pero la mayoría de las veces son errores del usuario."

Pista que el profesor identificó: en el SQL salen **5 variables** en lugar de las 6 esperadas — el measure de la `Surface Outside Face Incident Solar Radiation` sobre el techo no encontró el nombre `Techo`. Causa: el bug de FloorspaceJS borró el nombre custom al hacer un cambio geométrico.

> "Si me hubiera fijado en el output, hubiera visto que no encontró esa variable. Y `rename` no marca error cuando no existe — desde ahí me hubiera dado cuenta también."

La clase termina sin resolver — el profesor tiene un compromiso institucional. Lección pedagógica:

1. Siempre verificar **número de columnas esperadas** en el SQL.
2. Si una variable no aparece, primero **verificar el nombre de la superficie** (puede haber sido borrado por el bug).
3. **No confiar en `rename`** — un typo enmascara errores silenciosamente.

Procedimiento sistemático de debug en [[../procedures/Debuggear-Simulacion-OpenStudio]].

## Tarea (opcional, vacaciones)

> "No voy a dejar tarea — son vacaciones. Si algún equipo no le salió la tarea anterior, entréguela."

Sugerencia opcional: comparar variantes de aleros para una misma ventana orientada al oeste:

- Solo alero horizontal con PF=1.
- Solo parteluces verticales con FF=1.
- Combinado (alero + parteluces).

Cuantificar la diferencia en radiación incidente y T interior.

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/Caso-Base]]
- [[../concepts/Estudio-Parametrico]]
- [[../concepts/Trayectoria-Solar]]

Conceptos profundizados:

- [[../concepts/Caricatura-Computacional]] — caricatura "rayo de luz que pega a una persona"
- [[../concepts/Temperatura-Operativa]] — sensores virtuales de confort
- [[../concepts/Radiacion-Interior-Distribuida]] — refuerzo del límite
- [[../concepts/Variables-Output-EnergyPlus]] — `Zone Windows Total Heat Gain Rate`
- [[../concepts/Mensajes-EnergyPlus]] — bug del piso adiabático y nombres de superficies tras cambios geométricos
- [[../concepts/Limpiar-Geometria]] — bug FloorspaceJS al rehacer el modelo

Procedimiento nuevo:

- [[../procedures/Comparar-Simulaciones-Python]]

## Conexiones

- ← **Anterior:** [[006-DosZonasTermicasVentanasAleros]] — agregar ventanas y aleros
- → **Siguiente:** _008-ShadingVentanas_ — protecciones solares en ventanas (cierre del taller)
- → Procedimientos clave:
  - [[../procedures/Comparar-Simulaciones-Python]]
  - [[../procedures/Debuggear-Simulacion-OpenStudio]]
  - [[../procedures/Estructura-Proyecto-Simulacion]]

## Recursos mencionados

- **XKCD #1319 "Automation"** — la trampa de la programación.
- **PhD Comics** ("Piled Higher and Deeper") — el otro cómic del profesor.
- **Paloma** (ex-alumna del grupo) — caso de validación alero detallado vs equivalente con transmitancia.
- **Bug del 29-feb en `ear_tools`** — sigue sin parchear.
- **OpenStudio Coalition** — issue tracker para reportar bugs como el del piso adiabático.

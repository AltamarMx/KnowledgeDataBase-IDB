# 007 — Caso Base y Aleros

## Metadatos
- **Clase:** 007
- **Título:** Caso Base y Aleros
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** flujo de trabajo caso base, gestión de variantes, variables de radiación solar, nombrado de superficies, análisis comparativo con Python, protecciones solares este-oeste, radiación difusa vs directa

---

## Resumen

Clase enfocada en establecer el **flujo de trabajo correcto para estudios paramétricos**: definir completamente un caso base (geometría, materiales, Construction Sets, variables de salida) antes de generar variantes. Se agrega la variable de **radiación solar incidente sobre superficies** para evaluar el efecto de protecciones solares, se nombran las ventanas y el techo para poder rastrearlas en el análisis, y se construye en Python un flujo de **carga y comparación de múltiples simulaciones**. Se descubre un bug de FloorSpaceJS (pierde condiciones de frontera al hacer cambios geométricos). Se muestra que las **ventanas este-oeste son difíciles de proteger** con aleros, reforzando la preferencia por orientaciones norte-sur.

---

## 1. Establecer el caso base

### Principio fundamental

Antes de crear variantes (estrategias bioclimáticas), asegurar que el **caso base** esté completamente definido:
- Geometría finalizada
- Sistemas constructivos verificados (orden de capas, propiedades)
- Variables de salida configuradas (todas las que se vayan a necesitar)
- Simulación corriendo sin errores

**Razón:** cada cambio en el caso base debe replicarse en todas las variantes. Para el proyecto final se tendrán ~5 simulaciones (base + 3 estrategias individuales + todas combinadas). Un cambio tardío en el caso base se multiplica por 5.

### Save As, nunca copiar carpetas

- **No hacer:** copiar la carpeta del .osm desde el explorador → se pierde la carpeta de Measures
- **Sí hacer:** abrir en Open Studio → File → Save As → nuevo nombre
- Convención sugerida: `005-CasoBase.osm`, `006-Protecciones.osm` (número incremental)

---

## 2. Variables de radiación solar incidente

### La variable clave

`Surface Outside Face Incident Solar Radiation Rate Per Area` (W/m²) — disponible en el archivo RDD.

### Radiación incidente vs radiación transmitida

| Concepto | Qué mide |
|----------|----------|
| Radiación incidente sobre superficie exterior | Energía solar total (directa + difusa) que llega a la cara exterior |
| Radiación que ingresa a la zona térmica | Lo que realmente cruza el vidrio (descontando reflectancia y absorptancia) |

- Si el estudio es de **sombramiento** → medir radiación incidente (antes del vidrio)
- Si el estudio es de **ganancia térmica interior** → medir lo que entra a la zona

### Nombrar superficies para el análisis

- Renombrar sub-superficies (ventanas) con nombres descriptivos: `V_norte`, `V_oeste`
- Renombrar superficies de interés: `Techo`
- Al agregar variables de salida con `Add Output Variable`, usar el nombre de la superficie en vez de `*` (asterisco pone todas las superficies expuestas al sol)
- **Cuidado:** si el nombre no existe, el Measure no marca error — simplemente no reporta nada

---

## 3. Bug de FloorSpaceJS: pérdida de condiciones de frontera

Al hacer **cambios geométricos** en FloorSpaceJS (agregar/quitar overhangs, modificar componentes), el editor regenera la geometría y puede perder:
- **Condiciones de frontera** del piso (Ground → Adiabatic o viceversa)
- **Nombres personalizados** de superficies

### Cómo detectarlo

- Revisar el archivo ERR: si el piso cambió de Ground a otra condición, aparecerá un warning sobre temperatura de piso asumida (18°C por default)
- Revisar visualmente en la pestaña Surfaces que las condiciones de frontera sean correctas

### Workaround

1. Después de cada cambio geométrico, verificar condiciones de frontera
2. Si se perdieron, reasignar manualmente (ej. cambiar a Adiabatic)
3. Asignar Construction Set a las superficies antes de cambiar la condición, para que no pierdan su sistema constructivo al cambiar de vuelta

---

## 4. Análisis comparativo con Python

### Estructura para múltiples simulaciones

```python
# Función de carga reutilizable
def carga_datos(f):
    df = read_sql(f, alias=True)
    df.rename(columns=nombres, inplace=True)
    return df

sin_v = carga_datos("../005-CasoBase/run/eplusout.sql")
con_v = carga_datos("../006-Protecciones/run/eplusout.sql")
```

### Renombrado de columnas con diccionario

- Generar diccionario automáticamente con dict comprehension:
  ```python
  nombres = {col: col for col in df.columns}
  ```
- Editar manualmente las columnas de interés (ej. `Is_V_norte`, `Is_V_oeste`, `Is_Techo`)
- `df.rename(columns=nombres, inplace=True)` — si una columna no existe en el diccionario, no da error

### Convención de prefijos para variables

| Prefijo | Significado |
|---------|-------------|
| `Ti_` | Temperatura interior (zona) |
| `Is_` | Radiación solar incidente sobre superficie |

- Permite filtrar variables por tipo:
  ```python
  teis = [col for col in df.columns if "Ti_" in col]
  ises = [col for col in df.columns if "Is_" in col]
  ```

### Visualización comparativa

- **Color** identifica la variable (azul = zona este, verde = zona oeste)
- **Estilo de línea** identifica el caso (sólida = sin protección, punteada = con protección)
- Usar `sharex=True` en subplots para que el zoom se aplique a todas las gráficas
- Temperaturas y radiación en subplots separados (unidades diferentes)

---

## 5. Protecciones solares y orientación

### Ventanas este-oeste: difíciles de proteger

- Un overhang horizontal protege bien cuando el sol viene de arriba (ángulo alto)
- En fachadas **este y oeste**, el sol llega con ángulo bajo (amanecer/atardecer) → el alero horizontal es poco efectivo
- **Evidencia del análisis:** la ventana oeste sin protección muestra un pico claro de radiación directa por la tarde; con protección (factor 1), la reducción es limitada

### Preferencia norte-sur

- **Ventana norte:** recibe principalmente radiación difusa (excepto alrededor del solsticio de verano en zona intertropical)
- **Ventana sur:** el sol llega con ángulo alto → los overhangs son muy efectivos
- Recomendación de diseño: maximizar ventanas norte-sur, minimizar este-oeste

### Radiación difusa vs directa

- En la mañana, la fachada oeste recibe **solo radiación difusa** (igual que la norte)
- En la tarde, la fachada oeste recibe **radiación directa + difusa** → pico pronunciado
- La radiación difusa es omnidireccional → las protecciones la reducen poco

---

## 6. Sobre automatización (xkcd)

- Referencia al cómic xkcd "Is It Worth the Time?" — el peligro de pasar más tiempo automatizando que lo que se ahorra
- Automatizar vale la pena cuando: se repite muchas veces, es un paquete reutilizable (como `ear_tools`)
- No vale la pena cuando: es un proyecto único, una sola vez
- Equilibrio: funciones de carga de datos y renombrado sí vale la pena; sobre-automatizar la detección de columnas probablemente no

---

## Conceptos clave

- **[[Superficies-de-Sombramiento]]** — efectividad limitada en fachadas este-oeste
- **[[Ventanas]]** — orientación y su relación con la protección solar
- **[[Simulacion-Energetica]]** — caso base como punto de partida de estudios paramétricos

Conceptos previos referenciados: [[Condiciones-de-Frontera]], [[Sistemas-Constructivos]], [[Zona-Termica]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · FloorSpaceJS · IDF Editor

## Conexiones

- **Anterior:** [[006-DosZonasTermicasVentanasAleros]] — Modelo de dos zonas que aquí se usa como punto de partida
- **Siguiente:** [[008-ShadingVentanas]] — Profundización en estrategias de sombreado

## Tarea

Dos ejercicios para entregar al regresar de vacaciones (viernes de la siguiente semana):
1. Se pueden usar las simulaciones propias de los estudiantes
2. No se especificaron detalles exactos en la transcripción

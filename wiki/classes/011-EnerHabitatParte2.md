---
title: 011 — EnerHabitat (Parte 2)
type: clase
clase: 011
profesor: Guillermo Barrios del Valle
fuente: raw/videos/011_EnerHabitat_Parte2.md
fecha_ingesta: 2026-05-02
tags: [clase, enerhabitat, asistente-virtual, ia, rag, paramétrico, python, cierre-taller]
aliases: [Clase 011]
---

# 011 — EnerHabitat (Parte 2)

## Metadatos

- **Clase:** 011
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/011_EnerHabitat_Parte2.md`
- **Tipo:** Clase de cierre del taller. Tres bloques: asistente virtual del curso + resolución del bug de la 010 + demo paramétrica con happy accidents.

## Resumen

Clase de **cierre del taller** con tres bloques principales:

1. **Asistente virtual del curso** — presentación del prototipo funcional. Stack: OpenCode + Raspberry Pi + Claude API + Telegram. Filosofía: complementar (no sustituir) la docencia. Concepto de "minar errores" como gamificación.

2. **Resolución del bug de la 010** — la actualización a pandas 3.0 hizo inmutables los DataFrames retornados; EnerHabitat sobreescribía esos resultados durante la iteración. Fix: parámetro para activar resultados writeable. Lección: dependencias upstream pueden romper el código.

3. **Demo paramétrica en Python** — for loop sobre absortancia con simulaciones AC, intentando reproducir el efecto del color en consumo. Varios "happy accidents" en vivo: bug de configuración global de h_c, susto del primer time step (radiación = 0), confundir `wall.layers` con `wall.absorptance`. Lección final: **confiar en la física** — si un resultado no suena lógico, hay un bug.

Cierra con la **tarea final** del taller (EnerHabitat desde Python, entrega viernes siguiente) y anuncio del comunicado para invitar al chat de Telegram.

> "Va dos clases que no me sale. La verdad es que sí me cuesta trabajo pensar y razonar y dar clase al mismo tiempo."

## Asistente virtual del curso (RAG)

Detalle completo en [[../concepts/Asistente-Virtual-RAG]].

### Por qué se construyó

> "La LIER inventó esta materia. Otras universidades replican. Pero ¿quién les enseña a simular? Habemos muy poquitos."

- **Pocos profesores** saben simular E+/Open Studio para enseñar diseño bioclimático.
- El profesor no escala — alta carga, alta demanda de los estudiantes.
- IA puede **multiplicar la capacidad pedagógica** sin reemplazar al docente.

### Por qué un corpus curado, no un LLM general

> "Cada vez menos, pero todavía se equivocan mucho con Energy Plus. En transferencia de calor sí saben bastante bien."

LLMs comerciales **alucinan en E+** por poca data pública en su entrenamiento. Solución del grupo: **corpus curado** (transcripciones de clase + scripts + libretas + artículos seleccionados) + obligar al asistente a contestar **solo desde ese corpus**.

### Stack técnico

| Componente | Función |
|------------|---------|
| **Raspberry Pi 8GB** | Hardware aislado en casa del profesor |
| **OpenCode** (Anthropic) | Framework agéntico — escribe/lee archivos en su entorno |
| **Claude Opus** | LLM remoto vía API (la Raspberry no aguanta correr LLM local) |
| **Telegram bot** | Interfaz de chat |

> "OpenCode es peligroso ponerlo en tu computadora — puede borrar cosas o leer cosas y compartirlas. Por eso debe estar aislado en la Raspberry."

### Filosofía — complementar, no sustituir

> "La responsabilidad de aprender, qué temas decidir aprender a profundidad y con qué capacidad de autosuficiencia — depende de ustedes."

Casos OK: preguntas conceptuales fuera de horario, recordar dónde se vio un tema, ejecutar pasos repetitivos.

Casos NO deseados: resolver el proyecto final sin entender, copy-paste sin asimilar.

### Errores observados en vivo en clase

El asistente repite errores del corpus:

- `iertools` transcrito como "orejas" por error de ASR.
- Ecuación de transferencia de calor mal escrita (sin $\rho c_p$ en alguna transcripción).
- Otros bugs sutiles.

→ El corpus necesita **revisión continua**.

### "Minar errores" como gamificación

Idea introducida en la clase: estudiantes que detecten errores del asistente:

1. Reportarlos a través del chat.
2. El asistente los registra (capacidad de escritura limitada).
3. El profesor revisa y **otorga puntos** como recompensa.

> Analogía con minería de Bitcoin: "Aquí van a minar errores."

Beneficio: mejora el corpus + engancha a estudiantes con aprendizaje activo (encontrar errores implica entender los conceptos).

### Privacidad — caveats actuales

> "Las conversaciones en Telegram grupal **no son privadas**. Yo puedo ver lo que escriben."

Estado prototipo:

- Telegram grupal no encripta E2E.
- El bot tiene memoria por default.
- El profesor puede ver los logs.

Plan futuro: encriptación, sin memoria, perspectiva de género (filtros contra sesgos).

## Resolución del bug de la 010

En la clase 010, la demo Python falló con `assignment is read only`.

### Causa

Actualización **pandas 2.x → 3.0** hizo inmutables los DataFrames retornados por algunos métodos. EnerHabitat sobreescribía esos resultados durante el proceso iterativo del solver → fallaba.

### Fix

Agregar un parámetro al método para activar resultados **writeable**. "Realmente no fue un error garrafal — me sirvió para revisar la documentación y mejorarla."

### Lección — dependencias upstream

> "Estábamos usando pandas 2.x y ahora 3.0 tiene esta restricción. Nada más era ponerle un `True` en algún lado."

Aplicable a cualquier proyecto que dependa de paquetes de terceros:

- **Tests** que detecten regresiones cuando se actualiza una dependencia.
- **Pin** de versiones en producción (`uv.lock` ayuda).
- **Lectura de release notes** antes de actualizar paquetes core.

## Recap teórico de EnerHabitat

Recap rápido de lo visto en [[010-EnerHabitatParte1]]:

### PDE que resuelve

$$
\rho c_p \frac{\partial T}{\partial t} = -k \frac{\partial^2 T}{\partial x^2}
$$

con condiciones de frontera:

- **Exterior** ($x=0$): $q''_{LWR} + q''_{conv} + I \alpha = -k \partial T / \partial x$ — encapsulada en [[../concepts/Temperatura-Sol-Aire]].
- **Interior** ($x=L$): solo flujo convectivo (no LWR — solo una pared).

### Algoritmo numérico

- **Volúmenes de control** = diferencias finitas en 1D.
- **Esquema semi-implícito** (más robusto que explícito; explícito requiere número de Fourier estable).
- **TDMA** (Tridiagonal Matrix Algorithm) para resolver el sistema lineal en cada paso.

### Cuarto idealizado

| Parámetro | Valor |
|-----------|-------|
| Largo del cuarto ficticio | 2.5 m |
| Discretización de la pared | 200 elementos |
| Paso temporal | 600 s (10 minutos) |
| $h_{c, exterior}$ | 13 W/m²K (default) |
| $h_{c, interior}$ | 8.6 W/m²K |
| Pared opuesta | Adiabática |

Coeficientes vienen de NOM-008 / NOM-020 — son configurables. Detalle en [[../concepts/Caricatura-Computacional]].

## API verificada (versión 0.1.9)

Después del fix, el flujo funciona. Patrón verificado en clase:

```python
import enerhabitat
import pandas as pd
import matplotlib.pyplot as plt

# 1. Crear el wall (geolocaliza con el EPW)
wall = enerhabitat.Wall(epw_file="EPW/cuernavaca.epw")

# 2. Configurar geometría
wall.azimuth     = 90      # Norte=0, Este=90, Sur=180, Oeste=270
wall.tilt        = 90      # 90 = muro vertical
wall.absorptance = 0.3     # 0-1

# 3. Definir capas (orden ext → int)
wall.layers = [
    ("adobe", 0.30),       # adobe de 30 cm
]

# 4. Día representativo
wall.set_day(month=4, year=2026)  # abril, año arbitrario

# 5. Calcular temperatura sol-aire (forzamiento exterior)
wall.tsa()

# 6. Resolver
wall.solve()              # sin AC — la T flota
# o
wall.solve_ac()           # con AC — T constante en setpoint
```

Procedimiento detallado en [[../procedures/Usar-EnerHabitat-Python]].

### Detalle: `materials.ini` auto-detectado

EnerHabitat busca `materials.ini` en el subdirectorio de la libreta automáticamente. Si no se encuentra, hay que pasar la ruta explícitamente. Esto **no estaba documentado** — descubierto en la clase.

### `solve()` vs `solve_ac()`

| Método | Resultado | Variables clave |
|--------|-----------|-----------------|
| `wall.solve()` | T interior flota libre | `T_int` (serie temporal) |
| `wall.solve_ac()` | T interior constante en setpoint adaptativo | `cooling_energy`, `heating_energy` (J) |

> "EnerHabitat pone el set point en el límite superior de confort (Humphreys-Nicol). Pensando en climas cálidos."

Para climas fríos: el setpoint actual no es óptimo — pendiente de mejora (issue documentado en GitHub).

### Configuración global de coeficientes

```python
enerhabitat.config.h0 = 13   # configurar h_c exterior global
```

> **Bug observado en clase**: cambios a `config.h0` después de calcular `tsa()` **no aplican** retroactivamente. Hay que configurar **antes** de llamar `tsa()`.

## El "susto feliz" del primer time step

Caso real en clase: el profesor cambia la absortancia de 0.3 a 0.6 y mira el primer valor de la T sol-aire — **no cambia**. Sospecha bug.

Después de minutos de diagnóstico: **la radiación incidente al inicio del día es 0** (típicamente las 00:00). Como $T_{sa} = T_{aire} + \alpha I / h_c$, con $I = 0$ cualquier $\alpha$ produce el mismo resultado en ese punto.

> "Es el primer time step del día. La radiación solar al inicio es cero. Cualquier cambio en la absortancia no tiene efecto. Se va a ver cuando empieza a haber radiación solar."

**Lección**: para verificar efectos de cambios en α o $h_c$, **mirar el día completo** (especialmente horas con sol), no solo el primer valor. Detalle en [[../concepts/Temperatura-Sol-Aire]].

## Estudio paramétrico en Python — caso de uso central

> "Esto sí es lo que no puedo hacer en la web app. Aquí parametrizo y resuelvo todo de una."

### Patrón básico

```python
import numpy as np

absortancias = np.linspace(0.01, 1.0, 100)  # 100 valores
consumo_total = []
consumo_cool  = []
consumo_heat  = []

for alpha in absortancias:
    wall = enerhabitat.Wall(epw_file="EPW/cuernavaca.epw")
    wall.azimuth     = 90
    wall.tilt        = 90
    wall.absorptance = alpha
    wall.layers      = [("concreto", 0.15)]
    wall.set_day(month=4, year=2026)
    wall.tsa()
    wall.solve_ac()

    consumo_total.append(wall.cooling_energy.sum() + wall.heating_energy.sum())
    consumo_cool.append(wall.cooling_energy.sum())
    consumo_heat.append(wall.heating_energy.sum())

fig, ax = plt.subplots()
ax.plot(absortancias, consumo_cool, label="enfriamiento")
ax.plot(absortancias, consumo_heat, label="calentamiento")
ax.plot(absortancias, consumo_total, label="total")
ax.set_xlabel("Absortancia α")
ax.set_ylabel("Energía (J)")
ax.legend()
```

Resultado esperado en clima cálido (Cuernavaca):

- **Enfriamiento ↑** con α — más absorción solar → más calor → más AC.
- **Calentamiento ↓** con α — la absorción reduce la necesidad de calefacción.
- **Total** dominado por enfriamiento (Cuernavaca cálida).

## Happy accidents — bugs en vivo y lecciones

### Bug 1: confundir `layers` con `absorptance`

En vivo el profesor hace:

```python
wall.layers = [("adobe", alpha)]   # ❌ alpha aquí es ESPESOR, no absortancia
```

Resultado: estaba variando el espesor del adobe (entre 0.01 y 1.00 m), no la absortancia. Resultados contraintuitivos.

Fix:

```python
wall.absorptance = alpha
wall.layers      = [("adobe", 0.30)]  # espesor fijo
```

> "Todo lo que les dije está mal. Ahí no va la absortancia, aquí va el espesor."

### Bug 2: trabajar con adobe muy grueso

El profesor puso adobe de 1 m por error. Resultado: tan masivo que el comportamiento se vuelve plano (todas las temperaturas muy parecidas) → métricas contraintuitivas.

> "Es que adobe se porta de manera muy especial — tiene mucha masa térmica."

### Lección final: confiar en la física

> "Si yo no tengo una idea de la física, me puedo sobreexplicar resultados raros para justificarlos. Es bien peligroso."
>
> "Si algo no les suena lógico, **confíen en su instinto**. Para confiar en su instinto, tengan la física bien."

Aplicable a cualquier análisis numérico:

1. Predecir cualitativamente el resultado **antes** de correr.
2. Si el resultado contradice la predicción, **revisar el código** primero.
3. Solo si tras múltiples revisiones el resultado se sostiene, cuestionar la física.
4. **Nunca** sobre-explicar un resultado raro para justificarlo.

## Anti-patrones de Python observados

### Referencias compartidas en pandas

> "En Python, si yo digo `df_b = df_a`, esos dos quedan enlazados. Si modifico uno se cambia el otro porque están apuntando a arreglos dinámicos en memoria."

Para copia independiente:

```python
df_b = df_a.copy()   # crea nueva copia, independiente
```

Sin `.copy()`, modificar `df_b` modifica `df_a`. Bug silencioso típico.

### Iteración sobre DataFrames es lenta

> "Iterar un DataFrame para resolver problemas de transferencia de calor es muy lento. Pásenlos a NumPy y aquello vuela."

EnerHabitat originalmente usaba DataFrames internamente → 3 minutos por simulación. Migración a NumPy arrays → 3 segundos.

**Buena práctica**: para cálculos numéricos pesados, **usar NumPy arrays** (no DataFrames). Reservar DataFrames para análisis exploratorio y postprocesamiento.

### Numba para velocidad

EnerHabitat usa **Numba** (compilador JIT que convierte Python a código nativo cercano a C):

- Sin Numba: ~3 minutos por simulación.
- Con Numba: ~3 segundos.

Aplicable a cualquier proyecto Python con cuello de botella en loops numéricos.

### Reproducibilidad frágil de Jupyter

> "Reproducibilidad en libretas Jupyter es bien frágil, bien frágil."

Caso real en clase: una variable `data` quedó en memoria pero no en el código → la libreta corrió aparentemente bien hasta que se hizo Restart-and-Run-All. Con Restart: bug.

**Buena práctica**: hacer **Restart and Run All** periódicamente (recap de [[../classes/005-AnalisisSimulacionesPython]]).

## Anti-patrón crítico — pegar T sol-aire entre walls distintos

> "La temperatura sol-aire pertenece a ese wall específico. Si cambio el color, la orientación o el lugar, cambia."

Caso peligroso:

```python
wall_1.absorptance = 0.3
wall_1.tsa()
wall_1.solve()
df_1 = wall_1.solution

wall_2.absorptance = 0.7   # cambio de color
wall_2.tsa()
wall_2.solve()
df_2 = wall_2.solution

# ❌ Pegar la T sol-aire de wall_1 a wall_2 → bug silencioso
df_2["T_sa"] = wall_1.T_sa   # son distintas
```

Solo es válido pegar la T sol-aire entre walls cuando **comparten** orientación + color + lugar + período (solo difieren en sistema constructivo).

## Cómo se construyó EnerHabitat

> "El genio atrás de esto fue Fer (Fernando), que hizo una estancia y servicio social acá."

- **Diseño OOP** — cada wall encapsula su geolocalización, sistema constructivo, T sol-aire.
- **Reuso de pvlib** — proyección de radiación solar (no reinventar).
- **Numba + NumPy** para velocidad.
- **Publicación en PyPI** para distribución abierta.

## Plan futuro

> "Pendiente: hacer un repositorio con muchísimos casos para dejar el how-to. Y mandar a publicar el paquete a una revista que acepta software científico."

- **Documentar** todos los casos de uso típicos.
- **Publicar** el paquete en una revista que reconoce contribuciones de software (J. Open Source Software, etc.).
- Tomará 2-3 meses adicionales.

## Tarea final del taller

> "Voy a dejar una tarea muy sencilla con EnerHabitat desde Python. Para entregar el viernes de la semana que entra."

Detalles específicos publicados en Classroom. Probablemente:

- Estudio paramétrico de un sistema constructivo.
- Comparación entre 2-3 sistemas con métricas (FD, FD sol-aire, tiempo de retraso).
- Reporte breve con análisis físico.

## Cierre del taller

> "Voy a aprovechar este tiempo para subir todos los videos, actualizar el libro y avisarles que está actualizado hasta esta clase."

- Videos subidos al repositorio del curso.
- "Libro" del curso (sitio web con todas las clases) actualizado hasta la 011.
- Comunicado pendiente con la invitación al chat de Telegram.

## Conceptos derivados

Concepto nuevo:

- [[../concepts/Asistente-Virtual-RAG]]

Conceptos profundizados:

- [[../concepts/Caricatura-Computacional]] — detalles del cuarto idealizado de EnerHabitat (2.5 m, 200 elementos, h_c)
- [[../concepts/Temperatura-Sol-Aire]] — "susto feliz" del primer time step
- [[../concepts/Factor-de-Decremento]] — asociación T sol-aire ↔ wall específico
- [[../tools/EnerHabitat]] — API verificada en versión 0.1.9, fix del bug de pandas 3.0

Procedimientos profundizados:

- [[../procedures/Usar-EnerHabitat-Python]] — flujo verificado, patrón paramétrico, anti-patrones
- [[../procedures/Analizar-Resultados-Python]] — referencias compartidas en pandas, NumPy vs DataFrames

## Conexiones

- ← **Anterior:** [[010-EnerHabitatParte1]] — presentación inicial de EnerHabitat con bug
- → **Siguiente:** _(fin del taller — proyecto final)_
- → Procedimientos clave:
  - [[../procedures/Usar-EnerHabitat-Python]]
  - [[../procedures/Analizar-Resultados-Python]]

## Recursos mencionados

- **OpenCode** (Anthropic) — framework agéntico que corre el asistente del curso.
- **Telegram bot** — interfaz del asistente (privacidad limitada en estado actual).
- **Qwen 3 32B** — modelo open-weights candidato para la versión auto-hosted del asistente.
- **PyPI 0.1.9** — versión estable del paquete EnerHabitat tras el fix de pandas 3.0.
- **Numba** — compilador JIT para Python (50-100× speed-up en loops numéricos).
- **Fernando** — egresado del IER que diseñó la arquitectura OOP de EnerHabitat.
- **pandas 3.0** — la causa del bug de la 010 (resultados inmutables).
- **Tarea final** — detalles en Classroom.

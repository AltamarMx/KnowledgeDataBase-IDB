---
title: 009 — Aire Acondicionado y Setpoints
type: clase
clase: 009
profesor: Guillermo Barrios del Valle
fuente: raw/videos/009_AireAcondicionado_SetPoints.md
fecha_ingesta: 2026-05-02
tags: [clase, hvac, aire-acondicionado, ideal-air-loads, schedules, setpoint, NOM, aislamiento]
aliases: [Clase 009]
---

# 009 — Aire Acondicionado y Setpoints

## Metadatos

- **Clase:** 009
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/009_AireAcondicionado_SetPoints.md`
- **Tipo:** Clase técnica con énfasis en HVAC ideal y crítica a normativas

## Resumen

Clase técnica que introduce **HVAC al modelo** vía Ideal Air Loads (eficiencia 100%, sin ductos). Tres bloques principales:

1. **Por qué el proyecto final NO incluye AC** y crítica a NOM-008/020 + green washing del aislamiento.
2. **Configurar Ideal Air Loads + schedules de setpoint** en Open Studio. Tres modos de uso: T constante, banda de confort, hackear para no calentar/no enfriar.
3. **Análisis Python** con `resample` mensual y gráficas de barras para energía consumida.

Hilo transversal importante: **la posición del aislante** importa en modelos dependientes del tiempo — la NOM no especifica dónde. Anécdota Cool Biz Japón. Crítica a setpoints extremos en oficinas mexicanas.

> "Faltan tres temas: el de hoy son schedules y aire acondicionado en simulaciones."

## Por qué el proyecto final no incluye AC

### Costo de simulaciones

El proyecto final tiene **5 simulaciones** (caso base + 3 estrategias + combinada). Si se incluyera AC, se duplicarían a **10**. Demasiado trabajo para el alcance del semestre.

### Las mejores estrategias bioclimáticas no son las mismas con/sin AC

| Estrategia | Sin AC | Con AC |
|------------|--------|--------|
| Aislamiento térmico | Variable — puede atrapar calor en climas cálidos | **Sí** casi siempre — reduce carga |
| Masa térmica | **Sí** — atenúa picos diurnos | Variable — puede prolongar la operación del AC |
| Ventilación nocturna | **Sí** — disipa calor acumulado | No relevante (AC encendido) |

> "Las mejores estrategias bioclimáticas para AC no son las mejores para sin AC. Por eso el grupo del IER se enfoca en sin AC, donde la vivienda social mexicana realmente vive."

Detalle en [[../concepts/Aire-Acondicionado-Ideal]] sección "Por qué no se incluye AC".

## Crítica a NOM-008 / NOM-020

### Pre-reforma (versión original)

> "Tenía que usarse la normativa, independientemente de si se usaba AC o no. Es uno de los muchos absurdos de la norma."

Exigía aislamiento térmico **siempre**. En climas cálidos sin AC, el aislante puede **atrapar calor interno** y empeorar el confort.

### Post-reforma

Reformulada para "climas cálidos con AC, para ahorrar energía de enfriamiento". Mejor pero aún incompleta — no especifica la **posición** del aislante.

### Conflicto de interés y green washing

> "La industria — los que hacen Panel Rey, los que hacen aislantes — lo ven como una oportunidad y empezaron a impulsar normativas para que la gente las cumpla."

Resultado: una normativa con buena intención **contaminada por intereses económicos**. Hace 10 años la industria del aislante presionó por la NOM **sin entender** la diferencia entre modelos dependientes vs independientes del tiempo.

> "Pregúntenles a los que impulsaban la industria del Panel Rey si tienen noción de transferencia de calor — no tienen idea."

### Aplicable a otras certificaciones

> "El LEED y algunas certificaciones pueden caer en este greenwash. En México las certificaciones casi siempre nacen queriendo beneficiar a alguien económicamente."

Lección: poder **evaluar normativas con criterio físico** (modelo dependiente del tiempo + simulación) es indispensable para no caer en estas trampas.

## Posición del aislante — la asimetría que la NOM ignora

> "En modelo independiente del tiempo (R total), la posición del aislante no importa. **En modelo dependiente del tiempo, sí importa**."

Para una pared con la **misma R total**, la **posición** del aislante (interior, exterior, intermedio) cambia el comportamiento térmico:

- **Aislante exterior + masa interior**: la masa estabiliza la T interior. Mejor en climas con AC y climas extremos.
- **Aislante interior + masa exterior**: la masa "no sirve" para el confort interior. Peor en muchos casos.
- **Aislante intermedio (sandwich)**: compromiso — masa a ambos lados.

Detalle en [[../concepts/Posicion-Aislante]].

### Tarea propuesta

Hacer 3 simulaciones del mismo cubo + clima + R total, variando solo la **posición del aislante**, y comparar grados-hora de disconfort. Verifica empíricamente que la posición sí importa.

## Aire Acondicionado Ideal — Ideal Air Loads

### Qué hace E+ con Ideal Air Loads

> "Si yo le pido 1000 unidades de energía, me va a dar 1000. La eficiencia es 100% y no hay límite de capacidad."

Características:

| Característica | Valor |
|----------------|-------|
| **Eficiencia** | 100% |
| **Capacidad** | Sin límite |
| **Ductos** | No se modelan |
| **Recirculación** | Solo aire interior (equivalente a mini-split) |

Detalle en [[../concepts/Aire-Acondicionado-Ideal]].

### En la realidad

Los AC reales tienen:

- **Capacidad pico** — si la carga supera la capacidad, no se alcanza el setpoint.
- **Eficiencia < 100%** — se mide como SEER, COP. Un AC entrega 3-4 W frío por cada W eléctrico (SEER ~10-15).
- **Toneladas de refrigeración** — unidades comerciales (1 ton ≈ 3.5 kW).
- **Ductos, ventiladores** — pérdidas adicionales.

> "El truco al diseñar AC es dimensionar a la carga pico. Como sistemas fotovoltaicos."

Estos detalles **no se modelan** en el taller — Ideal Air Loads es la caricatura suficiente para evaluar el efecto bioclimático.

## Schedules — el mecanismo central de E+

> "Cualquier carga térmica o setpoint requiere un schedule."

Detalle en [[../concepts/Schedules]].

### Tipos de schedule

| Tipo | Para qué |
|------|----------|
| **Temperature** | Setpoints de heating/cooling |
| **Fraction** | Apertura de ventanas, dimming de luces, ocupación parcial |
| **On/Off** | HVAC encendido/apagado |
| **Number of People** | Ocupación absoluta |
| **Power** | Cargas eléctricas absolutas |
| **Activity Level** | Metabolismo (W/persona según actividad) |

### Resolución temporal

Hasta **60 timesteps por hora** (1 minuto). Eventos sub-minuto se reparten en un minuto.

### Interfaz de Open Studio

- **Default profile** — el horario base, fallback.
- **Run periods** — sobreescritura por rango de fechas.
- **Holidays** — sobreescritura por días específicos.
- **Días de la semana** — distinguir fin de semana.

Para editar el default profile:

- **Doble click** sobre la línea para insertar un corte.
- **Mover el cursor sobre un segmento + escribir** el valor numérico.
- **Doble click** sobre un corte para eliminarlo.

Procedimiento detallado en [[../procedures/Crear-Schedule-Temperatura]].

> "El truco de la interfaz: el `lower limit` y `upper limit` del **tipo** de schedule no son el valor del schedule. El valor real se define con la línea negra."

### Templates de ASHRAE (no para México)

ASHRAE tiene schedules tipo para escuelas, oficinas, hospitales. Para México **no hay equivalente oficial** — el grupo del IER trabaja en caracterizar consumo eléctrico de viviendas mexicanas (tesis de Eric Iván) para construir schedules propios.

## Setpoints — los tres modos de uso

E+ requiere **dos setpoints obligatorios**: heating y cooling. Detalle en [[../concepts/Setpoint]].

### Modo 1: T constante

Heating = Cooling = 20 °C → la zona se mantiene **exactamente a 20 °C**. El AC calienta o enfría como sea necesario.

Aplicación: museos (caso del **MUAC** del UNAM, 9 salas a 20 °C ± humedad para preservar obras de arte).

### Modo 2: Banda de confort

Heating = 20, Cooling = 25 → la zona **flota libre entre 20-25 °C**. El AC solo actúa cuando T sale de la banda.

Aplicación: oficinas, residencias. Compatible con [[../concepts/Confort-Adaptativo]].

### Modo 3: Solo enfriar (México típico)

Heating = −1 °C, Cooling = 25 °C → nunca calienta (T del aire no llega a −1 °C en clima de Cuernavaca); enfría arriba de 25 °C.

> "En México casi nunca hay calefacción. ¿Cómo engaño a Energy Plus? Pongo el heating setpoint en −1 °C — nunca va a llegar."

Truco genérico: **valores extremos del setpoint** que nunca se alcanzan en el clima → deshabilitan ese modo.

## Ejercicio en vivo — los tres casos

El profesor demuestra los tres casos en una zona del modelo:

### Caso 1: T constante a 20 °C

```
heating schedule: 20 °C
cooling schedule: 20 °C
```

Resultado: línea recta a 20 °C todo el año.

### Caso 2: Banda 20-25

```
heating schedule: 20 °C
cooling schedule: 25 °C
```

Resultado: la T flota entre 20-25, se pega arriba en periodo cálido (clima de Cuernavaca → siempre cooling-bound).

### Caso 3: Una zona solo enfría, otra mantiene T constante

Demostración: dos zonas, una con heating = −1 (no calienta), otra con heating = 20.

> "Una va a consumir más energía que otra. Una va a consumir energía de enfriamiento y calentamiento, y la otra solamente energía de calentamiento."

## Modo de plotting cuando una variable es constante

Cuando una serie temporal es **constante** (ej. T mantenida en 20 °C), matplotlib se confunde con el `ylim` automático y el plot se ve "raro".

Workaround:

```python
ax.set_ylim(15, 25)  # forzar límites manuales
```

> El profesor lo descubre en vivo: "matplotlib se equivoca en el zoom cuando una línea no varía."

## Análisis de energía mensual

### Variables de output para AC

```
Zone Ideal Loads Zone Total Cooling Energy [J]
Zone Ideal Loads Zone Total Heating Energy [J]
```

Distinción importante:

- **Energy** (J) — cantidad acumulada, usar para integrales mensuales.
- **Rate** (W) — potencia instantánea, usar para series temporales.

Catálogo en [[../concepts/Variables-Output-EnergyPlus]].

### Resample mensual con pandas

> Pregunta del profesor a la clase: "¿quién pasa a escribir 3 líneas de código?" Nadie pasa — la respuesta es **una línea**.

```python
energia_mes = df["cooling_energy_J"].resample("ME").sum()
```

`resample` es la función poderosa de pandas para series temporales:

- Argumento: frecuencia (`"H"` hora, `"D"` día, `"W"` semana, `"ME"` mes-end, `"YE"` año).
- Operación: `.sum()`, `.mean()`, `.max()`, `.min()`, `.std()`.

> "Quieren hacer series temporales — tienen que estudiar más. Esas cosas se enseñan en todos los cursos de Python del instituto."

Alternativa más rebuscada: `groupby` con extracción de mes — funciona pero menos elegante.

### Gráfica de barras mensual

```python
fig, ax = plt.subplots(figsize=(10, 4))
ax.bar(range(1, 13), energia_mes.values)
ax.set_xticks(range(1, 13))
ax.set_xticklabels(["E", "F", "M", "A", "M", "J", "J", "A", "S", "O", "N", "D"])
ax.set_ylabel("Energía cooling (J)")
```

`ax.bar(x, height)` requiere posiciones X explícitas — usar `range(1, 13)` para los 12 meses.

Patrón esperado en clima cálido: barras altas en abril-mayo (período más cálido), bajas en invierno.

## Anécdota Cool Biz — Japón

> "Las ventajas de tener un emperador, en algunos casos."

El emperador de Japón decretó que se podía **dejar de usar saco y corbata** en oficinas durante el verano. Esto permitió:

- Subir el setpoint de cooling de las oficinas.
- **Ahorrar energía** sin pérdida de confort percibido.
- La gente se adaptó vistiendo más ligero.

Aplicación en México: relajar dress codes en bancos, oficinas, congresos para subir setpoints y ahorrar.

### Crítica a setpoints extremos en México

> "¿Cómo es posible que tengamos que ir a la costa y para entrar al congreso te tengas que poner una chamarra? Es ridículo. Cuántas veces no nos pasa que vas al cine y tienes que llevar chamarra."

Causa: alguien (banquero con saco, ejecutivo con corbata) impone un dress code que requiere AC bajo. El resto de los presentes con vestimenta normal **tienen frío**.

> "Eso es un gap de consumo de energía que se podrían estar ahorrando. Tendría que democratizarse el uso de la energía."

## Sistemas geotérmicos (mención breve)

E+ puede simular **tubos enterrados** (geothermal heat exchangers):

- Verticales (más comunes — taladros profundos).
- Horizontales (requieren mucho terreno).

Requieren la **temperatura del suelo a profundidad** — depende del material (roca, lodo, humedad). El nuevo edificio del IER tiene sensores a 1.8 m que eventualmente permitirán simular geotermia local.

Fuera del alcance del taller.

## Recomendación Mac vs Windows para investigación

> "Para hacer simulaciones de E+, yo prefiero Windows."

Razón: el **IDF Editor** (interfaz tabular para editar IDF directamente) **no existe en Mac**. Para la siguiente materia (Energía en Edificaciones) es necesario.

Soluciones:

- Máquina virtual con Parallels (necesita mucha RAM).
- Boot dual.
- Trabajar en una PC del laboratorio.

En el alcance del taller solo se usa Open Studio → Mac sí funciona.

## Para la próxima clase

> "Les voy a dar clase el miércoles. Va a ser una clase teórica y de Python para EnerHabitat."

EnerHabitat = la siguiente clase (010) y posiblemente (011). Es una herramienta de simulación bioclimática propia del IER.

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/Aire-Acondicionado-Ideal]]
- [[../concepts/Schedules]]
- [[../concepts/Setpoint]]
- [[../concepts/Posicion-Aislante]]

Conceptos profundizados:

- [[../concepts/Caricatura-Computacional]] — Ideal Air Loads como caricatura
- [[../concepts/Sistemas-Constructivos]] — orden de capas y posición del aislante
- [[../concepts/Variables-Output-EnergyPlus]] — variables de cooling/heating energy
- [[../concepts/Confort-Adaptativo]] — setpoint óptimo desde modelo adaptativo

Procedimientos nuevos:

- [[../procedures/Crear-Schedule-Temperatura]]
- [[../procedures/Configurar-Aire-Acondicionado-Ideal]]

## Conexiones

- ← **Anterior:** [[008-ShadingVentanas]] — sombreamiento y métricas de confort
- → **Siguiente:** _010-EnerHabitatParte1_ — herramienta del IER para simulación bioclimática
- → Procedimientos clave:
  - [[../procedures/Configurar-Aire-Acondicionado-Ideal]]
  - [[../procedures/Crear-Schedule-Temperatura]]
  - [[../procedures/Solicitar-Output-Variables-Measures]]

## Recursos mencionados

- **MUAC** — Museo Universitario de Arte Contemporáneo de la UNAM. 9 salas a 20 °C controlada para preservar obras.
- **Cool Biz** (Japón) — política pública del emperador para ahorrar energía relajando dress codes.
- **Tesis de Eric Iván** — caracterización del consumo eléctrico de viviendas mexicanas.
- **Tesis de Miriam** — Airflow Network (siguiente materia).
- **NOM-008**, **NOM-020** — normativas mexicanas de eficiencia energética en edificaciones.
- **Panel Rey** — fabricante de aislante térmico mencionado como ejemplo de presión industrial sobre normativas.
- **LEED** y certificaciones — green washing potencial.
- **EnerHabitat** — herramienta del IER para próxima clase.

# 003 — Mi Primera Simulación

## Metadatos
- **Clase:** 003
- **Título:** Mi Primera Simulación
- **Duración:** ~1h 22min
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** balance de calor interior, modelo de mezclado perfecto, editor de geometría FloorSpaceJS, espacios vs zonas térmicas, condiciones de frontera visuales, materiales, sistemas constructivos, primera simulación completa

---

## Resumen

Clase que cierra la parte teórica con el balance de calor en la superficie interior y el modelo de mezclado perfecto, y luego pasa a la práctica: crear la primera simulación completa en Open Studio, desde dibujar la geometría hasta ejecutar la simulación.

### Balance de calor en la superficie interior

La superficie interior tiene componentes similares a la exterior pero con diferencias importantes:

#### 1. Convección interior

> q_conv_int = h_conv_int × (T_superficie_int - T_indoor)

El coeficiente convectivo depende del **tipo de superficie** (techo, piso, muro) por la dirección de la convección natural — el calor sube, por lo que techos, pisos y muros tienen diferentes coeficientes.

#### 2. Radiación de onda larga interior

Intercambio radiativo **solo entre superficies interiores** (no cielo, ground ni aire como en el exterior). La radiación de onda larga **no atraviesa vidrios** → las superficies interiores solo se "ven" entre sí.

- Depende de **factores de vista** entre superficies y **emitancia** de los materiales
- Emitancia típica: materiales de construcción ~0.9; aluminio pulido ~0.1; vidrio baja emitancia ~0.01
- Puede ser el **60-70% de la transferencia de calor** en un muro — es un mecanismo dominante
- Si se inhibe en la simulación, los resultados cambian mucho
- Es una **estrategia de climatización**: una pared fría enfría radiativamente a las personas y objetos cercanos (proporcional a ΔT⁴, selectivo y potente)

**Ejemplo cotidiano:** sentir el calor al pasar junto a un muro de piedra caliente → es radiación, no convección. El traga fuegos en un semáforo: la llama está arriba de 700 K (emite en el visible), el calor que sentimos a través del vidrio es radiación térmica.

#### 3. Radiación de onda corta interior

Luz solar que entra por ventanas + equipos/luces que emiten en el visible. EnergyPlus la **distribuye uniformemente** en todas las superficies del espacio (simplificación — no puede apuntar un proyector a un muro específico).

- Modelo por defecto: radiación solar difusa y de equipos se reparte a todas las superficies
- Modelo "full interior exterior": proyecta la directa al piso, distribuye la difusa
- Equipos (ej. proyector de 900W): el 90% es luz visible, 10% calor → se distribuye a todas las superficies

### Modelo de mezclado perfecto (well-mixed)

EnergyPlus toma **todo** el flujo de calor que entra a la zona térmica en cada paso temporal (conducción por cada muro, radiación por ventanas, personas, infiltración) y lo mezcla instantáneamente con TODO el aire:

> **Toda la zona tiene UNA sola temperatura (T_indoor)**

**Limitaciones:**
- No es realidad — hay gradientes de temperatura, plumas térmicas, estratificación vertical
- Peor aproximación cuanto más grande sea el espacio
- No distingue temperatura cerca de una ventana vs. centro del cuarto
- Para validar experimentalmente: medir en múltiples puntos y promediar

**Mediciones de confort térmico:** se debería medir temperatura a la altura de tobillos, cadera y pecho (sentado o parado — las alturas cambian). Estas pueden ser diferentes por plumas térmicas y ráfagas de aire acondicionado.

---

## Primera simulación en Open Studio — Paso a paso

### 1. Editor de geometría (FloorSpaceJS)

Open Studio incluye un editor de geometría 2D integrado llamado **FloorSpaceJS** (JavaScript):

- Pestaña **Geometry** → **Editor** para dibujar en planta
- Pestaña **Geometry** → **Preview** (3D View) para visualizar el modelo
- Dibujo en planta 2D → **extrusión automática** a 3D
- Conectado a **OpenStreetMap** para referencia geográfica (buscar dirección)
- Puede importar imágenes de planta y escalarlas
- **Grid configurable**: 1m, 0.5m, 0.1m — solo se puede dibujar en el grid
- Herramientas: rectángulo, polígono, borrador
- Muestra dimensiones y área mientras se dibuja
- Formato: JSON (FloorSpaceJS)

Después de dibujar, se hace **Merge with Current OSM** para transferir la geometría al modelo.

### 2. Stories (pisos)

- Cada **Story** tiene una altura floor-to-ceiling (default: 2.43m)
- Se puede cambiar (ej. 3m)
- Se pueden renombrar (ej. "PB" para planta baja)
- Todos los espacios creados dentro de un story heredan su altura

### 3. Espacios vs Zonas térmicas

| Concepto | Origen | Descripción |
|----------|--------|-------------|
| **Espacio** | Open Studio | Volumen geométrico, puede tener tipo de uso (ej. centro de cómputo). No existe en EnergyPlus |
| **Zona térmica** | EnergyPlus | Donde se resuelve la temperatura del aire. Es lo que importa para la simulación |

- En casos simples: un espacio → una zona térmica
- Los nombres **deben ser diferentes** (ej. espacio: "s:Norte", zona: "Norte")
- Zonas térmicas se crean en la pestaña Thermal Zones (botón verde +)
- Se asignan a los espacios en la pestaña Spaces

**Reglas de nombres:**
- Usar nombres descriptivos: Cocina, Baño, LivingRoom, Norte, Sur
- **NO** usar ThermalZone1, ThermalZone2 (una semana después no sabrás qué es)
- **NO** usar espacios en nombres de zonas (causa problemas en Python)
- Evitar acentos y eñes (pueden causar errores en algunas computadoras)

### 4. Tipos de superficie y condiciones de frontera

**Render by Surface Type:**

| Color | Tipo | Nota |
|-------|------|------|
| Amarillo | Muro (Wall) | Coeficiente convectivo de muro |
| Rojo | Techo (Roof) | Coeficiente convectivo de techo |
| Gris | Piso (Floor) | Coeficiente convectivo de piso |

Los tipos importan porque el **coeficiente convectivo** depende de la inclinación de la superficie (convección natural: el calor sube).

**Render by Boundary Condition:**

| Color | Condición | Significado |
|-------|-----------|-------------|
| **Azul** | Outdoor | Expuesta a sol y viento — balance exterior completo |
| **Verde** | Surface / Interzone | Conectada a zona adyacente — flujo de calor pasa entre zonas |
| **Rojo** | Adiabatic | Sin transferencia de calor (flujo = 0) |
| **Otro** | Ground | Temperatura del suelo asociada |

**Condición Surface/Interzone:**
- Cuando dos espacios se tocan (merge en el editor), la pared compartida cambia automáticamente a Surface
- Es **un solo muro** (no doble) — el flujo que sale de un lado entra al otro
- Esas superficies ya no tienen radiación incidente ni intercambio con el exterior
- La condición Surface muestra con qué superficie hace "match" (ej. cara 10 ↔ cara 12)

**Error común:** si una pared compartida entre zonas queda en Outdoor → EnergyPlus calculará sol y viento donde no existen.

### 5. Aplicaciones de condiciones adiabáticas

- **Salones adyacentes** con el mismo comportamiento térmico → paredes laterales adiabáticas (no modelar vecinos)
- **Edificio multi-piso:** pisos intermedios adiabáticos arriba y abajo → solo modelar el piso de interés
- Solo el **último piso** (techo expuesto al sol) y **planta baja** (contacto con suelo) necesitan condiciones especiales
- **Pasillos/corredores** que no son zonas térmicas → se convierten en shading surfaces (solo bloquean luz, no participan en transferencia de calor, no bloquean viento)
- Si hay **A/C** abajo → ya no vale adiabático (el piso inferior tiene temperatura diferente y afecta)

### 6. Definición de materiales

En la pestaña de materiales (icono de ladrillos), crear **Material** (con masa, no "No Mass"):

| Propiedad | Descripción | Ejemplo (CAD) |
|-----------|-------------|---------------|
| Roughness | Rugosidad superficial (afecta h_conv) | Medium Rough |
| Thickness | Espesor [m] | 0.15 |
| Conductivity | Conductividad térmica [W/m·K] | 2.4 |
| Density | Densidad [kg/m³] | 2500 |
| Specific Heat | Calor específico [J/kg·K] | 1400 |
| Thermal Absorptance | Emitancia (cuerpo gris: emisividad = absorptancia) | 0.9 |
| Solar Absorptance | Fracción de radiación solar absorbida | 0.3 (blanco) – 0.9 (negro) |
| Visible Absorptance | Para cálculos de iluminación | ≈ Solar Absorptance |

- Valores en **verde** = default. Al modificar, desaparece el verde.
- Calores específicos típicos: 800–1600 J/kg·K
- Pasar de blanco (0.3) a rojo/oscuro (0.7) **duplica** la energía solar absorbida — una de las estrategias de mayor impacto

### 7. Sistemas constructivos (Construction)

- Crear en la pestaña Construction
- **Capas ordenadas de exterior a interior** (ej. concreto → tabique → yeso)
- Arrastrar materiales desde la librería del modelo (My Model)
- Asignar a cada superficie en la columna "Construction" de la pestaña Spaces/Surfaces

**Superficies vs Subsuperficies:**
- **Superficies** = muros, pisos, techos (opacos)
- **Subsuperficies** = ventanas, puertas → contenidas dentro de una superficie
- Una ventana **no puede existir** sin un muro contenedor, ni ocupar el 100% del área

### 8. Archivo de clima (EPW)

- Descargar de **One Building** → North America → Mexico → estado → ciudad
- Descomprimir ZIP → el archivo EPW es el que importa
- **Set Weather File** en Open Studio → seleccionar EPW
- Muestra: latitud, longitud, time zone, elevación
- Las simulaciones usan **horario civil** (no solar)

### 9. Ejecución

- Botón **Run** en la parte inferior de Open Studio
- Barra de progreso hasta 100%
- Requisitos mínimos: geometría + zonas térmicas + materiales/construcciones + EPW + condiciones de frontera correctas

### Metodología de archivos

```
PrimeraSimulacion/
├── OSM/
│   ├── 001_cubo_volumetria.osm
│   ├── 002_2zonas.osm
│   └── 003_primeraSimulacion.osm
├── EPW/
│   └── MEX_MOR_Cuernavaca_2007-2023.epw
└── notebooks/
    └── ...
```

**Reglas:**
- **NO** acentos, eñes ni espacios en nombres de archivos/carpetas
- Usar CamelCase o guiones_bajos
- **Versionado numérico:** 001, 002, 003... con nombre descriptivo
- **Nunca borrar** versiones anteriores — puede ser necesario regresar ("practicar el desapego")
- **Guardar como (Save As)** cada vez que se avanza → no sobreescribir
- Compartir como **ZIP** del proyecto completo (no solo el OSM)
- Open Studio crea una carpeta asociada con archivos temporales — no guardar cosas ahí

**Narrativa computacional:** el proyecto debe contar su propia historia. Al final, las últimas versiones serán las variantes bioclimáticas (cambio de color, materiales, orientación, protecciones solares).

**Sobre el hash en OSM:** Open Studio asigna un hash único a cada objeto. Si dos archivos tienen los mismos hashes, se detecta que se copió el archivo. No se puede cambiar fácilmente.

### Tarea

- Cubo de **3×3×3 metros**, una zona térmica
- Muros de **tabique**, piso y techo de **concreto** (dos sistemas constructivos, un material cada uno)
- Un **EPW** de la ciudad del equipo
- Condiciones de frontera: piso **adiabático**, resto **outdoor**
- Seguir los pasos de la clase
- Entrega: siguiente clase

---

## Conceptos clave

- **[[Mezclado-Perfecto]]** — suposición de EnergyPlus: todo el aire de la zona tiene una temperatura uniforme
- **[[Emitancia]]** — fracción de radiación emitida respecto a un cuerpo negro; igual a la absorptancia térmica (cuerpo gris)

Conceptos previos referenciados: [[Balance-de-Calor]], [[Zona-Termica]], [[Condiciones-de-Frontera]], [[Factor-de-Vista]], [[Absorptancia-Solar]], [[Sistemas-Constructivos]], [[Envolvente-Arquitectonica]], [[TMY]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · FloorSpaceJS · OpenStreetMap · SketchUp · Blender · Revit

## Conexiones

- **Anterior:** [[002-ConceptosBasicosBalancesCalor]] — Balance exterior, TMY, documentación
- **Siguiente:** [[004-InterpretandoMensajesConstructionSets]] — Mensajes de error y construction sets

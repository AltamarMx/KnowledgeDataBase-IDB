---
title: Posición del Aislante en el Sistema Constructivo
type: concepto
tags: [concepto, aislamiento, sistemas-constructivos, masa-termica, normativa, NOM]
aliases: [posicion aislante, aislante interior, aislante exterior, panel rey, NOM-008]
clases: [009, 010]
updated: 2026-05-02
---

# Posición del Aislante en el Sistema Constructivo

## La asimetría que las normas ignoran

> "En modelo independiente del tiempo (R total), la posición del aislante no importa. **En modelo dependiente del tiempo, sí importa**. La NOM-008 no especifica dónde."

Para una pared con la **misma resistencia térmica total**, la **posición** del aislante (interior, exterior, intermedio) cambia el comportamiento térmico de la edificación cuando se considera el modelo dependiente del tiempo (que es lo que hace Energy Plus).

## Por qué importa — modelo dependiente del tiempo

Una pared está hecha de capas con propiedades térmicas distintas:

```
Exterior  ┃    Capa A    ┃    Capa B    ┃   Capa C   ┃ Interior
                                                       
          ↑                                            ↑
        T_amb                                        T_int
```

En **estado estable** (modelo independiente del tiempo):

$$
\frac{T_{amb} - T_{int}}{R_A + R_B + R_C}
$$

→ no importa el orden de las capas. La R total es la misma.

En **régimen transitorio** (modelo dependiente del tiempo, con [[Masa-Termica]]):

- Cada capa tiene su propio $\rho \cdot c_p$ → almacena/libera calor en distintos momentos.
- La onda térmica que viaja del exterior al interior pasa por cada capa **secuencialmente**.
- El **orden importa**: poner el aislante antes o después de la masa cambia cómo la masa interactúa con el clima.

## Tres configuraciones típicas

### A. Aislante exterior (interior masivo)

```
Exterior  ┃ Aislante ┃        Concreto        ┃ Interior
```

- El aislante **protege la masa térmica** del clima exterior.
- La masa interior se mantiene a una T cercana a la del aire interior.
- **Estabiliza la T interior** — la masa actúa como volante térmico atemperado.
- **Mejor en climas con AC**: la masa absorbe los picos del AC y los libera lentamente.
- **Mejor en climas extremos** (calor o frío): aísla la masa de la fluctuación.

### B. Aislante interior (exterior masivo)

```
Exterior  ┃        Concreto        ┃ Aislante ┃ Interior
```

- La masa térmica está expuesta al clima → se calienta de día, se enfría de noche.
- El aislante interior **desacopla** la masa del aire interior.
- La masa "no sirve" para el confort interior — su calor no llega adentro.
- **Peor en muchos casos** que la configuración A.

### C. Aislante intermedio

```
Exterior  ┃ Concreto ┃ Aislante ┃ Concreto ┃ Interior
```

- Masa térmica a ambos lados del aislante.
- **Compromiso**: la masa interior estabiliza el aire interior; la masa exterior amortigua el clima.
- Configuración común en construcción tradicional con sandwich panels.

## Clima cálido sin AC — el caso peligroso

> "Si yo pongo un aislante, es posible — no voy a generalizar — que el flujo de calor que se genera aquí adentro no pueda salir y se va a almacenar. Entonces se puede volver caluroso."

Caso típico: **vivienda mexicana en clima cálido sin AC**.

Sin aislante:

- De día, el sol calienta la pared exterior y entra calor al interior.
- De **noche**, cuando T_amb baja, la pared exterior se enfría y el calor interior **sale** hacia afuera por conducción.

Con aislante:

- De día, el aislante reduce la entrada de calor (bien).
- De noche, el aislante **también reduce la salida** del calor interior (mal — atrapa el calor que se generó adentro o que entró por ventanas).

Resultado: la edificación puede estar **más caliente** en la noche con aislamiento que sin él. Empeora el confort.

> Por eso "**el aislamiento térmico no siempre es bueno**" — depende del clima, las cargas internas, la presencia de AC y la configuración geométrica.

## Crítica a NOM-008 / NOM-020

### Pre-reforma (versión original)

- Exigía aislamiento térmico **independientemente de si la edificación tiene AC**.
- En climas cálidos sin AC, esto puede empeorar el confort (atrapar calor interior).
- Acompañado de un argumento de "estarás más confortable" — falso en general.

### Post-reforma

- Reformulada para "climas cálidos con AC, para ahorrar energía de enfriamiento". Mejor pero aún incompleta.
- **No especifica la posición del aislante**. La mayoría de las edificaciones lo ponen al **exterior** porque es más fácil constructivamente (aunque el clima lo degrada).

> "Afortunadamente casi todo el mundo lo pone en el exterior porque es más fácil. Pero hemos visto casos donde lo ponen al interior por mantenimiento — y la NOM no dice nada al respecto."

### Conflicto de interés y green washing

> "La industria — los que hacen Panel Rey, los que hacen aislantes — lo ven como una oportunidad y empezaron a impulsar normativas para que la gente las cumpla."

La NOM se construyó con presión de la industria del aislante. Los promotores no necesariamente entendían:

- Diferencia entre modelo dependiente vs independiente del tiempo.
- Que el aislante puede empeorar el confort en ciertos climas.
- La importancia de la posición.

Resultado: una normativa con buena intención **contaminada por intereses económicos**. Aplicable a muchas certificaciones (LEED y otras) según el profesor.

## Herramienta primaria de evaluación — EnerHabitat

[[../tools/EnerHabitat]] es la herramienta específicamente diseñada para evaluar la **posición del aislante**:

- Simula la PDE 1D dependiente del tiempo → **sí captura la asimetría** que la NOM ignora.
- Permite comparar 3 sistemas (sin aislante / aislante exterior / aislante interior) con la misma R total **en segundos**.
- Reporta [[Factor-de-Decremento|FD sol-aire]] y tiempo de retraso → métricas que distinguen los tres casos.

Procedimiento sugerido en [[../procedures/Usar-EnerHabitat-Web]] o [[../procedures/Usar-EnerHabitat-Python]] para estudios paramétricos.

## Tarea propuesta — ejercicio de posición

> Hacer 3 simulaciones del mismo cubo, mismo clima, misma R total:
>
> 1. **Sin aislante** — solo concreto.
> 2. **Aislante exterior** — concreto al interior.
> 3. **Aislante interior** — concreto al exterior.
>
> Comparar:
> - T máxima al interior.
> - T mínima al interior.
> - Amplitud diaria.
> - Grados-hora de disconfort cálido y frío.
>
> Verificar empíricamente que la posición sí importa.

Tarea opcional propuesta en clase 009.

## Implicaciones para diseño

### Climas cálidos sin AC

- Cuestionar el aislamiento. En muchos casos vale más:
  - Pintura clara (reduce α — entrada de radiación).
  - Sombreamiento (aleros, vegetación).
  - Ventilación nocturna (disipa calor acumulado).
- Si se aísla, **al exterior** y considerar las cargas internas con cuidado.

### Climas cálidos con AC

- Aislamiento **al exterior** + masa térmica interior — configuración óptima.
- Reduce la carga del AC y aprovecha la inercia.

### Climas fríos

- Aislamiento al exterior (preserva masa interior caliente).
- Combinado con ganancia solar pasiva (ventanas al sur en hem. norte).

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — introducción del concepto y crítica a NOM
- [[../classes/010-EnerHabitatParte1]] — herramienta primaria para evaluar la posición del aislante

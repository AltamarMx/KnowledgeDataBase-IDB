# Leer el Archivo ERR de EnergyPlus

Procedimiento para diagnosticar errores y warnings en una simulación.

## Dónde encontrar el archivo

1. En Open Studio → botón **Show Simulation** (abre la carpeta de resultados)
2. Buscar el archivo con extensión `.err`
3. Abrirlo con un editor de texto (Notepad, VS Code, etc.)

## Tipos de mensajes

| Tipo | Significado | Acción |
|------|-------------|--------|
| **Warning** | Algo no definido o inusual — EnergyPlus asume un valor y continúa | Evaluar si afecta los resultados |
| **Severe** | Error fatal — la simulación se detiene | Corregir y volver a correr |

## Warnings comunes (ignorables en el curso)

- **Lifecycle Assessment outputs** — Open Studio espera métricas de consumo energético que no se definieron
- **Design days** no incluidos en el EPW — no afecta si no se dimensiona HVAC
- **Output variables** no definidas — se resolverán cuando se agreguen measures de salida

Estos warnings aparecen en prácticamente todas las simulaciones del curso. Lo importante es contar los warnings y verificar que no hay ninguno nuevo o inesperado.

## Errores severos comunes

### Falta archivo de clima (EPW)
- **Mensaje:** `No weather file found`
- **Causa:** no se asignó el EPW, o se movió el .osm y perdió la ruta
- **Solución:** Set Weather File → seleccionar el .epw

### Material faltante en sistema constructivo
- **Mensaje:** `Missing material in property "Outside Layer"` + nombre de la Construction
- **Causa:** una Construction no tiene todas sus capas definidas
- **Solución:** abrir la Construction y arrastrar el material desde My Model

### Condición de frontera incorrecta
- **Causa:** una pared compartida entre zonas quedó en Outdoor en vez de Surface
- **Solución:** verificar en Render by Boundary que los colores sean correctos

## Flujo recomendado

1. Correr la simulación
2. Si falla → Show Simulation → abrir `.err`
3. Buscar los **Severe errors** — son los que impiden correr
4. Leer el mensaje y el nombre del objeto afectado
5. Ir al objeto en Open Studio y corregir
6. Volver a correr → verificar que el error desapareció
7. Revisar los warnings restantes — decidir si son aceptables

## Aparece en

- [[004-InterpretandoMensajesConstructionSets]] — Explicación completa con ejemplos en vivo

Resumen
=======

**Agente Especialista en la Simulación de Gimnasio (TP4).** Entiende, mantiene y extiende el notebook `src/TP4_Simulacion_Gimnasio.ipynb`.

* Conoce la estructura completa del notebook: 12 celdas organizadas secuencialmente.
* Domina el modelo teórico subyacente: sistema **M/G/c/K con Cola de Prioridades** resuelto con metodología **Evento a Evento (EaE)**.
* Es capaz de modificar parámetros, agregar escenarios, corregir errores, generar nuevas visualizaciones y extender la lógica de simulación.

---

Contexto del Modelo
====================

### Dominio
Simulación del **horario pico de un gimnasio** (lunes a viernes, 18:00–21:00 hs — 180 min).
Los socios llegan, eligen una de tres áreas, usan una máquina y se van.
Existe una **credencial Premium** que otorga prioridad de acceso sobre socios regulares.

### Áreas del Gimnasio

| Área | Qué incluye | % de socios |
|---|---|---|
| **Cardio** | Cintas, bicicletas fijas, elípticas | 55% |
| **Musculación** | Máquinas de fuerza, pesas libres | 35% |
| **Funcional** | Área de circuito / crossfit | 10% |

### Sistema de Prioridades

- Si hay máquina libre → entra cualquier socio (Premium o Regular) sin esperar.
- Si NO hay máquina libre y llega un **Premium** → espera en la cola Premium (máx. 3 por área).
- Si NO hay máquina libre y llega un **Regular** → espera en la cola Regular (máx. 5 por área).
- Al liberarse una máquina → **SIEMPRE entra primero** el primer Premium en espera.
- Solo si NO hay Premiums esperando → entra el primer Regular.
- Si la cola respectiva está llena → el socio es **rechazado** (no puede entrar).

### Los 4 Escenarios Comparados

| Escenario | Cardio | Musculación | Funcional | Premium | Mejora |
|---|---|---|---|---|---|
| **No Eficiente** | 5 | 6 | 2 | 0% | Ninguna |
| **Real** | 9 | 10 | 4 | 0% | Ninguna |
| **Real + Premium** | 9 | 10 | 4 | 25% | Solo prioridad |
| **Eficiente + Premium** | 9 | 10 | 4 | 25% | App de reservas |

---

Estructura del Notebook — Celdas y Funcionalidad
=================================================

El notebook se organiza en **12 celdas** ejecutables secuencialmente. Cada celda cumple un rol específico en el pipeline de simulación:

### Celda 1 — Importaciones
- Librerías: `numpy`, `pandas`, `heapq`, `matplotlib`, `scipy.stats`.
- Suprime warnings con `warnings.filterwarnings('ignore')`.

### Celda 2 — Parámetros del Sistema (Variables del Análisis Previo)
- **Semilla**: `SEMILLA_BASE = 42` para reproducibilidad.
- **Áreas**: `AREAS = ['Cardio', 'Musculacion', 'Funcional']`.
- **F.D.P. del Intervalo entre Arribos (IA)**: Exponencial, λ=0.40 socios/min, media=2.5 min.
- **F.D.P. del Tiempo de Uso (TA)**: Normal truncada por área, con μ y σ propios, truncada en [8, 90] min.
  - Cardio: μ=32, σ=8
  - Musculación: μ=55, σ=12
  - Funcional: μ=45, σ=10
- **Probabilidad por área**: `[0.55, 0.35, 0.10]`.
- **Simulación**: 180 min, 30 réplicas.
- **Parámetros Premium**:
  - `PROPORCION_PREMIUM = 0.25` (25% de socios).
  - `MAX_COLA_PREMIUM = 3`, `MAX_COLA_REGULAR = 5`.
  - Tiempos Premium ~15% menores: Cardio μ=27, Musculación μ=45, Funcional μ=38.

### Celda 3 — Definición de Escenarios (Variables de Control)
- Diccionario `ESCENARIOS` con 4 configuraciones.
- Cada escenario define: `equipos_por_area`, `max_socios_en_cola_regular`, `max_socios_en_cola_premium`, `proporcion_premium`, `reduccion_tiempo_uso_minutos`.

### Celda 4b — Visualización de las F.D.P.
- Gráficos de las distribuciones teóricas (Exponencial para IA, Normales truncadas para TA).
- Verificación visual antes de generar el dataset.

### Celda 4 — Generación del Dataset
- Simula 5 semanas × 5 días = 25 jornadas de hora pico.
- Genera ~1800 registros, toma los primeros 1000.
- Cada registro: `id_socio`, `fecha`, `dia_semana`, `semana`, `hora_llegada`, `intervalo_entre_llegadas_min`, `area`, `fdp_ia`, `fdp_ta`, `tiempo_uso_maquina_min`.
- Usa `np.random.seed(SEMILLA_BASE)` para reproducibilidad.

### Celda 5 — Depuración y Validación del Dataset
Verifica 6 aspectos de calidad:
1. Sin valores nulos.
2. Sin valores negativos o cero en columnas numéricas.
3. Rangos realistas (TTE media ~2.5, uso entre 8 y 90 min).
4. Distribución por área dentro del 5% de tolerancia.
5. Estadísticas de uso por área consistentes con el modelo.
6. Horas de llegada dentro del horario pico.

### Celda 6 — Análisis Exploratorio Visual
- 3 gráficos: distribución de tiempo de uso por área, intervalo entre llegadas, socios por área y día.
- Guarda `analisis_exploratorio.png`.

### Celda 7 — Prueba de Bondad de Ajuste (Kolmogorov-Smirnov)
- Verifica que IA sigue Exponencial y TA sigue Normal por área.
- α=0.05. Si p-valor > 0.05 → acepta H0 (distribución ajusta).

### Celda 8 — Motor de Simulación (EaE con Cola de Prioridades)
**Esta es la celda más importante.** Define dos funciones:
- `simular_una_jornada(...)`: Simula UNA jornada de 180 min usando:
  - **TEF** (Tabla de Eventos Futuros): min-heap con tuplas `(tiempo, tipo_evento, id, area, tipo_socio)`.
  - **Vector de estado**: `maquinas_ocupadas`, `cola_premium`, `cola_regular` por área.
  - **Eventos**: `'llegada'` y `'salida'`.
  - **Lógica de prioridad**: al liberar máquina → primero Premium, luego Regular.
  - **Métricas de resultado**: llegadas, atendidos, rechazados, tasa de rechazo, utilización, espera promedio (todo separado Premium/Regular y por área).
- `simular_escenario(config, replicas, semilla)`: Ejecuta N réplicas de un escenario con semillas incrementales `(semilla_base + rep * 37)`.

### Celda 9 — Ejecución de los 4 Escenarios
- Itera sobre `ESCENARIOS`, ejecuta `simular_escenario()` para cada uno.
- Imprime media ± IC 95% de métricas clave.
- Almacena resultados en `resultados_por_escenario` (dict de DataFrames).

### Celda 10 — Análisis de Sensibilidad
- Varía máquinas de Musculación de 3 a 15.
- Mantiene fijo: Cardio=9, Funcional=4, sin Premium.
- 20 réplicas por punto.
- Genera `df_analisis_sensibilidad`.

### Celda 11 — Gráficos de Resultados
- 6 subplots en una figura 2×3:
  1. Tasa de rechazo Regular (%) por área.
  2. Tasa de rechazo Premium (%) por área.
  3. Tasa de rechazo total (%) ± IC 95%.
  4. Espera promedio Premium vs Regular en Musculación.
  5. Utilización (%) por área (Real vs Real+Premium vs Eficiente+Premium).
  6. Tabla resumen comparativa.
- Guarda `graficos_simulacion.png`.

### Celda 12 — Conclusiones
- Imprime resumen numérico por escenario.
- Calcula impacto de la credencial Premium (comparación Real vs Real+Premium).
- Conclusiones generales sobre el sistema.

---

Variables Clave del Código
===========================

### Constantes Globales
```
SEMILLA_BASE                     = 42
AREAS                            = ['Cardio', 'Musculacion', 'Funcional']
PROBABILIDAD_POR_AREA            = [0.55, 0.35, 0.10]
TASA_LLEGADA_SOCIOS_POR_MINUTO   = 0.40
DURACION_SIMULACION_MINUTOS      = 180
CANTIDAD_REPLICAS                = 30
PROPORCION_PREMIUM               = 0.25
MAX_COLA_PREMIUM                 = 3
MAX_COLA_REGULAR                 = 5
```

### Diccionarios de F.D.P.
- `FDP_INTERVALO_ENTRE_ARRIBOS`: Exponencial, λ=0.40.
- `FDP_TIEMPO_USO_POR_AREA`: Normal truncada por área.
- `TIEMPO_USO_REGULAR`: Igual que FDP_TIEMPO_USO_POR_AREA.
- `TIEMPO_USO_PREMIUM`: ~15% menor que Regular.

### Diccionario de Escenarios
- `ESCENARIOS`: 4 escenarios con claves descriptivas.

### Variables de Estado (dentro de `simular_una_jornada`)
- `maquinas_ocupadas[area]`: NS (Number in System).
- `cola_premium[area]`: socios Premium en espera.
- `cola_regular[area]`: socios Regular en espera.

### Variables de Resultado (métricas retornadas)
Por área y por tipo de socio: `Llegadas_*`, `Atendidos_*`, `Rechazados_*`, `Tasa_Rechazo_pct_*`, `Utilizacion_pct_*`, `Espera_Promedio_min_*`, `Cola_Max_*`.
Totales: `Total_Llegadas`, `Total_Atendidos`, `Total_Rechazados_Premium/Regular`, `Tasa_Rechazo_pct_*_Total`.

---

Instrucciones de Uso
=====================

### Cuándo Usar Este Agente
- Modificar parámetros del modelo (tasas, distribuciones, capacidades).
- Agregar nuevos escenarios de simulación.
- Corregir errores en la lógica de simulación EaE.
- Extender el análisis (nuevas métricas, nuevos gráficos).
- Interpretar resultados de la simulación.
- Responder preguntas conceptuales sobre el modelo M/G/c/K.

### Cómo Operar
1. **Siempre** ejecutar las celdas en orden secuencial (1 → 12).
2. Si se modifican parámetros en Celda 2 o 3, re-ejecutar desde esa celda.
3. Si se modifica la lógica del motor (Celda 8), re-ejecutar celdas 8 → 12.
4. Los gráficos se guardan automáticamente como archivos PNG.
5. La semilla garantiza reproducibilidad: misma semilla = mismos resultados.

### Mejores Prácticas
- Mantener la estructura de 12 celdas.
- Todo parámetro debe definirse en Celda 2 o 3, nunca hardcodeado en el motor.
- Nuevos escenarios se agregan al diccionario `ESCENARIOS` en Celda 3.
- Nuevas métricas se agregan tanto al retorno de `simular_una_jornada()` como a la visualización.
- Los comentarios en español deben mantenerse; el código sigue convenciones existentes.

---

Restricciones
==============

### Reglas Requeridas (DEBE HACER)
1. Respetar la estructura secuencial de las 12 celdas.
2. Mantener la reproducibilidad mediante la semilla base.
3. Separar siempre las métricas por tipo de socio (Premium/Regular) y por área.
4. Validar el dataset antes de simular (Celda 5).
5. Incluir intervalos de confianza (IC 95%) en todos los resultados reportados.

### Prohibiciones (NO DEBE HACER)
1. No modificar la lógica de prioridad (Premium antes que Regular al liberar máquina) sin instrucción explícita.
2. No eliminar la prueba de bondad de ajuste KS.
3. No hardcodear valores que deben venir de las constantes globales.
4. No cambiar la semilla base sin documentar el motivo.

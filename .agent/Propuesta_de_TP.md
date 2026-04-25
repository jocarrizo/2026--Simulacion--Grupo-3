# Propuesta de Trabajo Práctico: Optimización de Infraestructura para Servicios de IA

## 1. Contexto del Problema

Una empresa proveedora de servicios de Inteligencia Artificial (IA) gestiona un cluster de GPUs para procesar dos flujos de trabajo principales: Consultas de Usuarios (Inferencia) y Entrenamiento de Modelos de Gran Escala. Debido al crecimiento de la demanda, se requiere simular el comportamientso del sistema para evaluar su capacidad actual y proponer mejoras en la eficiencia operativa.

## 2. Descripción de Recursos y Demanda

El cluster se encuentra segmentado en dos niveles de procesamiento basados en el rastro de carga real (workload trace) analizado:

### Nivel Estándar (Inferencia)

 Un pool de $N$ unidades de procesamiento destinadas a consultas de baja latencia. Estas representan el 89% de los arribos totales. Los trabajos en este nivel pueden aguardar en una cola única en caso de saturación del pool.

### Nivel Premium (Entrenamiento)

 Una unidad de procesamiento de alta performance reservada exclusivamente para tareas de entrenamiento de modelos complejos. Estas representan el 11% de los arribos.

## 3. Lógica de Operación y Derivación

Dada la criticidad y el alto requerimiento de memoria de los trabajos de Entrenamiento, el sistema opera bajo las siguientes reglas:

### Procesamiento de Inferencia

 Si una unidad estándar está libre, se asigna el trabajo. Caso contrario, ingresa a una cola de espera con gestión FIFO.

### Procesamiento de Entrenamiento

 Debido a que estos procesos no pueden quedar suspendidos en memoria por largos periodos, si la unidad Premium está ocupada al momento del arribo, el trabajo es derivado automáticamente a una infraestructura externa (Cloud). Esta derivación se considera un costo de oportunidad y una pérdida de eficiencia del cluster propio.

### Distribuciones

 Los intervalos entre arribos y los tiempos de procesamiento para cada nivel se modelarán a partir de funciones de densidad de probabilidad ajustadas a los datos históricos del dataset provisto.

## 4. Objetivos de la Simulación

El equipo deberá desarrollar un modelo de simulación de eventos discretos para determinar:

### Tasa de Derivación

 Porcentaje de trabajos de entrenamiento que no pudieron ser absorbidos por la infraestructura local.

### Latencia Promedio

 Tiempo de permanencia en cola para las consultas de los usuarios.

### Utilización de Recursos

 Porcentaje de tiempo ocioso y de ocupación de ambos niveles de procesamiento.

### Análisis de Sensibilidad

 Evaluar si el incremento en el número de unidades estándar o la implementación de una política de asignación flexible mejora los indicadores globales.

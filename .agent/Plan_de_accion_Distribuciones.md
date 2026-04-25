# Plan de Acción: Análisis y Ajuste de Distribuciones de Datos

Este documento sintetiza el flujo de trabajo utilizado para cargar, procesar, analizar y ajustar datos a distribuciones de probabilidad. Este plan servirá como plantilla para crear un nuevo cuaderno de Jupyter (`.ipynb`) aplicable a cualquier otro dataset, asegurando la reproducibilidad tanto en entornos de la nube como locales.

## Librerías Necesarias
Las siguientes bibliotecas son fundamentales para ejecutar el código. Asegúrate de importarlas al inicio de tu nuevo cuaderno:
*   `numpy`: Operaciones matemáticas y manejo de arrays.
*   `pandas`: Carga, manipulación y análisis exploratorio preliminar de los datos.
*   `matplotlib.pyplot`: Creación de visualizaciones gráficas (ej. histogramas).
*   `scipy.stats`: Herramientas estadísticas, generación de variables aleatorias y funciones de densidad.
*   `fitter`: Ajuste automático de los datos a diversas distribuciones de probabilidad de Scipy para encontrar la más adecuada. *(Nota: suele requerir instalación previa ejecutando `!pip install fitter` en una celda).*

---

## Pasos a Seguir

### 1. Carga de Datos
El primer paso consiste en cargar el dataset en un `DataFrame` de Pandas. El código variará según el entorno de trabajo elegido.

*   **Desde Google Drive (ej. Google Colab):**
    ```python
    from google.colab import drive
    drive.mount('/content/drive')
    
    # Ruta al archivo dentro de Google Drive
    ruta_drive = '/content/drive/MyDrive/ruta/al/archivo.xlsx'
    df = pd.read_excel(ruta_drive, sheet_name='Nombre_Hoja')
    ```

*   **Desde un Directorio Local (ej. Jupyter Notebook o VSCode):**
    ```python
    import pandas as pd
    
    # Ruta relativa o absoluta en tu sistema de archivos local
    ruta_local = 'datos/mi_dataset.xlsx' 
    # Usar pd.read_csv() en lugar de read_excel si el archivo es CSV
    df = pd.read_excel(ruta_local, sheet_name='Nombre_Hoja')
    ```

### 2. Análisis Exploratorio de Datos (EDA)
Una vez cargados los datos, se debe realizar una inspección básica para conocer su estructura, limpieza y calidad.
*   **Dimensionalidad:** `df.shape` (Permite ver el número de filas y columnas).
*   **Tipos de datos:** `df.dtypes` (Verificar que las columnas a analizar tengan formato numérico como `int` o `float`). Si hay formatos incorrectos (ej. texto), se deben convertir usando `astype()`.
*   **Datos Nulos:** `df.isnull().sum()` (Identificar cuántos valores faltantes existen por columna para tratarlos debidamente).
*   **Inspección visual:** `df.head()` o `df.tail()` (Permite visualizar una muestra de las primeras o últimas filas).

### 3. Extracción y Limpieza de la Variable Objetivo
Se debe aislar la variable continua que se desea analizar y ajustar a una distribución.
```python
# Extracción de la columna como un array
datos = df['Nombre_Columna'].values
```
> **Punto Clave:** Antes de avanzar al ajuste probabilístico, es vital lidiar con valores nulos o "NaN" (ya sea descartándolos o imputándolos) para evitar errores matemáticos.

### 4. Visualización Inicial (Histograma)
Se genera un histograma con la variable de interés para obtener una idea preliminar de su forma y distribución empírica.
```python
import matplotlib.pyplot as plt

plt.hist(datos, bins=30, density=True, alpha=0.6, color='b')
plt.title('Histograma de la Variable de Interés')
plt.show()
```

### 5. Ajuste Probabilístico (Uso de Fitter)
Este paso emplea la librería `Fitter` para evaluar múltiples distribuciones estadísticas y encontrar aquella que se ajusta de mejor manera a la forma empírica de los datos.

*   **Inicializar y Ejecutar el Ajuste:**
    ```python
    from fitter import Fitter
    # Se puede inicializar sin indicar distribuciones (evaluará por defecto más de 80)
    # o restringir a algunas específicas usando: distributions=['norm', 'expon', ...]
    f = Fitter(datos, bins=30)
    f.fit()
    ```
*   **Obtener el Resumen de las Mejores Distribuciones:**
    ```python
    # Despliega un gráfico con las mejores distribuciones y una tabla con métricas
    f.summary(10)
    ```
*   **Extraer los Parámetros Óptimos:**
    ```python
    # Seleccionamos la mejor distribución según el criterio del menor error cuadrado
    mejores_parametros = f.get_best(method='sumsquare_error')
    print(mejores_parametros)
    ```

### 6. Verificación y Simulación
Con los parámetros devueltos en el paso previo (ej. `rho`, `loc`, `scale`), usaremos `scipy.stats` para generar un array de datos sintéticos simulados y contrastarlos visualmente.

*   **Ejemplo de Extracción y Generación:**
    ```python
    import scipy.stats as stats
    
    # NOTA: Cambiar 'rel_breitwigner' por el nombre de la distribución ganadora
    # que haya arrojado la función get_best().
    parametros = mejores_parametros['rel_breitwigner']
    rho = parametros['rho']
    loc = parametros['loc']
    scale = parametros['scale']
    
    # Generamos 4000 datos aleatorios con los parámetros identificados
    datos_simulados = stats.rel_breitwigner.rvs(rho, loc, scale, size=4000)
    ```
*   **Comprobación Visual:** El último paso recomendado es graficar un nuevo histograma a partir de los `datos_simulados` y observar si la estructura general guarda fidelidad con el histograma original.

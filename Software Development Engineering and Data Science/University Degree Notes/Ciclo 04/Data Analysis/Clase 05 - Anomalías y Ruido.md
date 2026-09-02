---
Fecha de creación: 2026-08-11T18:32:00
Materia:
  - Data Analysis
Fecha de clase: 2026-08-11
---
# Anomalías y Ruido

En ciencia de datos y estadística, el **ruido** son variaciones aleatorias o errores sin importancia dentro de la información, mientras que una **anomalía** (o valor atípico) es un dato raro y real que se aparta por completo del patrón normal y merece atención especial.
## Naturaleza y Origen

Las anomalías representan los casos atípicos de los datos numéricos, los cuales no se corresponden con las distribuciones estadísticas previamente calculadas, ya que presentan desviaciones significativas de los comportamientos de agrupaciones mayoritarias.

Los atípicos son aquellos valores que no resultan esperados por alguna razón. Se puede inferir que, en algunos casos, la atipicidad de los datos puede no ser un error técnico, ya que puede ser un error de comportamiento lógico del modelo. Las causas fundamentales pueden ser un error del ser humano en la codificación de los datos, mediciones imprecisas de los aparatos, un error en la transmisión de los datos a través de canales imprecisos, o un error en la elección de la distribución estadística.

---

## Detección estadística y visual

La identificación de extremos se puede realizar de la siguiente manera: de forma computacional mediante el cálculo de márgenes estadísticos y la construcción de gráficas.

- **Márgenes estadísticos:** se utiliza la mediana y el RIC (rango intercuartílico) ya que son medidas posicionales resistentes a la dispersión de los datos. Se definen como atípicos leves a los valores fuera del conjunto de valores entre [Q1-1.5*RIC y Q3+1.5*RIC], y atípicos graves a los valores que exceden el límite del conjunto de valores entre [Q1-3*RIC y Q3+3*RIC]. Otra herramienta es la de Z-score, que mide cuantas desviaciones típicas se aleja un valor de la media en una distribución normal.

| Método                | Descripción                                                     | Uso                                  |
| --------------------- | --------------------------------------------------------------- | ------------------------------------ |
| Z-Score               | Mide cuántas desviaciones estándar se aleja un dato de la media | Datos con distribución normal        |
| Rango intercuartílico | Usa Q1 y Q3 para detectar valores extremos                      | Datos con distribuciones no normales |
| Boxplot               | Representación gráfica de los valores extremos                  | Visualización exploratoria           |
_**Tabla 1.** Métodos comunes para detección de anomalías en un conjunto de datos._

- **Diagramas de caja (Boxplots):** Son una representación compacta de los datos y aísla los datos desviados a los límites esperados; sin embargo, tienen desventajas cuando los datos son demasiados para los recursos memoria disponibles.

![[Drawing 2026-08-11 18.46.51.excalidraw]]

---

## Outliers

Un **outlier** (o valor atípico) es una observación o dato que se aleja de forma extrema del resto de las observaciones en un conjunto de datos.

En el análisis de datos, un valor atípico rompe el patrón general del grupo. Puede distorsionar cálculos como la media aritmética o alterar los resultados de un modelo de aprendizaje automático.

### Corrección de Outliers

Una vez que se encuentran los extremos, éstos deben ser tratados para no degradar la analítica.

- **Eliminación de registros:** Implica la eliminación de la información de la instancia comprometida, siendo adecuada cuando la falla compromete la validez de toda la información de la instancia.
- **Corrección estadística (Truncado):** Se imputan valores razonables. La técnica de winsorización es resaltada por su facilidad y precisión en la corrección de extremos, donde los valores extremos son ajustados a los límites de los percentiles predefinidos (por ejemplo, percentil 5 y percentil 95). Una variante de winsorización es la pseudowinsorización que reemplaza directamente los atípicos graves por los límites exactos calculados con el RIC.

#### Winsorización

La **winsorización** es una técnica estadística que reemplaza los valores extremos o atípicos (_outliers_) de un conjunto de datos por otros valores menos extremos (como los percentiles superior e inferior), en lugar de eliminar los datos por completo, reduciendo así su impacto en el análisis.

#### Pseudowinsorización

La **pseudo winsorización** es una variación iterativa de la técnica estadística de winsorización. En lugar de fijar un porcentaje de corte estricto basado en percentiles (como el 5% o 95%), calcula la media y la desviación estándar, recorta los valores que superan un umbral (por ejemplo, a 2 desviaciones estándar), recalcula y repite el proceso hasta que ningún valor excede el límite.

#### Alternativas

- Se pueden retener los atípicos y confiar en algoritmos analíticos robustos o resistentes a ellos.
- Modelos no paramétricos (ej. árboles de decisión) son más robustos frente a atípicos que los paramétricos (ej. regresión).
- Se puede medir la efectividad con métricas robustas, como el error medio absoluto frente al error cuadrático medio.
- La métrica elegida no solo mide calidad: también dirige el propio comportamiento del algoritmo.

---

## Tres enfoques para tratar datos ruidosos

- **Aprendices robustos.** Técnicas poco influenciadas por el ruido. Ej: C4.5, que usa poda para evitar el sobreajuste a datos ruidosos.
- **Métodos de pulido.** Corrigen las instancias ruidosas antes de entrenar. Solo viables en datasets pequeños (consume mucho tiempo).
- **Filtros de ruido.** Identifican instancias ruidosas y las eliminan del conjunto de entrenamiento antes del aprendizaje.

---

## Identificación y tipología de ruido

El ruido se diferencia de la anomalía en que afecta significativamente las relaciones de clasificación supervisada, lo que reduce la capacidad para caracterizar los ejemplos.

- **Ruido de clase (etiqueta):** Ocurre cuando los ejemplos están mal etiquetados por subjetividad o error de entrada. Se divide en ejemplos contradictorios (donde los datos son duplicados, pero tienen etiquetas inconsistentes) y ejemplos erróneos.
- **Ruido de atributos:** Se le denomina a la corrupción intrínseca de los valores y engloba datos erróneos, incompletos o incógnitos.

### Filtrado y algoritmos robustos

Para combatir el ruido sin eliminar los atributos valiosos, se aplican los esquemas de votación por consenso (todos los clasificadores fallan la instancia) o mayoría (más de la mitad falla la instancia). Entre los filtros se incluyen:

- **Ensamble filter (EF):** Utiliza varios algoritmos (C4.5, 1-NN) en un subconjunto de datos.
- **Cross-Validated committees filter (CVCF):** Utiliza árboles de decisión C4.5 bajo la validación cruzada.
- **Iterative partitioning filter (IPF):** Elimina el ruido presente en grandes conjuntos de datos a través de varios pasos. La otra opción es el aprendizaje robusto. C4.5 tiene estrategias de "poda" que buscan minimizar el sobreajuste de la modelización debido a la presencia de ruidos en los datos y proveer modelos robustos sin eliminar ninguno de ellos.

---

## Diagnóstico en Python

### Inspección Visual

Se realiza usando bibliotecas gráficas como Matplotlib y Seaborn combinadas con técnicas estadísticas o de aprendizaje automático para resaltar puntos alejados del comportamiento normal.

```python
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns

# Crear datos de ejemplo con ruido y una anomalía clara
datos = np.random.normal(loc=0.0, scale=1.0, size=100)
datos = np.append(datos, [8.5, -7.0])  # Inyectar anomalías

# Crear diagrama de caja visual
sns.boxplot(data=datos)
plt.title("Detección visual de anomalías con Boxplot")
plt.show()
```

### Desviación Estándar (Diagnóstico Gaussiano)

Para realizar un diagnóstico o ajuste gaussiano de datos en Python, se utilizan librerías como **NumPy**, **SciPy** y **Matplotlib**. Puedes calcular la media y la desviación estándar para una prueba de normalidad o usar `scipy.optimize.curve_fit` para ajustar una curva de campana de Gauss a un conjunto de puntos.

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit

# Datos de ejemplo
x = np.linspace(-5, 5, 50)
y = 3.0 * np.exp(- (x - 1.0)**2 / (2 * 1.5**2)) + np.random.normal(0, 0.1, size=50)

# Definir la función gaussiana
def func_gauss(x, a, x0, sigma):
    return a * np.exp(-(x - x0)**2 / (2 * sigma**2))

# Estimar parámetros iniciales [amplitude, mean, stddev]
p0 = [1.0, 0.0, 1.0] 
popt, pcov = curve_fit(func_gauss, x, y, p0=p0)

# Resultados del diagnóstico/ajuste
print("Amplitud (A), Media (x0), Desviación (sigma):", popt)
```

### Rango Intercuartílico (IQR)

El rango intercuartílico (IQR) es la diferencia entre el tercer cuartil (Q3) y el primer cuartil (Q1) de un conjunto de datos, midiendo la dispersión central. Se calcula con la librería `numpy` mediante `np.percentile(datos, 75) - np.percentile(datos, 25)` y se visualiza de forma gráfica usando un diagrama de caja con `seaborn.boxplot` o `matplotlib.pyplot.boxplot`.

```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Datos de ejemplo
datos = [12, 15, 14, 18, 22, 25, 27, 30, 32, 45, 10, 19]

# Cálculo del IQR
q1 = np.percentile(datos, 25)
q3 = np.percentile(datos, 75)
iqr = q3 - q1

print(f"Q1: {q1}, Q3: {q3}, IQR: {iqr}")

# Visualización con Boxplot
plt.figure(figsize=(6, 4))
sns.boxplot(y=datos, color="skyblue")
plt.title("Diagrama de Caja (Boxplot)")
plt.ylabel("Valores")
plt.show()
```

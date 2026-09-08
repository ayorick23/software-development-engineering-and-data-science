---
tags: [estadistica, eda, cheat-sheet]
---

# 01 — Estadística Descriptiva para EDA

> Este cheat-sheet es la versión aplicada, orientada a Data Science/ML, de la estadística — no repite las clases universitarias. Para el fundamento teórico completo, ver [[University Degree Notes/Ciclo 03/Probabilidad y Estadística/Clase 01 - Introducción al Método Estadístico|Probabilidad y Estadística (Ciclo 03)]] y [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R|Estadística Inferencial (Ciclo 04)]]. Para el panorama conceptual dentro del flujo de ML, ver [[Machine Learning/05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML#Estadística|Machine Learning/05]].

## Por qué esta carpeta existe además de las clases de la universidad

Las clases de Estadística Inferencial cubren la teoría matemática con rigor (fórmulas, demostraciones, R). Esta carpeta cubre la **aplicación práctica en un contexto de Data Science/ML real**: qué estadístico calcular durante un EDA, qué distribución asumir para qué tipo de modelo, cómo diseñar un A/B test de principio a fin, qué supuestos hay detrás de un modelo de scikit-learn — el paso de la teoría al pipeline de trabajo diario.

## El resumen de cinco números (y por qué no basta la media)

```python
import pandas as pd

df = pd.DataFrame({"ingreso": [1200, 1350, 1100, 50000, 1280, 1400, 1150]})
df["ingreso"].describe()
```

`describe()` da de una vez: `count`, `mean`, `std`, `min`, `25%`, `50%` (mediana), `75%`, `max`. En un dataset real, comparar `mean` contra `50%` (mediana) es el primer diagnóstico de asimetría: si son muy distintos (como en el ejemplo, con un outlier de 50000), la media está siendo arrastrada por valores extremos y la mediana es una medida de centro más confiable.

## Medidas de forma: asimetría y curtosis

```python
df["ingreso"].skew()       # asimetría: >0 cola a la derecha, <0 cola a la izquierda, ~0 simétrica
df["ingreso"].kurtosis()   # curtosis: >0 colas más pesadas que la Normal (más outliers de lo esperado)
```

Estas dos métricas son el diagnóstico rápido de "¿puedo asumir Normalidad?" antes de aplicar un test paramétrico o un modelo que la asuma — ver [[08 - Supuestos Estadísticos Detrás de los Modelos de ML]].

## Detección de outliers: rango intercuartílico (IQR)

El método más usado en EDA, más robusto que "todo lo que esté a más de 2 desviaciones estándar" cuando los datos ya tienen outliers extremos (que inflan la propia desviación estándar):

```python
q1 = df["ingreso"].quantile(0.25)
q3 = df["ingreso"].quantile(0.75)
iqr = q3 - q1

limite_inferior = q1 - 1.5 * iqr
limite_superior = q3 + 1.5 * iqr

outliers = df[(df["ingreso"] < limite_inferior) | (df["ingreso"] > limite_superior)]
```

## Correlación entre variables numéricas

```python
df_num = pd.DataFrame({"x": [1, 2, 3, 4, 5], "y": [2.1, 3.9, 6.2, 7.8, 10.1]})

df_num.corr()                    # matriz de correlación de Pearson (lineal) entre todas las columnas
df_num.corr(method="spearman")   # correlación de rangos, no asume relación lineal ni Normalidad
```

Regla práctica: Pearson mide asociación **lineal**; dos variables pueden tener una relación fuerte pero no lineal (en forma de U, por ejemplo) y aun así dar una correlación de Pearson cercana a cero — siempre visualizar con un scatter plot antes de confiar solo en el número.

## Visualización rápida de distribución

```python
import matplotlib.pyplot as plt

df["ingreso"].hist(bins=20)   # forma de la distribución
df.boxplot(column="ingreso")  # centro, dispersión y outliers de un vistazo
```

Ver [[Python/Seaborn/04 - Gráficos de Distribución|Seaborn]] para versiones más completas (histograma + densidad superpuesta, violin plots).

## Ver también

- [[02 - Distribuciones y su Uso en ML]]
- [[Python/SciPy/04 - Estadística I - Distribuciones (scipy.stats)]]
- [[Python/Pandas/03 - Exploración e Inspección de Datos|Pandas — exploración inicial]]

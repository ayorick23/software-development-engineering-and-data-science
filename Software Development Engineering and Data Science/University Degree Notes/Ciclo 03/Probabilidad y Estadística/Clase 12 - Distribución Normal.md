---
Fecha de creación: 2026-04-22T18:52:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-04-17
---
[[Clase 10 - Laboratorio 2|← Clase anterior]] | [[Clase 13 - Tipos de Distribución|Clase siguiente →]]

# Distribución Normal

La **distribución normal**, también conocida como distribución gaussiana, es un modelo probabilístico continuo ampliamente utilizado para describir fenómenos naturales y sociales. Su forma característica es una campana simétrica en la que los valores tienden a concentrarse alrededor de un punto central. Esta distribución queda completamente definida por dos parámetros:

- **Media:** indica el centro de la distribución
- **Desviación estándar:** mide la dispersión de los datos respecto a esa media.

Se utiliza con frecuencia en contextos como la medición de estaturas, pesos, calificaciones o errores experimentales, ya que muchos procesos reales tienden a aproximarse a este comportamiento.

## Función de Densidad

La forma matemática de la distribución normal se describe mediante su función de densidad de probabilidad:

$$
f(x) = \frac{1}{\sigma\sqrt{2\pi}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$

Esta función no proporciona probabilidades directas en un punto específico, sino que describe la forma de la curva. Las probabilidades se obtienen como áreas bajo la curva.

## Propiedades

La distribución normal posee características que la hacen especialmente útil en estadística. Es simétrica respecto a su media, lo que implica que la media, la mediana y la moda coinciden en el mismo valor. Además, el área total bajo la curva es igual a 1, lo que representa el 100% de probabilidad.

Una de sus propiedades más importantes es la llamada **regla empírica**, que describe cómo se distribuyen los datos alrededor de la media:

- Aproximadamente el 68% de los datos se encuentran dentro de una desviación estándar de la media.
- Cerca del 95% se ubican dentro de dos desviaciones estándar.
- Alrededor del 99.7% se encuentran dentro de tres desviaciones estándar.

## Estandarización (Z-score)

La estandarización es un proceso que permite transformar cualquier valor de una distribución normal a una escala común llamada distribución normal estándar. Esto facilita el cálculo de probabilidades y la comparación entre distintos conjuntos de datos.

$$
Z = \frac{X-\mu}{\sigma}
$$

El valor $Z$ indica cuántas desviaciones estándar se encuentra un dato respecto a la media. Un valor positivo indica que está por encima de la media, mientras que uno negativo indica que está por debajo.

### Distribución normal estándar

La distribución normal estándar es un caso particular de la distribución normal en el que la media es cero y la desviación estándar es uno. Esta transformación permite utilizar tablas estadísticas o funciones computacionales para calcular probabilidades de manera más sencilla.

## Cálculo de probabilidades

El cálculo de probabilidades en una distribución normal se basa en el área bajo la curva. Estas probabilidades pueden representar valores acumulados menores que un punto, mayores que un punto o comprendidos entre dos valores. Para poder calcularlas correctamente, primero es necesario transformar los valores originales a su equivalente en Z.

### Ejemplo en Excel

Supongamos una variable con media 70 y desviación estándar 10, y se desea calcular la probabilidad de que un valor sea menor o igual a 85. En Excel, esto se puede resolver utilizando funciones estadísticas integradas que calculan directamente el área acumulada bajo la curva.

```excel
=NORM.DIST(85;70;10;VERDADERO)
```

Para obtener el valor estandarizado correspondiente:

```excel
=STANDARIZAR(85;70;10)
```

### Ejemplo en Python

En Python, el cálculo de probabilidades en una distribución normal se puede realizar utilizando la librería `scipy.stats`, la cual incluye funciones específicas para este propósito.

```python
from scipy.stats import norm

mu = 70
sigma = 10

probabilidad = norm.cdf(85, mu, sigma)
print(probabilidad)
```

También es posible calcular probabilidades entre dos valores restando probabilidades acumuladas:

```python
probabilidad = norm.cdf(85, mu, sigma) - norm.cdf(60, mu, sigma)
print(probabilidad)
```

### Visualización

La distribución normal puede visualizarse fácilmente mediante gráficos, lo cual ayuda a comprender su comportamiento y la relación entre los valores y sus probabilidades.

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

mu = 70
sigma = 10

x = np.linspace(40, 100, 100)
y = norm.pdf(x, mu, sigma)

plt.plot(x, y)
plt.title("Distribución Normal")
plt.show()
```

![[Drawing 2026-04-22 19.19.50.excalidraw]]
## Aplicaciones

La distribución normal es fundamental en múltiples áreas debido a su capacidad para modelar fenómenos reales y facilitar el análisis estadístico. Se utiliza especialmente en:

- Control de calidad de procesos
- Análisis de errores experimentales
- Finanzas y gestión de riesgo
- Machine Learning
- Pruebas de hipótesis e inferencia estadística

## Supuestos

Para que el uso de la distribución normal sea adecuado, los datos deben cumplir ciertas condiciones. En general, se espera que los datos sean continuos y que su distribución sea aproximadamente simétrica, sin una presencia significativa de valores atípicos extremos que puedan distorsionar el análisis.

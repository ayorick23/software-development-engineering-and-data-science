---
Fecha de creación: 2025-09-03T18:03:00
Materia:
  - Matemática II
Fecha de clase: 2025-09-03
---

# La Integral Definida

## Sumas de Reimann

## La Integral Definida y el Cálculo de Valores Exactos

Entender que la integral $\int_{a}^b f(x)dx$ representa la **acumulación de una tasa de cambio**, lo que permite calcular el **cambio total** en un intervalo, interpretado geométricamente como el área exacta bajo una curva.

**Contexto Histórico:** El concepto de aproximar áreas mediante la suma de infinitas partes se remonta a Arquímedes (~250 a.C.), quien utilizó el método de agotamiento para calcular el área de un círculo, un precursor del cálculo integral formalizado por Newton y Leibniz.

> **Aplicación en Ciencia de Datos:** Para una función de densidad de probabilidad `p(x)`, que describe la probabilidad de una variable continua (como la altura de una persona), la integral definida $\int_{a}^b p(x)dx$ calcula la probabilidad de que la variable caiga dentro del intervalo `[a, b]`. Este es un concepto central en la estadística.

## Teorema Fundamental del Cálculo

Analizar la relación inversa entre la diferenciación y la integración. Esto incluye la identificación de **integrales con límite superior variable** y el método para calcular integrales definidas a través de la antiderivada.

**Contexto Histórico:** Isaac Newton y Gottfried Leibniz, de forma independiente, establecieron la conexión formal entre la derivada y la integral. La notación de Leibniz, $\int$ y $\frac{dy}{dx}$, se convirtió en el estándar por su claridad y poder descriptivo.

> **Aplicación en Ciencia de Datos:** En el entrenamiento de modelos de machine learning, el algoritmo de descenso de gradiente ajusta los parámetros para minimizar una función de error. La derivada (gradiente) nos dice la dirección del cambio. El Teorema Fundamental del Cálculo nos asegura que al 'sumar' (integrar) estos pequeños ajustes, podemos determinar el cambio total en el error del modelo.

## Integración de Funciones Trascendentes

Identificar y aplicar las técnicas de integración para funciones exponenciales, logarítmicas y trigonométricas ($e^x$, $ln(x)$, $sin(x)$, $cos(x)$, etc.), las cuales son fundamentales para modelar fenómenos físicos y sistemas dinámicos.

> **Aplicación en Ciencia de Datos:** La función exponencial $e^x$ es la base de la función sigmoide en regresión logística y de distribuciones de probabilidad clave. Las funciones $sin(x)$ y $cos(x)$ son indispensables en el análisis de series de tiempo (mediante las Series de Fourier) para detectar ciclicidad en los datos, como ventas estacionales o patrones de tráfico web.

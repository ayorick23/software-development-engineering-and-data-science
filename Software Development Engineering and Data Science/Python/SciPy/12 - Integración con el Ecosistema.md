---
tags: [scipy, ecosistema, cheat-sheet]
---

# 12 — Integración con el Ecosistema

## NumPy — la base

Toda la API de SciPy recibe y devuelve `ndarray`. No hay conversión que hacer: los resultados de SciPy se pasan directamente a funciones de NumPy y viceversa. Ver [[Python/NumPy/16 - Integración con el Ecosistema]].

## Pandas

SciPy no entiende de `DataFrame` — trabaja con arrays. El patrón habitual es extraer una o más columnas como array antes de llamar a SciPy:

```python
import pandas as pd
from scipy import stats

df = pd.DataFrame({"grupo_a": [23, 25, 21, 28], "grupo_b": [30, 32, 29, 31]})

t_stat, p_valor = stats.ttest_ind(df["grupo_a"].values, df["grupo_b"].values)
# .values (o .to_numpy()) no siempre es estrictamente necesario -- pandas.Series
# suele funcionar directamente porque expone la interfaz de array de NumPy --
# pero es más explícito y evita sorpresas con el índice.
```

## scikit-learn

Scikit-learn usa `scipy.sparse` internamente para representar datos de texto vectorizado (`TfidfVectorizer`, `CountVectorizer` devuelven matrices CSR), y `scipy.optimize`/`scipy.linalg` como motor interno de varios algoritmos (regresión logística, SVM). Ver [[Python/Scikit-learn/01 - Introducción, Filosofía y la API Consistente]].

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizador = TfidfVectorizer()
matriz = vectorizador.fit_transform(["texto de ejemplo", "otro texto"])
type(matriz)   # <class 'scipy.sparse._csr.csr_matrix'>
```

## statsmodels

`statsmodels` se apoya en `scipy.stats` para las distribuciones subyacentes de sus pruebas, pero añade lo que SciPy no tiene: modelos con interpretación de coeficientes (regresión con resúmenes tipo R, series de tiempo con ARIMA, modelos lineales generalizados). Regla práctica: si solo se necesita el p-valor de una prueba, `scipy.stats` basta; si se necesita el modelo completo con coeficientes, errores estándar y diagnósticos, usar `statsmodels`.

## Matplotlib

Ninguna integración especial: los arrays que produce SciPy (una FFT, una curva ajustada, un histograma) se grafican igual que cualquier array de NumPy. Ver [[Python/Matplotlib/01 - Introducción y Arquitectura]].

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[13 - Buenas Prácticas, Errores Comunes y Comparativa]]

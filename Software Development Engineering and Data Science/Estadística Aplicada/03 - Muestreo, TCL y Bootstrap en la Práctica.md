---
tags: [estadistica, muestreo, bootstrap, cheat-sheet]
---

# 03 — Muestreo, TCL y Bootstrap en la Práctica

> Para la teoría del Teorema Central del Límite, ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 03 - Distribuciones de Probabilidad Continuas#Teorema Central del Límite (TCL)|Estadística Inferencial, Clase 03]].

## Por qué el TCL importa en la práctica, más allá de la teoría

El TCL garantiza que la media muestral se distribuye aproximadamente Normal cuando $n$ es grande — pero en la práctica de Data Science, el valor real está en lo que **habilita**: construir intervalos de confianza para *cualquier* métrica (no solo la media), incluso cuando no se conoce (ni se puede asumir) la distribución original de los datos. Ese es exactamente el rol del **bootstrap**.

## Bootstrap: la idea en una frase

Si no se puede repetir el estudio real miles de veces, se **remuestrea la muestra que ya se tiene**, con reemplazo, miles de veces — cada remuestreo simula "qué habría pasado con otra muestra", sin necesidad de asumir ninguna distribución teórica.

```python
import numpy as np

datos = np.array([23, 45, 12, 67, 34, 29, 88, 15, 52, 41])

def bootstrap_media(datos, n_iteraciones=10000):
    medias = []
    for _ in range(n_iteraciones):
        muestra = np.random.choice(datos, size=len(datos), replace=True)   # remuestreo CON reemplazo
        medias.append(muestra.mean())
    return np.array(medias)

medias_bootstrap = bootstrap_media(datos)
ic_95 = np.percentile(medias_bootstrap, [2.5, 97.5])   # intervalo de confianza del 95%, sin asumir Normalidad
print(f"Media: {datos.mean():.2f}, IC 95%: ({ic_95[0]:.2f}, {ic_95[1]:.2f})")
```

Con `scipy.stats.bootstrap` el mismo cálculo es una sola línea — ver [[Python/SciPy/05 - Estadística II - Pruebas de Hipótesis (scipy.stats)#Bootstrap: intervalos de confianza sin supuestos de distribución|SciPy]].

## Cuándo el bootstrap es preferible al intervalo clásico (t/Z)

| Situación | Preferir |
|---|---|
| Muestra grande, datos aproximadamente Normales, se quiere IC para la media | Intervalo clásico (t) — más rápido, misma respuesta |
| Métrica sin fórmula analítica de error estándar (mediana, un percentil, un ratio de dos métricas) | Bootstrap — no existe una fórmula cerrada equivalente a $s/\sqrt{n}$ para muchas de estas métricas |
| Datos claramente no Normales o con outliers, muestra pequeña | Bootstrap — más robusto que asumir Normalidad que no se sostiene |
| Se necesita el IC de una métrica compuesta de un modelo de ML (AUC, F1) | Bootstrap — remuestrear el conjunto de test y recalcular la métrica en cada remuestreo |

## Bootstrap aplicado a una métrica de modelo de ML

Un uso extremadamente común en la práctica: reportar no solo el AUC de un modelo sino su incertidumbre, remuestreando el conjunto de test:

```python
from sklearn.metrics import roc_auc_score

def bootstrap_auc(y_true, y_pred_proba, n_iteraciones=1000):
    n = len(y_true)
    aucs = []
    indices = np.arange(n)
    for _ in range(n_iteraciones):
        idx_muestra = np.random.choice(indices, size=n, replace=True)
        if len(np.unique(y_true[idx_muestra])) < 2:
            continue   # remuestreo sin ambas clases -- AUC no definido, se descarta
        aucs.append(roc_auc_score(y_true[idx_muestra], y_pred_proba[idx_muestra]))
    return np.percentile(aucs, [2.5, 97.5])
```

Esto responde a la pregunta "¿es este modelo genuinamente mejor que el anterior, o la diferencia de AUC está dentro del ruido de muestreo?" — algo que reportar un único número de AUC no permite evaluar.

## Muestreo estratificado: preservar proporciones al dividir datos

Al dividir en train/test, un muestreo puramente aleatorio puede desbalancear accidentalmente la proporción de clases (crítico en datasets desbalanceados, como detección de fraude):

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42   # stratify preserva la proporción de clases de y
)
```

## Ver también

- [[04 - Intervalos de Confianza Aplicados]]
- [[Python/SciPy/05 - Estadística II - Pruebas de Hipótesis (scipy.stats)]]
- [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 07 - Estimación e Intervalos de Confianza]]

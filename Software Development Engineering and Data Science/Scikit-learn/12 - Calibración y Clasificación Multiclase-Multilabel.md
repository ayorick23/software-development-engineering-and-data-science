---
tags: [scikit-learn, machine-learning, calibracion, multiclase, multilabel, cheat-sheet]
---

# 12 — Calibración y Clasificación Multiclase / Multilabel

> Continúa de [[11 - Datos Faltantes y Clases Desbalanceadas]].

## Calibración de probabilidades (`CalibratedClassifierCV`)

### El problema: no todos los modelos producen probabilidades confiables

`predict_proba()` existe en la mayoría de los clasificadores, pero el valor que devuelve no siempre es una probabilidad **bien calibrada** — un modelo puede decir "80% de probabilidad de churn" para un grupo de clientes donde en realidad solo el 60% termina yéndose. Esto importa cuando las probabilidades se usan directamente para decisiones de negocio (priorizar leads, fijar umbrales de riesgo), no solo para clasificar.

```python
from sklearn.calibration import CalibratedClassifierCV, CalibrationDisplay
from sklearn.svm import SVC

modelo_base = SVC(probability=False)   # SVC sin probability=True es más rápido de entrenar

modelo_calibrado = CalibratedClassifierCV(
    estimator=modelo_base,
    method="sigmoid",   # "sigmoid" (Platt scaling, mejor con pocos datos) o "isotonic" (más flexible, requiere más datos)
    cv=5,
)
modelo_calibrado.fit(X_train, y_train)

probabilidades = modelo_calibrado.predict_proba(X_test)   # ahora sí, mejor calibradas
```

### Visualizar la calibración — reliability diagram

```python
CalibrationDisplay.from_estimator(modelo_calibrado, X_test, y_test, n_bins=10)
```

Grafica la probabilidad predicha vs. la frecuencia observada real dentro de cada bin — un modelo perfectamente calibrado produce una línea diagonal (45°); desviaciones indican sobre-confianza o sub-confianza sistemática en ciertos rangos de probabilidad.

### Qué modelos suelen necesitar calibración

| Modelo | Calibración nativa |
|---|---|
| `LogisticRegression` | Generalmente bien calibrado por construcción |
| `GaussianNB` | Con frecuencia sobre-confiado (probabilidades muy cercanas a 0 o 1) — se beneficia de calibración |
| `SVC` | No calibrado por defecto — requiere `CalibratedClassifierCV` o `probability=True` (que internamente ya usa un método similar) |
| `RandomForestClassifier` / árboles | Con frecuencia mal calibrado, especialmente con pocos árboles — se beneficia de calibración |

## Clasificación Multiclase

### Estrategias — One-vs-Rest vs. One-vs-One

```python
from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier
from sklearn.svm import SVC

# One-vs-Rest (OvR): entrena N clasificadores binarios, uno por clase contra "el resto"
modelo_ovr = OneVsRestClassifier(SVC(probability=True))
modelo_ovr.fit(X_train, y_train)

# One-vs-One (OvO): entrena N*(N-1)/2 clasificadores binarios, uno por cada PAR de clases
modelo_ovo = OneVsOneClassifier(SVC())
modelo_ovo.fit(X_train, y_train)
```

La mayoría de los clasificadores de scikit-learn (`LogisticRegression`, `RandomForestClassifier`, árboles) **ya manejan multiclase de forma nativa**, sin necesitar estos envoltorios — `OneVsRestClassifier`/`OneVsOneClassifier` son necesarios principalmente para algoritmos que son intrínsecamente binarios (como `SVC` con ciertos kernels, o cuando se quiere forzar una estrategia específica sobre el comportamiento por defecto).

- **OvR**: más rápido de entrenar (N modelos vs. N*(N-1)/2), es la estrategia por defecto en la mayoría de casos.
- **OvO**: cada clasificador ve solo datos de dos clases, lo que puede ayudar con algoritmos sensibles a desbalance entre muchas clases simultáneas — pero escala mal cuando hay muchas clases.

### `OutputCodeClassifier` — codificación de salida robusta a errores

```python
from sklearn.multiclass import OutputCodeClassifier

modelo = OutputCodeClassifier(SVC(), code_size=2, random_state=42)
```

Estrategia menos común, basada en códigos de corrección de errores — útil en escenarios académicos/específicos, raramente necesaria en la práctica frente a OvR.

## Clasificación y Regresión Multilabel / Multi-output

### La diferencia: multiclase vs. multilabel

- **Multiclase**: cada muestra pertenece a **exactamente una** de N clases (ej. `"bajo"`/`"medio"`/`"alto"`).
- **Multilabel**: cada muestra puede pertenecer a **varias** etiquetas simultáneamente, de forma independiente (ej. un ticket de soporte puede ser `"urgente"` Y `"facturación"` a la vez).

### `MultiOutputClassifier` / `MultiOutputRegressor`

```python
from sklearn.multioutput import MultiOutputClassifier, MultiOutputRegressor

# y_train tiene MÚLTIPLES columnas objetivo, ej. predecir demanda de 3 productos distintos a la vez
modelo = MultiOutputRegressor(RandomForestRegressor(n_estimators=200), n_jobs=-1)
modelo.fit(X_train, y_train_multicolumna)

predicciones = modelo.predict(X_test)   # devuelve un array con una columna por target
```

Entrena **un modelo independiente por cada columna objetivo** — no captura relaciones entre los distintos outputs (por ejemplo, que la demanda del producto A y B suelen moverse juntas). Simple y efectivo cuando esa independencia es una suposición razonable.

### `ClassifierChain` — modela dependencias entre etiquetas

```python
from sklearn.multioutput import ClassifierChain

modelo = ClassifierChain(LogisticRegression(), order="random", random_state=42)
modelo.fit(X_train, y_train_multilabel)
```

A diferencia de `MultiOutputClassifier` (etiquetas independientes), `ClassifierChain` encadena los clasificadores: cada etiqueta se predice usando las features originales **más** las predicciones de las etiquetas anteriores en la cadena — captura correlaciones entre etiquetas a costa de que el orden de la cadena pueda afectar el resultado.

### Algunos modelos ya soportan multi-output nativamente

```python
# RandomForestRegressor y DecisionTreeRegressor soportan y de múltiples columnas SIN envoltorio:
from sklearn.ensemble import RandomForestRegressor

modelo = RandomForestRegressor(n_estimators=200)
modelo.fit(X_train, y_train_multicolumna)   # funciona directamente, sin MultiOutputRegressor
```

Vale revisar la documentación del estimador específico antes de envolver innecesariamente con `MultiOutputRegressor`/`Classifier` — varios modelos basados en árboles ya soportan múltiples columnas objetivo de forma nativa.

## Ver también

- [[05 - Métricas y Evaluación]] (`average` en métricas multiclase)
- [[06 - Modelos Lineales - Sintaxis y API]]
- [[08 - SVM, Vecinos Cercanos y Naive Bayes]]

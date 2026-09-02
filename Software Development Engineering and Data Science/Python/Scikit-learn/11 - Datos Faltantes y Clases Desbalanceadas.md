---
tags: [scikit-learn, machine-learning, imputacion, desbalance, smote, cheat-sheet]
---

# 11 — Datos Faltantes y Clases Desbalanceadas

> Continúa de [[10 - Selección de Features]]. Cubre `sklearn.impute` y la integración con la librería externa `imbalanced-learn`.

## Imputación de valores faltantes (`sklearn.impute`)

### `SimpleImputer` — el caso general

```python
from sklearn.impute import SimpleImputer

imputer_num = SimpleImputer(strategy="median")      # "mean", "median", "most_frequent", "constant"
imputer_cat = SimpleImputer(strategy="most_frequent")

X_train_imputado = imputer_num.fit_transform(X_train[columnas_numericas])

# Con valor constante explícito:
imputer_constante = SimpleImputer(strategy="constant", fill_value=-1)
```

`strategy="median"` es preferible a `"mean"` para features con outliers o distribuciones asimétricas — la mediana no se distorsiona por valores extremos. `strategy="most_frequent"` (la moda) es la opción estándar para columnas categóricas.

### `add_indicator` — no perder la señal de "esto estaba faltante"

```python
imputer = SimpleImputer(strategy="median", add_indicator=True)
X_imputado = imputer.fit_transform(X_train)
# agrega columnas booleanas adicionales: "esta feature estaba faltante en esta fila"
```

A veces el hecho de que un valor **falte** es en sí mismo informativo (por ejemplo, un cliente sin historial de compras porque es nuevo) — `add_indicator=True` preserva esa señal en vez de que se pierda al rellenar el valor.

### `KNNImputer` — imputar según muestras similares

```python
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=5, weights="distance")
X_imputado = imputer.fit_transform(X_train_scaled)   # requiere features ya escaladas
```

En vez de un valor fijo (media/mediana global), imputa usando el promedio de los `k` vecinos más similares (basado en las demás features) — más preciso que `SimpleImputer` cuando hay estructura real en los datos que predice el valor faltante, a costa de mayor tiempo de cómputo.

### `IterativeImputer` — imputación multivariada (estilo MICE)

```python
from sklearn.experimental import enable_iterative_imputer   # necesario, API aún experimental
from sklearn.impute import IterativeImputer

imputer = IterativeImputer(max_iter=10, random_state=42)
X_imputado = imputer.fit_transform(X_train)
```

Modela cada feature con valores faltantes como una función de las demás features (regresión), iterando varias rondas hasta converger — la técnica de imputación más sofisticada disponible en scikit-learn, inspirada en el método MICE (Multiple Imputation by Chained Equations) usado en estadística.

### Integración en Pipeline — igual que cualquier transformador

```python
from sklearn.pipeline import Pipeline

pipeline_numerico = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])
```

Como con cualquier paso de preprocesamiento, la imputación debe vivir **dentro** del Pipeline (ver [[03 - Pipelines y ColumnTransformer]]) — calcular la mediana/moda usando todo el dataset (incluyendo test) antes de dividir es una forma sutil de leakage.

## Clases desbalanceadas

### `class_weight` — la primera línea de defensa, sin librerías externas

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier

modelo = LogisticRegression(class_weight="balanced")
modelo = RandomForestClassifier(class_weight="balanced")   # también disponible en la mayoría de clasificadores

# Peso manual explícito por clase:
modelo = LogisticRegression(class_weight={0: 1, 1: 10})
```

Disponible nativamente en la mayoría de los clasificadores de scikit-learn (`LogisticRegression`, `SVC`, `RandomForestClassifier`, `DecisionTreeClassifier`) — penaliza más los errores sobre la clase minoritaria en la función de pérdida, sin necesidad de generar o eliminar muestras. Es la opción más simple y suele ser un buen primer intento antes de recurrir a resampling.

### `imbalanced-learn` — la librería complementaria estándar

```bash
pip install imbalanced-learn
```

Extiende la API de scikit-learn (mismo patrón `fit_resample`, compatible con `Pipeline` usando su propia clase `Pipeline` extendida) con técnicas de resampling.

### `SMOTE` — sobremuestreo sintético de la clase minoritaria

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42, k_neighbors=5)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)
```

En vez de simplemente duplicar muestras minoritarias (lo que puede llevar a overfitting sobre esas copias exactas), SMOTE genera muestras **sintéticas nuevas** interpolando entre muestras minoritarias existentes y sus vecinos más cercanos.

### `RandomUnderSampler` — submuestreo de la clase mayoritaria

```python
from imblearn.under_sampling import RandomUnderSampler

rus = RandomUnderSampler(random_state=42)
X_resampled, y_resampled = rus.fit_resample(X_train, y_train)
```

Elimina muestras de la clase mayoritaria hasta balancear las proporciones — más simple que SMOTE, pero descarta información potencialmente útil; preferible cuando el dataset es tan grande que perder muestras mayoritarias no es un problema práctico.

### `imblearn.pipeline.Pipeline` — CRÍTICO: no usar `sklearn.pipeline.Pipeline` con resampling

```python
from imblearn.pipeline import Pipeline as ImbPipeline   # NO from sklearn.pipeline

pipeline = ImbPipeline([
    ("scaler", StandardScaler()),
    ("smote", SMOTE(random_state=42)),
    ("modelo", LogisticRegression()),
])
pipeline.fit(X_train, y_train)   # SMOTE se aplica SOLO dentro de cada fold de entrenamiento, nunca en validación/test
```

> **El error más costoso en este tema**: aplicar SMOTE (o cualquier resampling) sobre el dataset **antes** de dividir en train/test, o antes de cross-validation. Esto genera muestras sintéticas basadas en toda la información disponible, incluyendo lo que debería ser "test" — y como las muestras sintéticas de SMOTE son interpolaciones cercanas a muestras reales, es fácil que una muestra sintética de train sea casi idéntica a una muestra real de test, causando fuga de información severa y métricas de validación falsamente optimistas. El `Pipeline` de `imbalanced-learn` (no el de scikit-learn estándar) es indispensable aquí porque garantiza que `fit_resample` se ejecute solo con los datos de entrenamiento de cada fold, nunca sobre validación/test — el `Pipeline` estándar de scikit-learn ni siquiera soporta pasos con `fit_resample`, por lo que este error suele evitarse simplemente porque el código no correría; el riesgo real aparece cuando alguien aplica SMOTE manualmente, fuera de cualquier Pipeline, antes del split.

### `SMOTEENN` / `SMOTETomek` — combinar sobre y submuestreo

```python
from imblearn.combine import SMOTEENN, SMOTETomek

smote_enn = SMOTEENN(random_state=42)
X_resampled, y_resampled = smote_enn.fit_resample(X_train, y_train)
```

Genera muestras sintéticas con SMOTE y luego limpia muestras ruidosas/ambiguas cerca de la frontera de decisión — suele producir clusters más limpios que SMOTE solo, a costa de mayor complejidad y tiempo de cómputo.

## Tabla de decisión rápida

| Situación | Enfoque |
|---|---|
| Desbalance moderado (ej. 80/20), primer intento | `class_weight="balanced"` |
| Desbalance severo (ej. 99/1), dataset pequeño-mediano | `SMOTE` dentro de `imblearn.pipeline.Pipeline` |
| Dataset muy grande, se puede perder algo de la clase mayoritaria | `RandomUnderSampler` |
| Necesitas máxima limpieza en la frontera de decisión | `SMOTEENN` / `SMOTETomek` |

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- [[06 - Modelos Lineales - Sintaxis y API]] (`class_weight`)
- [[16 - Buenas Prácticas, Errores Comunes y Rendimiento]]

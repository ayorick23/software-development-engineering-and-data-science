---
tags: [scikit-learn, machine-learning, preprocessing, cheat-sheet]
---

# 02 — Preprocessing y Escalado

> Continúa de [[01 - Introducción, Filosofía y la API Consistente]].

El módulo `sklearn.preprocessing` contiene transformadores para llevar los datos crudos a la forma que los algoritmos necesitan: escalas comparables, distribuciones más simétricas, variables categóricas convertidas a numéricas.

## Por qué escalar — qué algoritmos lo necesitan y cuáles no

| Sensibles a la escala (SIEMPRE escalar) | Insensibles a la escala (no hace falta) |
|---|---|
| KNN, SVM, regresión con regularización (Ridge/Lasso), redes neuronales, PCA, K-Means | Árboles de decisión, Random Forest, Gradient Boosting, XGBoost/LightGBM |

Los modelos basados en árboles dividen por umbrales en cada feature de forma independiente — no les afecta que una variable esté en rango `[0, 1]` y otra en `[0, 1000000]`. Los modelos basados en distancias o en gradiente descendente sí, porque las magnitudes distintas distorsionan la noción de "distancia" o desestabilizan la convergencia del optimizador.

## `StandardScaler` — media 0, desviación estándar 1

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # aprende media y std de TRAIN
X_test_scaled = scaler.transform(X_test)          # aplica los mismos parámetros a TEST

print(scaler.mean_, scaler.scale_)   # atributos aprendidos
```

Fórmula: `z = (x - media) / desviación_estándar`. Es el escalador por defecto para la mayoría de casos — asume (sin requerirlo estrictamente) que los datos son razonablemente simétricos, sin outliers extremos.

## `MinMaxScaler` — rango fijo, típicamente [0, 1]

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 1))
X_scaled = scaler.fit_transform(X_train)
```

Fórmula: `x_scaled = (x - min) / (max - min)`. Útil cuando se necesita un rango acotado explícito (por ejemplo, para redes neuronales con funciones de activación sensibles al rango de entrada). **Muy sensible a outliers**: un solo valor extremo comprime todo el resto de los datos hacia un rango minúsculo.

## `RobustScaler` — resistente a outliers

```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler()
X_scaled = scaler.fit_transform(X_train)
```

Usa la **mediana** y el **rango intercuartílico (IQR)** en vez de media/desviación estándar — no se distorsiona por outliers extremos. Preferible a `StandardScaler` cuando el dataset tiene valores atípicos legítimos que no se quieren eliminar pero tampoco deben dominar el escalado.

## `Normalizer` — escala por fila, no por columna

```python
from sklearn.preprocessing import Normalizer

normalizer = Normalizer(norm="l2")   # cada FILA (muestra) tiene norma euclidiana = 1
X_normalized = normalizer.fit_transform(X)
```

Conceptualmente distinto a los anteriores: en vez de normalizar cada columna/feature a través de todas las muestras, normaliza cada muestra individual a través de sus features. Común en texto (vectores TF-IDF) y en contextos donde importa la **dirección** del vector de features más que su magnitud absoluta.

## `PowerTransformer` — normalizar distribuciones asimétricas

```python
from sklearn.preprocessing import PowerTransformer

pt = PowerTransformer(method="yeo-johnson")   # soporta valores negativos
# method="box-cox" — requiere valores estrictamente positivos, a veces da mejores resultados si aplica
X_transformed = pt.fit_transform(X_train)
```

Aplica una transformación que acerca la distribución de cada feature a una normal — útil antes de modelos que asumen (implícita o explícitamente) normalidad, como regresión lineal con inferencia estadística, o simplemente para reducir el efecto de colas largas.

## `QuantileTransformer` — forzar una distribución objetivo

```python
from sklearn.preprocessing import QuantileTransformer

qt = QuantileTransformer(output_distribution="normal", n_quantiles=1000)
X_transformed = qt.fit_transform(X_train)
```

Mapea los datos a los cuantiles de una distribución objetivo (normal o uniforme) — más agresivo que `PowerTransformer`, colapsa outliers extremos hacia los bordes de la distribución objetivo. Es una transformación no lineal, por lo que puede distorsionar relaciones lineales entre features si eso importa para el modelo posterior.

## Codificación de variables categóricas

### `OneHotEncoder` — el estándar para categorías sin orden

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(
    handle_unknown="ignore",   # categorías nuevas en test/producción no rompen el pipeline
    sparse_output=False,        # devuelve array denso en vez de matriz dispersa
    drop="first",                # evita colinealidad perfecta (dummy variable trap) para modelos lineales
)
X_encoded = encoder.fit_transform(X_train[["region"]])

print(encoder.categories_)           # categorías aprendidas por columna
print(encoder.get_feature_names_out())  # nombres de las columnas resultantes
```

`handle_unknown="ignore"` es crítico en producción: sin esto, si aparece una categoría nunca vista durante el entrenamiento, `transform()` lanza una excepción en vez de simplemente codificarla como todo-ceros.

### `OrdinalEncoder` — para categorías con orden natural

```python
from sklearn.preprocessing import OrdinalEncoder

encoder = OrdinalEncoder(
    categories=[["bajo", "medio", "alto"]],   # orden EXPLÍCITO — si no se especifica, usa orden alfabético
    handle_unknown="use_encoded_value",
    unknown_value=-1,
)
X_encoded = encoder.fit_transform(X_train[["nivel_riesgo"]])
```

A diferencia de `OneHotEncoder`, produce **una sola columna** con enteros ordenados (0, 1, 2, ...) — apropiado solo cuando la variable tiene un orden real (`bajo < medio < alto`). Usarlo en una variable sin orden (como `región`) introduce una relación de magnitud falsa que el modelo puede aprender erróneamente.

### `LabelEncoder` — específico para la variable objetivo (`y`)

```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
y_encoded = le.fit_transform(y_train)   # convierte etiquetas de texto a enteros 0..n_clases-1

print(le.classes_)                       # las clases originales, en el orden usado
y_original = le.inverse_transform(y_pred)   # revertir para interpretar predicciones
```

> **Diferencia clave con `OrdinalEncoder`**: `LabelEncoder` está diseñado para codificar `y` (una sola columna, la variable objetivo), no para features (`X`). Usar `LabelEncoder` sobre columnas de `X` es un antipatrón común — no es compatible con `ColumnTransformer` de la misma forma que `OrdinalEncoder`/`OneHotEncoder`.

### `TargetEncoder` — para categorías de alta cardinalidad

```python
from sklearn.preprocessing import TargetEncoder

encoder = TargetEncoder(target_type="continuous")   # o "binary" / "multiclass"
X_encoded = encoder.fit_transform(X_train[["oficina_id"]], y_train)   # requiere y en fit()
```

Reemplaza cada categoría por una estimación (con suavizado interno vía cross-validation) del valor promedio de `y` para esa categoría — evita la explosión de columnas de `OneHotEncoder` cuando hay cientos de categorías (ej. un `oficina_id` con 200+ valores únicos). **Requiere y como argumento de `fit()`**, a diferencia de los demás encoders — y por eso es especialmente sensible a leakage si no se integra correctamente dentro de un `Pipeline`/cross-validation (scikit-learn lo maneja internamente con un esquema tipo out-of-fold, pero sigue siendo el encoder más delicado de la lista).

## Tabla de decisión rápida

| Situación | Transformador |
|---|---|
| Features numéricas, sin outliers extremos | `StandardScaler` |
| Necesitas rango acotado explícito (ej. redes neuronales) | `MinMaxScaler` |
| Outliers legítimos presentes | `RobustScaler` |
| Distribución muy asimétrica (colas largas) | `PowerTransformer` |
| Normalizar por muestra (texto, vectores de dirección) | `Normalizer` |
| Categórica sin orden, pocas categorías | `OneHotEncoder` |
| Categórica con orden natural | `OrdinalEncoder` |
| Variable objetivo `y` categórica | `LabelEncoder` |
| Categórica de alta cardinalidad | `TargetEncoder` |

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- [[10 - Selección de Features]]
- `Machine Learning/40-Feature-Engineering-Avanzado.md`

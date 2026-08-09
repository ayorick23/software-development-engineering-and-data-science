---
tags: [machine-learning, arboles-de-decision, overfitting]
---

# 35 — Árboles de Decisión en Profundidad: Gini, Entropía, Pruning, Overfitting

> Nota del mentor: XGBoost, Random Forest y Gradient Boosting son, en el fondo, cientos o miles de árboles de decisión combinados. Si no entiendes profundamente cómo un solo árbol decide dónde partir los datos, nunca vas a entender realmente por qué un ensamble de árboles se comporta como se comporta — vas a quedarte tuneando hiperparámetros por ensayo y error, sin criterio real.

## 1. Cómo decide un árbol dónde partir — el concepto central

Un árbol de decisión, en cada nodo, busca la pregunta ("¿`total_demand` > 45?") que **mejor separa** los datos según la variable objetivo. Para regresión, esto se mide típicamente con la reducción de varianza; para clasificación, con Gini o entropía.

## 2. Gini Impurity — usado en clasificación (CART, el algoritmo detrás de scikit-learn)

```
Gini = 1 - Σ(p_i)²
```

Donde `p_i` es la proporción de cada clase en el nodo. Un nodo perfectamente puro (todas las observaciones de la misma clase) tiene Gini = 0; un nodo con 50/50 de dos clases tiene Gini = 0.5 (el peor caso para dos clases).

```python
# Nodo con 80% clase A, 20% clase B
gini = 1 - (0.8**2 + 0.2**2)  # = 1 - (0.64 + 0.04) = 0.32

# Nodo con 50% clase A, 50% clase B (máxima impureza para 2 clases)
gini = 1 - (0.5**2 + 0.5**2)  # = 1 - 0.5 = 0.5
```

El árbol prueba múltiples posibles cortes (para cada feature, múltiples umbrales) y elige el que produce la **mayor reducción de Gini** entre el nodo padre y el promedio ponderado de los nodos hijos resultantes.

## 3. Entropía — la alternativa basada en teoría de la información

```
Entropía = -Σ p_i · log2(p_i)
```

Conceptualmente similar a Gini (ambas miden impureza), pero la entropía viene de la teoría de la información de Shannon — mide "cuánta sorpresa" hay en el nodo. En la práctica, Gini y entropía casi siempre eligen los mismos cortes; Gini es computacionalmente más rápido (no requiere logaritmos) y es el default de scikit-learn por esa razón, no porque sea "mejor" estadísticamente.

## 4. Para regresión (tu caso real en Claro RD) — reducción de varianza / MSE

Como tu problema es predecir `total_demand` y `avg_service_time` (valores continuos, no clases), el árbol no usa Gini/entropía — usa reducción de varianza (equivalente a MSE):

```python
from sklearn.tree import DecisionTreeRegressor

arbol = DecisionTreeRegressor(criterion="squared_error", max_depth=5)
```

En cada nodo, el árbol busca el corte que minimiza la varianza combinada de los dos grupos resultantes — el mismo espíritu de "maximizar la pureza/homogeneidad de los grupos", adaptado a valores continuos.

## 5. Overfitting en árboles — el peligro real y por qué es tan fácil caer en él

Un árbol sin restricciones puede crecer hasta que **cada hoja contiene una sola observación** — esto da error de entrenamiento igual a cero, y generaliza terriblemente mal, porque memorizó ruido específico de cada fila de entrenamiento en vez de patrones generales.

```python
# Sin restricciones — receta garantizada de overfitting severo
arbol_sobreajustado = DecisionTreeRegressor()  # max_depth=None por defecto
```

**Señales de overfitting en un árbol**: error de entrenamiento cercano a cero, error de validación mucho más alto, y un árbol visualmente enorme y profundo con hojas de una o dos observaciones cada una.

## 6. Pruning (poda) — las tres formas de controlarlo

### Pre-pruning (limitar el crecimiento desde el inicio)

```python
arbol = DecisionTreeRegressor(
    max_depth=6,                # profundidad máxima del árbol
    min_samples_split=20,        # mínimo de observaciones para considerar un corte
    min_samples_leaf=10,         # mínimo de observaciones en cada hoja resultante
    max_features="sqrt",         # limita cuántos features considera en cada corte
)
```

- **`max_depth`**: el control más directo e interpretable — limita cuántas preguntas consecutivas puede hacer el árbol.
- **`min_samples_leaf`**: evita hojas basadas en 1-2 observaciones (ruido casi puro) — para una tabla de forecasting con cientos de miles de filas, un valor de al menos 20-50 suele ser razonable como punto de partida.
- **`min_samples_split`**: similar, pero controla el corte antes de que ocurra, no el resultado final.

### Post-pruning (crecer completo, luego podar)

```python
arbol_completo = DecisionTreeRegressor(max_depth=None)
path = arbol_completo.cost_complexity_pruning_path(X_train, y_train)
# ccp_alphas: valores de complejidad de poda a evaluar con cross-validation
mejores_resultados = [
    DecisionTreeRegressor(ccp_alpha=alpha).fit(X_train, y_train)
    for alpha in path.ccp_alphas
]
```

`cost_complexity_pruning_path` (Minimal Cost-Complexity Pruning) deja crecer el árbol completamente y después elimina las ramas que menos aportan a la reducción de error relativo a su complejidad añadida — se elige el nivel de poda óptimo evaluando cada candidato con cross-validation, igual que se elegiría `alpha` en Ridge/Lasso.

## 7. Por qué esto importa aunque nunca uses un árbol individual en producción

- Random Forest controla overfitting mediante **agregación de muchos árboles independientes** (ver [[36-Ensambles-en-Profundidad]]), no mediante pruning agresivo de cada árbol individual — de hecho, los árboles de un Random Forest suelen crecer casi sin restricción (`max_depth` alto o `None`), porque el overfitting de cada árbol individual se promedia y cancela con el resto del bosque.
- Gradient Boosting y XGBoost, en cambio, sí necesitan árboles individuales poco profundos (`max_depth=3-6` típicamente) porque cada árbol corrige los errores del anterior de forma secuencial — un árbol demasiado profundo en boosting sobreajusta agresivamente y arrastra ese sobreajuste a todo el ensamble siguiente.

Esta diferencia de filosofía entre Random Forest y Gradient Boosting/XGBoost —árboles profundos + promediado vs. árboles poco profundos + corrección secuencial— es exactamente lo que profundizamos en la siguiente nota, y es la razón real detrás de por qué sus hiperparámetros por defecto son tan distintos.

## Ver también
- [[34-Modelos-Lineales-en-Profundidad]]
- [[36-Ensambles-en-Profundidad]]
- [[37-Validacion-Rigurosa-en-ML]]

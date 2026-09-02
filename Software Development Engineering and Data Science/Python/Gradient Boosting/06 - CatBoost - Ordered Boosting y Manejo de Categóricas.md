---
tags: [gradient-boosting, catboost, cheat-sheet]
---

# 06 — CatBoost: Ordered Boosting y Manejo de Categóricas

> Continúa de [[01 - Introducción y Panorama]]. Este archivo cubre lo que distingue conceptualmente a CatBoost — el siguiente ([[07 - CatBoost - API, Pool y Funcionalidades Distintivas]]) cubre la API completa.

## El problema que CatBoost resuelve mejor que las demás: target leakage en categóricas

Cuando se usa target encoding tradicional (reemplazar una categoría por el promedio de `y` para esa categoría, ver `Scikit-learn/02 - Preprocessing y Escalado.md`), existe un riesgo sutil de leakage: si se calcula el promedio de `y` usando la **misma fila** que se está codificando, esa fila "ve" su propio valor objetivo indirectamente a través del encoding — un sesgo optimista que infla artificialmente el desempeño en entrenamiento.

## Ordered Target Encoding — la solución de CatBoost

CatBoost resuelve esto con un esquema llamado **Ordered Target Statistics**: para cada muestra, calcula el encoding de su categoría usando **solo las muestras que aparecen antes que ella** en un ordenamiento aleatorio artificial (distinto en cada árbol del ensamble) — nunca usa el propio valor objetivo de la fila que está codificando, ni el de muestras "futuras" en ese ordenamiento.

```
Ordenamiento aleatorio de las filas: [fila_7, fila_2, fila_9, fila_1, ...]

Para codificar "region" en fila_9:
  → usa SOLO las estadísticas de fila_7 y fila_2 (las que aparecen ANTES en este ordenamiento)
  → nunca usa fila_9 misma, ni fila_1 (que aparece después)
```

Este es el mismo principio de "no dejar que el futuro informe el pasado" que en series de tiempo motiva `TimeSeriesSplit` (ver `Scikit-learn/04 - Model Selection - Validación y Búsqueda.md`), aplicado aquí a un ordenamiento artificial aleatorio en vez de temporal — el objetivo es idéntico: evitar que una muestra influya en su propio encoding.

## Ordered Boosting — el mismo principio aplicado al boosting completo

CatBoost extiende esta idea más allá del encoding de categóricas: por defecto (`boosting_type="Ordered"`), también usa órdenes aleatorios distintos para calcular los gradientes/residuos usados en cada árbol, reduciendo el sesgo acumulativo que puede surgir cuando el mismo conjunto de datos se usa repetidamente para ajustar y para evaluar el progreso del boosting (un problema conocido como *prediction shift* en la literatura de boosting).

```python
from catboost import CatBoostRegressor

modelo = CatBoostRegressor(
    boosting_type="Ordered",   # el default en datasets pequeños/medianos — más lento pero más robusto
    # boosting_type="Plain"     # más rápido, similar al boosting clásico de XGBoost/LightGBM — mejor en datasets grandes
)
```

## Símmetric Trees (Oblivious Trees) — la otra decisión de diseño distintiva

```
Árbol simétrico de CatBoost — el MISMO criterio de split se usa en todos los nodos de un nivel:

           [split: region == "RD"?]
          /                        \
   [split: precio > 100?]      [split: precio > 100?]   ← MISMA condición en ambos nodos del nivel
     /            \                /            \
  [hoja]        [hoja]          [hoja]         [hoja]
```

A diferencia de XGBoost (level-wise, cada nodo puede tener un split distinto) o LightGBM (leaf-wise, estructura asimétrica), CatBoost por defecto usa **árboles simétricos**: todos los nodos de un mismo nivel usan exactamente la misma condición de split. Esto simplifica dramáticamente la evaluación del modelo (se puede vectorizar de forma muy eficiente, beneficiando la velocidad de inferencia) y actúa como una forma implícita de regularización — a costa de menos flexibilidad estructural que un árbol asimétrico.

## Por qué esto se traduce en menos necesidad de tuning manual

La combinación de Ordered Boosting + árboles simétricos hace que CatBoost sea, en la práctica, **más robusto con sus hiperparámetros por defecto** que XGBoost/LightGBM — es común obtener un desempeño competitivo con configuración mínima, mientras que las otras dos suelen requerir tuning más cuidadoso (especialmente LightGBM, dado el riesgo de overfitting del crecimiento leaf-wise, ver [[04 - LightGBM - Arquitectura Leaf-wise y API Nativa]]) para alcanzar resultados comparables.

## Comparación conceptual de las tres estrategias de árbol

| Librería | Estrategia | Ventaja principal | Costo |
|---|---|---|---|
| XGBoost | Level-wise (por defecto) | Balance predecible, buen control vía `gamma`/`max_depth` | Puede gastar cómputo en splits poco útiles |
| LightGBM | Leaf-wise | Converge rápido con pocas hojas totales | Riesgo de overfitting en datasets pequeños |
| CatBoost | Symmetric/oblivious | Inferencia muy rápida, regularización implícita | Menos flexibilidad estructural por árbol |

## Manejo de categóricas — declaración explícita, sin encoding previo

```python
from catboost import CatBoostRegressor, Pool

columnas_categoricas = ["region", "tipo_producto"]

modelo = CatBoostRegressor(iterations=500, cat_features=columnas_categoricas)
modelo.fit(X_train, y_train)   # X_train puede tener las columnas categóricas como strings CRUDOS, sin encoding
```

No requiere `OneHotEncoder`, `OrdinalEncoder` ni `TargetEncoder` manual — CatBoost aplica su Ordered Target Encoding internamente sobre las columnas marcadas en `cat_features`, a partir de los valores de texto originales. Esta es la ventaja práctica más citada de la librería: reduce significativamente el trabajo de feature engineering en datasets de negocio con muchas columnas categóricas (región, categoría de producto, tipo de cliente).

## Ver también

- [[07 - CatBoost - API, Pool y Funcionalidades Distintivas]]
- [[04 - LightGBM - Arquitectura Leaf-wise y API Nativa]]
- `Scikit-learn/02 - Preprocessing y Escalado.md`
- `Scikit-learn/04 - Model Selection - Validación y Búsqueda.md`

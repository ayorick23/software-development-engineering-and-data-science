---
tags: [gradient-boosting, xgboost, lightgbm, catboost, shap, interpretabilidad, cheat-sheet]
---

# 10 — Interpretabilidad e Importancia de Features

> Continúa de [[08 - Comparativa Técnica y Tuning Cruzado]]. Complementa `Machine Learning/39-Interpretabilidad-de-Modelos.md`.

## Importancia nativa — sintaxis por librería

```python
# XGBoost (API sklearn-compatible)
importancias_xgb = modelo_xgb.feature_importances_   # usa "gain" por defecto

# LightGBM
importancias_lgb = modelo_lgb.feature_importances_    # usa "split" (conteo) por defecto — distinto default a XGBoost

# CatBoost
importancias_cat = modelo_cat.get_feature_importance(train_pool, type="FeatureImportance")
```

> **Cuidado al comparar importancias entre librerías**: cada una usa un tipo de importancia distinto como default (`gain` en XGBoost, `split`/conteo en LightGBM) — comparar directamente los valores crudos entre modelos de librerías distintas sin homogeneizar el tipo de métrica lleva a conclusiones erróneas. Para comparación justa, especificar explícitamente el mismo tipo de importancia (`gain`) en las tres, o preferir SHAP (ver siguiente sección), que sí es directamente comparable entre modelos de árboles.

## SHAP — la forma robusta y comparable entre las tres librerías

```bash
pip install shap
```

```python
import shap

# TreeExplainer funciona de forma NATIVA y eficiente con las tres librerías, sin adaptación manual:
explainer_xgb = shap.TreeExplainer(modelo_xgb)
explainer_lgb = shap.TreeExplainer(modelo_lgb)
explainer_cat = shap.TreeExplainer(modelo_cat)

shap_values = explainer_xgb.shap_values(X_test)
shap.summary_plot(shap_values, X_test)
```

`shap.TreeExplainer` implementa un algoritmo exacto y eficiente específicamente diseñado para modelos basados en árboles (`TreeSHAP`) — funciona igual de bien con XGBoost, LightGBM, CatBoost y los ensambles nativos de scikit-learn, lo que lo convierte en el método preferido para comparar la importancia de features **de forma consistente** entre modelos entrenados con librerías distintas.

## SHAP nativo dentro de CatBoost — sin dependencia externa

```python
shap_values_nativo = modelo_cat.get_feature_importance(train_pool, type="ShapValues")
```

CatBoost es la única de las tres con soporte de cálculo de SHAP **integrado directamente en la librería** (sin necesitar `pip install shap` por separado) — ver [[07 - CatBoost - API, Pool y Funcionalidades Distintivas]].

## Gráficas de dependencia parcial — efecto de una feature sobre la predicción

```python
shap.dependence_plot("dias_atras", shap_values, X_test)

# Vía scikit-learn, funciona igual con estos modelos por ser sklearn-compatible:
from sklearn.inspection import PartialDependenceDisplay
PartialDependenceDisplay.from_estimator(modelo_xgb, X_train, features=["dias_atras", "temperatura"])
```

Ver `Scikit-learn/15 - Integraciones con el Ecosistema.md` para `PartialDependenceDisplay` — funciona sobre XGBoost/LightGBM/CatBoost exactamente igual que sobre un `RandomForestRegressor`, gracias a la compatibilidad de API.

## Permutation Importance — alternativa agnóstica al modelo

```python
from sklearn.inspection import permutation_importance

resultado = permutation_importance(modelo_xgb, X_test, y_test, n_repeats=10, random_state=42, n_jobs=-1)
importancias = pd.Series(resultado.importances_mean, index=X_test.columns).sort_values(ascending=False)
```

A diferencia de `feature_importances_` nativo (que puede variar en metodología entre librerías) y SHAP (que asume acceso a la estructura interna del árbol), Permutation Importance mide directamente el impacto en el desempeño del modelo al aleatorizar cada feature — funciona igual sobre cualquier estimador de scikit-learn, útil como verificación cruzada independiente de las importancias nativas.

## Explicar una predicción individual — SHAP force plot / waterfall

```python
explainer = shap.TreeExplainer(modelo_xgb)
shap_values_individual = explainer(X_test.iloc[[0]])   # explicación de UNA sola muestra

shap.plots.waterfall(shap_values_individual[0])
```

Responde a la pregunta "¿por qué el modelo predijo esto para ESTA muestra específica?" — indispensable en contextos donde una predicción individual necesita justificación (por ejemplo, por qué se rechazó un crédito o por qué se clasificó una transacción como fraude), a diferencia de la importancia global que solo describe el comportamiento promedio del modelo.

## Interacciones entre features — SHAP interaction values

```python
shap_interaction_values = explainer.shap_interaction_values(X_test)
```

Descompone el efecto de cada par de features en la predicción — más costoso computacionalmente que los SHAP values estándar, pero revela interacciones (ej. que `learning_rate` y `region` juntas afectan la predicción de forma distinta a la suma de sus efectos individuales) que `plot_contour`-style de un solo modelo lineal no puede capturar.

## Ver también

- [[07 - CatBoost - API, Pool y Funcionalidades Distintivas]]
- `Scikit-learn/07 - Árboles y Ensambles - Sintaxis y API.md` (permutation importance)
- `Machine Learning/39-Interpretabilidad-de-Modelos.md`
- `MLflow/12 - MLflow para LLMs y GenAI.md` (SHAP también mencionado en `MLflow/11 - Evaluación de Modelos (mlflow.evaluate).md`)

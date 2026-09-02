---
tags: [machine-learning, modelos-lineales, regularizacion]
---

# 34 — Modelos Lineales en Profundidad: Regularización, Ridge, Lasso, ElasticNet

> Nota del mentor: probablemente ya usaste regresión lineal en la universidad y la consideras "el modelo básico que se salta rápido para llegar a XGBoost". Error común. Entender regularización a fondo es lo que te permite entender **por qué** los modelos de árboles y boosting también necesitan controles similares contra el overfitting — el concepto es el mismo, solo cambia la forma.

## 1. El problema que la regularización resuelve

Una regresión lineal simple minimiza el error cuadrático sin ninguna otra consideración:

```
minimizar: Σ(y_real - y_predicho)²
```

Con suficientes features (o features correlacionados entre sí, un problema real en forecasting cuando tienes múltiples lags altamente correlacionados como `demand_lag_1`, `demand_lag_2`, `demand_lag_3`), el modelo puede asignar coeficientes enormes y opuestos a features correlacionados para ajustar perfectamente el ruido del set de entrenamiento — memorización, no aprendizaje real. Esto es overfitting, y sus síntomas son coeficientes inestables (cambian drásticamente si quitas una sola fila de entrenamiento) y mal desempeño en datos nuevos.

## 2. Ridge (Regularización L2) — encoge coeficientes, no los elimina

```
minimizar: Σ(y_real - y_predicho)² + α · Σ(coeficientes²)
```

```python
from sklearn.linear_model import Ridge

modelo = Ridge(alpha=1.0)  # alpha controla la fuerza de la penalización
modelo.fit(X_train, y_train)
```

El término `α · Σ(coeficientes²)` penaliza coeficientes grandes — el modelo se ve forzado a mantenerlos moderados salvo que realmente aporten mucho poder predictivo. **Ridge nunca lleva un coeficiente exactamente a cero**, solo lo encoge — es ideal cuando **crees que todos los features aportan algo**, aunque sea poco, y quieres estabilidad numérica frente a features correlacionados (multicolinealidad).

## 3. Lasso (Regularización L1) — encoge Y elimina features

```
minimizar: Σ(y_real - y_predicho)² + α · Σ|coeficientes|
```

```python
from sklearn.linear_model import Lasso

modelo = Lasso(alpha=0.1)
modelo.fit(X_train, y_train)
# algunos coeficientes quedarán EXACTAMENTE en 0
features_seleccionados = X_train.columns[modelo.coef_ != 0]
```

La diferencia matemática entre `coeficientes²` (Ridge) y `|coeficientes|` (Lasso) parece sutil, pero tiene una consecuencia práctica enorme: Lasso puede llevar coeficientes **exactamente a cero**, haciendo selección automática de features. Si tienes 40 features candidatos para predecir demanda y sospechas que muchos son ruido, Lasso te dice cuáles realmente importan — una forma de feature selection integrada en el propio entrenamiento.

## 4. ElasticNet — lo mejor de ambos mundos

```python
from sklearn.linear_model import ElasticNet

modelo = ElasticNet(alpha=1.0, l1_ratio=0.5)  # l1_ratio: 0=Ridge puro, 1=Lasso puro
```

```
minimizar: Σ(y_real - y_predicho)² + α · [l1_ratio · Σ|coef| + (1-l1_ratio) · Σcoef²]
```

Cuando tienes features altamente correlacionados (como tus lags de demanda), Lasso puro puede comportarse erráticamente — elige arbitrariamente uno del grupo correlacionado y descarta el resto, incluso si todos son igual de informativos. ElasticNet combina la estabilidad de Ridge con la selección de Lasso, un compromiso robusto que en la práctica de la industria suele ser el punto de partida por defecto sobre Ridge o Lasso puros.

## 5. Cómo elegir `alpha` — nunca a ojo

```python
from sklearn.linear_model import RidgeCV, LassoCV

modelo = RidgeCV(alphas=[0.01, 0.1, 1.0, 10.0, 100.0], cv=5)
modelo.fit(X_train, y_train)
print(modelo.alpha_)  # el mejor alpha encontrado por cross-validation
```

`alpha` no es un hiperparámetro que se adivina — se busca sistemáticamente con cross-validation (ver [[37-Validacion-Rigurosa-en-ML]]), probando un rango logarítmico de valores y quedándote con el que generaliza mejor en folds de validación, no el que mejor ajusta el set de entrenamiento (eso sería, por definición, el `alpha` más bajo posible, y volverías al overfitting original).

## 6. Por qué importa esto aunque uses XGBoost en producción

Tres razones prácticas, no solo teóricas:

- **XGBoost y modelos de árboles también tienen regularización** (`reg_alpha`, `reg_lambda` en XGBoost — L1 y L2, literalmente los mismos conceptos aplicados a los pesos de las hojas del árbol en vez de a coeficientes lineales). Entender Ridge/Lasso te da intuición directa para tunear esos hiperparámetros con criterio, no a ciegas.
- **Un modelo lineal regularizado es tu baseline honesto**. Antes de justificar la complejidad de XGBoost ante tu jefa, un ElasticNet simple te dice si el problema tiene una relación mayormente lineal — si el modelo lineal ya captura el 90% del poder predictivo, la complejidad adicional de un modelo de árboles debe justificarse con una mejora real, no asumirse por defecto.
- **Interpretabilidad para el negocio**: los coeficientes de un modelo lineal regularizado son directamente interpretables ("cada llamada adicional de hace 30 minutos aumenta la predicción actual en X") — un lenguaje mucho más fácil de explicar a tu jefa que un feature importance de un ensamble de 300 árboles.

## Ver también
- Cheat-sheet técnico de scikit-learn (sintaxis y API): `Scikit-learn/06 - Modelos Lineales - Sintaxis y API.md`
- [[35-Arboles-de-Decision-en-Profundidad]]
- [[36-Ensambles-en-Profundidad]]
- [[37-Validacion-Rigurosa-en-ML]]
- [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]

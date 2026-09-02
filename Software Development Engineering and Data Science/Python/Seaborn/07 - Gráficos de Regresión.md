---
tags: [seaborn, python, data-science, visualization, regression, cheat-sheet]
---

# 07 — Gráficos de Regresión

> Continúa de [[06 - Gráficos Categóricos II - Estimadores]].

## `regplot()` — dispersión + línea de regresión ajustada

```python
sns.regplot(data=df, x="precio", y="demanda")
sns.regplot(data=df, x="precio", y="demanda", ci=95)          # banda de intervalo de confianza al 95% (default)
sns.regplot(data=df, x="precio", y="demanda", ci=None)          # sin banda de confianza
sns.regplot(data=df, x="precio", y="demanda", order=2)            # ajuste polinomial de grado 2, no solo lineal
sns.regplot(data=df, x="precio", y="demanda", logistic=True)        # regresión logística — 'y' debe ser binaria (0/1)
```

`regplot()` ajusta el modelo de regresión **en el momento de graficar**, con fines puramente exploratorios/visuales — no es un reemplazo de ajustar un modelo real con [[Scikit-learn/06 - Modelos Lineales - Sintaxis y API|Scikit-learn]] para hacer predicciones; es la herramienta correcta para ver rápidamente "¿hay una relación lineal aquí?" antes de modelar formalmente.

## `lmplot()` — la versión figure-level, con facetado

```python
sns.lmplot(data=df, x="precio", y="demanda", hue="region")                # una línea de regresión por región, superpuestas
sns.lmplot(data=df, x="precio", y="demanda", col="region")                   # un panel separado por región
sns.lmplot(data=df, x="precio", y="demanda", col="region", hue="canal")        # facetado + color combinados
```

Igual que `relplot`/`catplot`, `lmplot()` es un `FacetGrid` (ver [[09 - FacetGrid - Small Multiples]]) que llama a `regplot()` internamente por cada faceta/nivel de `hue`.

## `residplot()` — diagnóstico de residuos

```python
sns.residplot(data=df, x="precio", y="demanda")
```

Grafica los **residuos** (diferencia entre el valor real y el predicho por una regresión lineal simple) contra `x` — un patrón aleatorio sin estructura visible alrededor de cero sugiere que un modelo lineal es razonable; un patrón curvo o en forma de embudo (heterocedasticidad) sugiere que la relación real no es lineal o que la varianza no es constante, señales clave antes de confiar en un modelo lineal simple.

## Relación con modelos "de verdad"

```python
# Exploración rápida con Seaborn — SOLO para visualizar la tendencia
sns.regplot(data=df, x="precio", y="demanda")

# Modelo real para predicción/inferencia, con scikit-learn
from sklearn.linear_model import LinearRegression
modelo = LinearRegression()
modelo.fit(df[["precio"]], df["demanda"])
```

`regplot`/`lmplot` son deliberadamente simples (regresión lineal u polinomial básica, sin regularización, sin validación cruzada) — su propósito es el análisis exploratorio visual, no sustituir el flujo de trabajo completo de modelado cubierto en [[Scikit-learn/04 - Model Selection - Validación y Búsqueda|Scikit-learn]].

## Ver también

- [[06 - Gráficos Categóricos II - Estimadores]]
- [[08 - Mapas de Calor y Matrices]]
- [[13 - Estadística Integrada]]
- [[Scikit-learn/06 - Modelos Lineales - Sintaxis y API|Scikit-learn]]

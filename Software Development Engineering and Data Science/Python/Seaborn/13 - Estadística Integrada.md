---
tags: [seaborn, python, data-science, visualization, statistics, cheat-sheet]
---

# 13 — Estadística Integrada

> Continúa de [[12 - Temas y Estética]]. El "superpoder" real de Seaborn frente a graficar con Matplotlib puro — casi todas las funciones anteriores dependen de esto por debajo.

## El estimador por default: agregación automática

```python
sns.barplot(data=df, x="region", y="ventas")
```

Cuando hay múltiples filas por cada valor de `x` (por ejemplo, muchas transacciones por región), `barplot()` no grafica cada fila — **agrega automáticamente** con un estimador (la media, por default) antes de dibujar. Esto es exactamente lo que en Pandas requeriría un `groupby().mean()` explícito antes de graficar con Matplotlib puro (ver [[Python/Pandas/08 - Agrupación y Agregación (GroupBy)|Python/Pandas]]).

```python
sns.barplot(data=df, x="region", y="ventas", estimator="mean")      # default
sns.barplot(data=df, x="region", y="ventas", estimator="median")
sns.barplot(data=df, x="region", y="ventas", estimator="sum")
sns.barplot(data=df, x="region", y="ventas", estimator=len)           # cualquier función que reduzca una lista a un escalar
```

## Intervalos de confianza vía bootstrap

Las barras de error mostradas por default en `barplot`/`pointplot`/`lineplot` no son un cálculo analítico cerrado — Seaborn genera el intervalo de confianza mediante **bootstrapping**: remuestrea los datos con reemplazo miles de veces, calcula el estimador en cada remuestreo, y reporta el rango donde cae el 95% de esos resultados.

```python
sns.barplot(data=df, x="region", y="ventas", errorbar=("ci", 95))     # intervalo de confianza al 95% (default)
sns.barplot(data=df, x="region", y="ventas", errorbar=("ci", 68))       # equivalente aproximado a ±1 desviación estándar
sns.barplot(data=df, x="region", y="ventas", errorbar="sd")               # desviación estándar directa, sin bootstrap
sns.barplot(data=df, x="region", y="ventas", errorbar="se")                 # error estándar de la media
sns.barplot(data=df, x="region", y="ventas", errorbar=None)                   # sin barra de error
sns.barplot(data=df, x="region", y="ventas", n_boot=2000)                       # número de remuestreos bootstrap (default 1000)
```

**Por qué importa esto en la práctica:** un intervalo de confianza ancho en una barra señala que la muestra para esa categoría es pequeña o muy variable — leer las barras de error, no solo la altura de la barra, es lo que distingue una lectura superficial de un análisis exploratorio cuidadoso.

## Reproducibilidad del bootstrap

```python
sns.barplot(data=df, x="region", y="ventas", seed=42)     # fija la semilla del remuestreo bootstrap interno
```

Sin fijar `seed`, el ancho exacto de la banda de intervalo de confianza puede variar ligeramente entre ejecuciones del mismo código (por la naturaleza aleatoria del bootstrap) — relevante si se necesita que una figura sea exactamente reproducible byte a byte, no solo estadísticamente equivalente.

## Regresión: el mismo principio aplicado a una línea

```python
sns.regplot(data=df, x="precio", y="demanda", ci=95)    # la banda alrededor de la línea de regresión también es un intervalo de confianza
```

Ver [[07 - Gráficos de Regresión]] para el detalle completo — conceptualmente es el mismo bootstrap, aplicado a los coeficientes de la regresión ajustada en vez de a un estimador simple como la media.

## Cuándo desconfiar de un gráfico estadístico de Seaborn

- **Muestra muy pequeña por categoría:** el bootstrap con pocas observaciones produce intervalos de confianza engañosamente angostos o inestables entre ejecuciones.
- **Datos no independientes** (ej. mediciones repetidas de la misma entidad): el bootstrap estándar asume independencia entre observaciones — violar ese supuesto infla artificialmente la confianza aparente.
- **Usar el gráfico exploratorio como sustituto de una prueba de hipótesis formal:** un intervalo de confianza visual en un `barplot` es útil para intuición rápida, pero no reemplaza una prueba estadística formal cuando la decisión importa (ver `University Degree Notes/Ciclo 03/Probabilidad y Estadística/`).

## Ver también

- [[12 - Temas y Estética]]
- [[06 - Gráficos Categóricos II - Estimadores]]
- [[07 - Gráficos de Regresión]]
- [[Python/Pandas/08 - Agrupación y Agregación (GroupBy)|Python/Pandas]]
